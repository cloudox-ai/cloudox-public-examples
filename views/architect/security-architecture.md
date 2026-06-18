# Architect View — Security Architecture

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Security Architecture

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain (see below).

The most pressing architectural concern is an unrestricted security group in the sandbox account that exposes SSH (port 22) and PostgreSQL (port 5432) directly to the internet — this requires an immediate design decision. Beyond that critical item, the production Atlas workload carries an expected internet-facing edge (ALB + public subnets) whose exposure is verified but intentional by design; the architecture question is whether the current segmentation is sufficient.

---

### Trust & Identity

The environment surfaces **72 IAM roles** across accounts and **0 customer-managed IAM policies** — all permissions are expressed through inline or AWS-managed policies, which limits reuse and central auditability of permission boundaries.

Key roles observed across accounts:

| Role | Account | Architectural Significance |
|---|---|---|
| `cloudox-demo-sandbox-unused-admin` | 161388682021 | Admin-scoped role with no evidence of active use — high-privilege, low-justification |
| `OrganizationAccountAccessRole` | 122980216815, 110019496666, 105769365151, 122122642149, 161388682021 | Standard cross-account break-glass role present in all member accounts |
| `cloudox-demo-atlas-dev-ecs-exec` / `ecs-task` | 105769365151 | ECS execution and task identity for the dev Atlas workload |
| `cloudox-demo-atlas-dev-lambda` | 105769365151 | Lambda execution identity in dev |
| `cloudox-demo-atlas-prod-ecs-exec` | 122122642149 | ECS execution identity in prod |
| `cloudox-demo-atlas-prod-backup` | 122122642149 | Backup automation role in prod |
| `cloudox-demo-atlas-prod-dr-replicator` | 122122642149 | DR replication role — implies cross-region or cross-account data movement |
| `cloudox-demo-sandbox-scratch-lambda` | 161388682021 | Scratch Lambda role in sandbox — scope unknown |
| `stacksets-exec-*` | 161388682021 | CloudFormation StackSets execution role, managed by org admin |

Org-level CloudTrail (`cloudox-demo-org-trail-o-aaaapzvebq`) is present in `eu-central-1` under account `110319895932`, with a dedicated log-delivery role (`cloudox-demo-org-trail-logs`). CloudFormation StackSets org-admin delegation is also active (`AWSServiceRoleForCloudFormationStackSetsOrgAdmin`), indicating centrally-managed baseline deployments.

**Design concern:** `cloudox-demo-sandbox-unused-admin` (`AROAAAAADPCL3BVEXUDTH`) carries admin-level permissions and shows no evidence of active use. An unused admin role in a sandbox account is a standing privilege escalation path that should be removed or scoped down.

**Unknown:** No Security Hub enablement was discovered. There is no evidence of centralised findings aggregation or compliance posture scoring across accounts.

---

### Exposure & Boundaries

Four security groups and one load balancer carry verified internet-facing exposure. One is **critical** and requires a decision; the others are **medium** severity and consistent with an intentional public-edge pattern.

#### Critical — Sandbox permissive group

`cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`, account `161388682021`, `eu-central-1`) allows `0.0.0.0/0` and `::/0` ingress on:
- **Port 22 (SSH)** — direct shell access from the internet
- **Port 5432 (PostgreSQL)** — database port directly reachable from the internet

This is a **Priority 2 / Critical** finding (`security_exposure:security:sg-054446d655de1ee7f`). A decision is required: confirm whether this exposure is intentional (e.g., a temporary scratch environment) and restrict or remove it if not. The combination of SSH and a database port open to the world in the same group is architecturally indefensible in any persistent environment.

#### Medium — Production Atlas edge exposure

The following resources form the internet-facing edge of the Atlas production workload (account `122122642149`, `eu-central-1`) and are consistent with a standard public ALB pattern:

| Resource | ID | Open Port(s) | Finding |
|---|---|---|---|
| `cloudox-demo-atlas-prod-alb` | `cloudox-demo-atlas-prod-alb` | — (internet-facing scheme) | `security_exposure:security:cloudox-demo-atlas-prod-alb` |
| `cloudox-demo-atlas-prod-sg-edge` | `sg-0459201826f8de5b3` | 443 | `security_exposure:security:sg-0459201826f8de5b3` |
| `cloudox-demo-atlas-prod-sg-alb` | `sg-06f2b4190bf01d261` | 80 | `security_exposure:security:sg-06f2b4190bf01d261` |

Port 80 open on `cloudox-demo-atlas-prod-sg-alb` is worth confirming: if the ALB is expected to redirect HTTP→HTTPS, the security group rule is technically required but the redirect behaviour should be validated at the listener level.

---

### Segmentation

Four public subnets are confirmed across the prod and dev Atlas environments, all in `eu-central-1`. Their route tables forward to an Internet Gateway, making them internet-reachable by definition.

| Subnet | Account | Environment | Finding |
|---|---|---|---|
| `cloudox-demo-atlas-prod-public-a` (`subnet-013a24d318ed6f3d0`) | 122122642149 | Prod | `security_exposure:security:subnet-013a24d318ed6f3d0` |
| `cloudox-demo-atlas-prod-public-b` (`subnet-016c22941a019a137`) | 122122642149 | Prod | `security_exposure:security:subnet-016c22941a019a137` |
| `cloudox-demo-atlas-dev-public-a` (`subnet-065f522206524ab12`) | 105769365151 | Dev | `security_exposure:security:subnet-065f522206524ab12` |
| `cloudox-demo-atlas-dev-public-b` (`subnet-0f64d71c952a7898a`) | 105769365151 | Dev | `security_exposure:security:subnet-0f64d71c952a7898a` |

The presence of named public subnets in both prod and dev is consistent with a multi-tier VPC design (public subnets for the ALB/edge, private subnets for compute and data). However, the package does not contain evidence of private subnet configuration, NAT gateway placement, or network ACL rules — so the depth of the private-tier segmentation cannot be confirmed from available evidence.

**Modernization opportunity:** With 0 customer-managed IAM policies observed, there is a clear opportunity to introduce permission boundaries and reusable managed policies — particularly to constrain the `OrganizationAccountAccessRole` in each member account and to replace any inline policies on the ECS task/exec roles with auditable, version-controlled managed policies.

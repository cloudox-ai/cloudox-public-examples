# Architect View — Security Architecture

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Security Architecture

The most urgent design issue is a **critical, decision-required exposure** in the sandbox account: a security group with world-open ingress on SSH (22) and PostgreSQL (5432). This demands an explicit architectural decision before any further work proceeds. Alongside this, the production Atlas workload presents a deliberate but unconfirmed internet-facing surface that architects should validate against intent. IAM trust is broadly established via `OrganizationAccountAccessRole` across all member accounts, but at least one over-privileged, apparently unused admin role warrants attention. No Security Hub enablement was discovered, leaving automated security posture management as a gap.

*Overall confidence: Likely — derived from graph evidence; some unknowns remain.*

### Trust & Identity

The environment uses `OrganizationAccountAccessRole` as the cross-account trust mechanism, with instances present in five member accounts: `122980216815` (AROAAAAAB33DHRZ72LLFE), `110019496666` (AROAAAAACLQX0PDP245PQ), `105769365151` (AROAAAAAC6A0RHGHCRIWC), `122122642149` (AROAAAAADGV42TTJ3H83B), and `161388682021` (AROAAAAADE0WRYDYAJBXU). This is the standard AWS Organizations break-glass pattern, but its breadth means the management account is a high-value lateral-movement target — architects should confirm that access to this role is tightly controlled and audited.

A CloudTrail org-level trail (`arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the management account (`110319895932`), with a dedicated log-delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This provides a centralised audit record across the organisation.

The sandbox account (`161388682021`) contains a role named **cloudox-demo-sandbox-unused-admin** (`AROAAAAADPCL3BVEXUDTH`, `arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin`). The name signals administrative privilege and apparent disuse — this is a standing access risk and should be reviewed for removal or scope reduction.

Workload-specific roles follow a least-privilege naming pattern: the Atlas dev account (`105769365151`) has separate roles for ECS exec (`cloudox-demo-atlas-dev-ecs-exec`, AROAAAAADLCZISR8G4FBU), ECS task (`cloudox-demo-atlas-dev-ecs-task`, AROAAAAADLX1PNSXHC8KC), and Lambda (`cloudox-demo-atlas-dev-lambda`, AROAAAAACJ2F32IXZJ88A). The Atlas prod account (`122122642149`) similarly separates backup (`cloudox-demo-atlas-prod-backup`, AROAAAAAADXSPE5ZZWXXZ), DR replication (`cloudox-demo-atlas-prod-dr-replicator`, AROAAAAACBUQ1L1VU4R09), and ECS exec (`cloudox-demo-atlas-prod-ecs-exec`, AROAAAAABKVS2YU8FPYNK) concerns. This role decomposition is architecturally sound for blast-radius containment.

A StackSets service-linked role (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`) in the management account and a corresponding StackSets execution role in the sandbox (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`) indicate that CloudFormation StackSets is used for cross-account infrastructure deployment — a relevant dependency for any change management or IaC modernisation work.

The summary indicates **72 IAM roles** across the environment and **0 customer-managed policies**. The absence of customer-managed policies means all permission boundaries are expressed through AWS-managed policies or inline policies — a gap that limits reuse, auditability, and fine-grained least-privilege enforcement.

### Exposure & Boundaries

Three security groups are flagged as open to the internet, spanning two accounts and two distinct risk levels.

**Critical (decision required):**

| Resource | Account | Ports Open | Item ID |
|---|---|---|---|
| cloudox-demo-sandbox-sg-permissive (`sg-054446d655de1ee7f`) | `161388682021` | 22 (SSH), 5432 (PostgreSQL) | `security_exposure:security:sg-054446d655de1ee7f` |

World-open ingress on SSH and PostgreSQL from `0.0.0.0/0`/`::/0` is a critical design defect. Direct database port exposure to the internet is almost never intentional in a production or shared environment. Architects must confirm whether this is a temporary scratch configuration or a persistent misconfiguration, and restrict or remove the rules accordingly.

**Medium (confirm intent):**

| Resource | Account | Ports Open | Item ID |
|---|---|---|---|
| cloudox-demo-atlas-prod-sg-edge (`sg-0459201826f8de5b3`) | `122122642149` | 443 | `security_exposure:security:sg-0459201826f8de5b3` |
| cloudox-demo-atlas-prod-sg-alb (`sg-06f2b4190bf01d261`) | `122122642149` | 80 | `security_exposure:security:sg-06f2b4190bf01d261` |
| cloudox-demo-atlas-prod-alb | (account unknown) | Internet-facing ALB | `security_exposure:security:cloudox-demo-atlas-prod-alb` |

The Atlas prod edge and ALB security groups (`sg-0459201826f8de5b3`, `sg-06f2b4190bf01d261`) allow world-open ingress on 443 and 80 respectively, consistent with an internet-facing application load balancer (`cloudox-demo-atlas-prod-alb`). This pattern is expected for a public-facing service, but architects should confirm: (a) whether HTTP/80 should redirect to HTTPS rather than remain open, and (b) whether WAF or AWS Shield is in front of the ALB — neither is evidenced in this section's package.

### Segmentation

Four public subnets are identified across the Atlas prod and Atlas dev environments, all in `eu-central-1`. Their route tables forward traffic to an Internet Gateway, confirming direct internet reachability:

| Subnet | Friendly Name | Account | Item ID |
|---|---|---|---|
| `subnet-013a24d318ed6f3d0` | cloudox-demo-atlas-prod-public-a | `122122642149` | `security_exposure:security:subnet-013a24d318ed6f3d0` |
| `subnet-016c22941a019a137` | cloudox-demo-atlas-prod-public-b | `122122642149` | `security_exposure:security:subnet-016c22941a019a137` |
| `subnet-065f522206524ab12` | cloudox-demo-atlas-dev-public-a | `105769365151` | `security_exposure:security:subnet-065f522206524ab12` |
| `subnet-0f64d71c952a7898a` | cloudox-demo-atlas-dev-public-b | `105769365151` | `security_exposure:security:subnet-0f64d71c952a7898a` |

The naming convention (`public-a`, `public-b`) suggests a deliberate multi-AZ public tier, which aligns with the internet-facing ALB pattern in the prod account. The presence of equivalent public subnets in the dev account (`105769365151`) is worth reviewing — dev workloads exposed via public subnets carry the same internet-reachability risk as prod, and tighter egress-only or private-subnet configurations are worth considering for non-production environments.

Private subnet and NAT Gateway topology is not covered in this section's package. The segmentation picture — specifically whether application and data tiers are isolated in private subnets behind the public ALB tier — is a material unknown for a complete security architecture assessment.

> **Modernisation opportunity:** The absence of customer-managed IAM policies, combined with 72 roles, suggests permission management has grown organically. Introducing a customer-managed policy library (or AWS IAM Identity Center permission sets if not already in use) would improve auditability and enable consistent least-privilege enforcement across accounts. Additionally, confirming Security Hub enablement across the organisation would provide continuous posture management — no evidence of this was found.

# Generic View — Security & Exposure

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Security & Exposure

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain. Treat gaps noted below as areas requiring manual confirmation.

The most urgent finding is a sandbox security group with world-open SSH and database ports — a decision is required. Alongside that, detective controls (GuardDuty and IAM Access Analyzer) are inconsistently deployed across the organisation, leaving four of five non-sandbox accounts with reduced visibility into threats and external access. The production workload carries expected internet exposure through a load balancer and edge security groups, but those rules warrant confirmation.

### Internet Exposure

Three security groups and one load balancer present internet-facing exposure across two accounts.

| Resource | Account | Open Ports / Scheme | Severity | Decision Required? |
|---|---|---|---|---|
| `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) | Sandbox Ma (161388682021) | 22 (SSH), 5432 (PostgreSQL) — `0.0.0.0/0` and `::/0` | **Critical** | **Yes** |
| `cloudox-demo-atlas-prod-alb` | Unknown (account not resolved) | Internet-facing scheme | Medium | No |
| `cloudox-demo-atlas-prod-sg-edge` (`sg-0459201826f8de5b3`) | Workload Prod (122122642149) | 443 — `0.0.0.0/0` / `::/0` | Medium | No |
| `cloudox-demo-atlas-prod-sg-alb` (`sg-06f2b4190bf01d261`) | Workload Prod (122122642149) | 80 — `0.0.0.0/0` / `::/0` | Medium | No |

**Critical — action required:** `cloudox-demo-sandbox-sg-permissive` exposes SSH (22) and PostgreSQL (5432) to the entire internet. There is no evidence this is intentional. The recommended action is to confirm whether the exposure is required and, if not, restrict or remove those ingress rules immediately (`arn:aws:ec2:eu-central-1:161388682021:security-group/sg-054446d655de1ee7f`).

The production ALB (`cloudox-demo-atlas-prod-alb`) and its associated security groups (`cloudox-demo-atlas-prod-sg-edge` on port 443, `cloudox-demo-atlas-prod-sg-alb` on port 80) are internet-facing, which is consistent with serving external traffic. These are flagged as medium severity; confirm the exposure is intentional and that port 80 is either redirecting to HTTPS or required for a specific reason.

In total, 25 internet-facing access paths were observed across the environment.

### Identity & Access

72 IAM roles are present across the organisation. No customer-managed IAM policies were discovered — all role permissions appear to use AWS-managed or inline policies.

Several roles are notable from a security perspective:

- **`cloudox-demo-sandbox-unused-admin`** (Sandbox Ma, 161388682021 — `arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin`): The name indicates administrative permissions that may be unused. This warrants review to confirm whether the role is still needed.
- **`OrganizationAccountAccessRole`** exists in multiple member accounts (161388682021, 105769365151, 122122642149, and others). This is the standard AWS Organizations cross-account access role, granting the management account administrative access to each member. Its presence is expected but should be governed carefully.
- **`cloudox-demo-sandbox-scratch-lambda`** (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-scratch-lambda`) and **`stacksets-exec-899a82a2cb437e970934262f85c7628a`** (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`) are present in the sandbox account. The StackSets execution role is consistent with CloudFormation StackSets deployment from the management account (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`).
- Workload roles in the Dev account (`cloudox-demo-atlas-dev-ecs-exec`, `cloudox-demo-atlas-dev-ecs-task`, `cloudox-demo-atlas-dev-lambda`) and Prod account (`cloudox-demo-atlas-prod-backup`, `cloudox-demo-atlas-prod-dr-replicator`, `cloudox-demo-atlas-prod-ecs-exec`) follow a consistent naming convention, suggesting structured role separation by function.

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the management account, with a dedicated log delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This provides a centralised audit record across the organisation.

### Notable Risks

Two medium-severity, organisation-wide risks stand out because they affect the same five accounts and reduce the environment's ability to detect threats and unintended access.

**Uneven GuardDuty coverage** (`risk:security:aws-guardduty-detector`): GuardDuty threat detection is enabled in only 1 of 6 scanned accounts. The following accounts lack coverage: Log Archive (122980216815), Workload Dev (105769365151), Workload Prod (122122642149), Management (110319895932), and Sandbox Ma (161388682021). The recommended remediation is to enable GuardDuty org-wide via a delegated administrator.

**Uneven IAM Access Analyzer coverage** (`risk:security:aws-accessanalyzer-analyzer`): IAM Access Analyzer is similarly enabled in only 1 of 6 accounts, with the same five accounts uncovered. Without it, externally accessible resources (roles, S3 buckets, KMS keys, etc.) in those accounts will not be automatically flagged. The recommended remediation is to enable Access Analyzer org-wide via a delegated administrator.

Enabling both controls centrally from the management account would close these gaps across all member accounts simultaneously.

> **Gap:** No Security Hub enablement was discovered in this environment. Security Hub would aggregate findings from GuardDuty, Access Analyzer, and other services into a single pane — its absence means there is currently no centralised security findings aggregation.

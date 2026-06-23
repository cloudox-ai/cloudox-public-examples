# Generic View — Security & Exposure

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Security & Exposure

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain. Treat findings as strong signals requiring confirmation before remediation.

The most urgent finding is a sandbox security group with world-open access to SSH and PostgreSQL — a critical exposure that requires an immediate decision. Alongside this, two foundational detective controls (GuardDuty and IAM Access Analyzer) are absent from the majority of scanned accounts, leaving most of the environment with reduced visibility into threats and unintended external access. Three internet-facing security groups and one internet-facing load balancer round out the exposure surface.

### Internet Exposure

Three security groups are open to the internet (0.0.0.0/0 or ::/0), and one load balancer is internet-facing. Their scope and risk differ meaningfully:

| Resource | Account | Open Port(s) | Severity | Decision Required |
|---|---|---|---|---|
| cloudox-demo-sandbox-sg-permissive (`sg-054446d655de1ee7f`) | Sandbox Ma (161388682021) | 22 (SSH), 5432 (PostgreSQL) | **Critical** | Yes |
| cloudox-demo-atlas-prod-sg-edge (`sg-0459201826f8de5b3`) | Workload Prod (122122642149) | 443 (HTTPS) | Medium | No |
| cloudox-demo-atlas-prod-sg-alb (`sg-06f2b4190bf01d261`) | Workload Prod (122122642149) | 80 (HTTP) | Medium | No |
| cloudox-demo-atlas-prod-alb | Account not confirmed | Internet-facing scheme | Medium | No |

**`sg-054446d655de1ee7f` (cloudox-demo-sandbox-sg-permissive)** is the critical item. Unrestricted ingress on port 22 exposes SSH directly to the internet, and port 5432 exposes PostgreSQL — a database port that should never be internet-reachable. This is flagged as requiring an explicit decision: confirm whether the exposure is intentional and restrict it if not.

The two production security groups (`sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261`) and the load balancer `cloudox-demo-atlas-prod-alb` carry world-open rules on ports 443 and 80 respectively. These are consistent with a public-facing web application and are rated medium severity, but should be confirmed as intentional. The load balancer's account association was not confirmed in this data.

In total, 9 internet-facing access paths were observed across the environment, across 15 VPCs and 63 subnets.

### Identity & Access

72 IAM roles are present across the scanned accounts. No customer-managed IAM policies were discovered in this section's scope.

Several roles are notable from a security perspective:

- **`cloudox-demo-sandbox-unused-admin`** (Sandbox Ma, `161388682021`) — The name indicates administrative permissions that may be unused. An unused admin role represents unnecessary standing privilege and should be reviewed for removal or scope reduction.
- **`OrganizationAccountAccessRole`** — Present in five accounts (122980216815, 110019496666, 105769365151, 122122642149, 161388682021). This is the standard AWS Organizations cross-account access role, granting the management account administrative access to member accounts. Its presence is expected in an AWS Organizations structure but warrants awareness — access to the management account (`110319895932`) transitively enables administrative access across all member accounts where this role exists.
- **`cloudox-demo-sandbox-scratch-lambda`** and **`stacksets-exec-899a82a2cb437e970934262f85c7628a`** (Sandbox Ma, `161388682021`) — Roles associated with Lambda execution and CloudFormation StackSets execution respectively. Their scope of permissions is not detailed in this section.
- **`cloudox-demo-atlas-prod-backup`**, **`cloudox-demo-atlas-prod-dr-replicator`**, **`cloudox-demo-atlas-prod-ecs-exec`** (Workload Prod, `122122642149`) — Purpose-named roles for backup, DR replication, and ECS execution in the production workload account.
- **`cloudox-demo-atlas-dev-ecs-exec`**, **`cloudox-demo-atlas-dev-ecs-task`**, **`cloudox-demo-atlas-dev-lambda`** (Workload Dev, `105769365151`) — Execution roles for ECS and Lambda in the dev workload account.

An org-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the management account, with a dedicated log delivery role (`cloudox-demo-org-trail-logs`). This provides a centralised audit record across the organisation.

### Notable Risks

Two medium-severity, organisation-wide risks stand out because they affect detective control coverage across 5 of 6 scanned accounts:

**GuardDuty is enabled in only 1 of 6 scanned accounts.** The five accounts without coverage are: Log Archive (122980216815), Workload Dev (105769365151), Workload Prod (122122642149), Management (110319895932), and Sandbox Ma (161388682021). Without GuardDuty, these accounts lack automated threat detection for credential compromise, unusual API activity, and network-level threats.

**IAM Access Analyzer is enabled in only 1 of 6 scanned accounts.** The same five accounts are uncovered. Without Access Analyzer, unintended external access to S3 buckets, IAM roles, KMS keys, and other resource-based policies may go undetected.

The recommended remediation for both is to enable them org-wide via a delegated administrator in the management account, which would provide consistent coverage without per-account configuration.

**Security Hub enablement was not discovered** in this data. Whether Security Hub is active in any account is unknown from this section's evidence.

The `cloudox-demo-sandbox-unused-admin` role (Sandbox Ma) also warrants attention as a standing privilege risk — an admin-scoped role that appears unused represents an unnecessary blast radius if credentials associated with it were ever compromised.

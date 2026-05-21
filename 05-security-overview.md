# Security Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

## Internet Exposure

Three security groups carry world-open ingress rules (0.0.0.0/0 or ::/0). One of these also exposes sensitive ports.

| Security Group | Account | Name | World-Open Ports | Sensitive Ports | Finding |
|---|---|---|---|---|---|
| `sg-054446d655de1ee7f` | 161388682021 (sandbox) | `cloudox-demo-sandbox-sg-permissive` | 22, 80, 5432 | 22 (SSH), 5432 (PostgreSQL) | `exposure:AWS::EC2::SecurityGroup:sg-054446d655de1ee7f` |
| `sg-06f2b4190bf01d261` | 122122642149 (workload-prod) | `cloudox-demo-atlas-prod-sg-alb` | 80 | — | `exposure:AWS::EC2::SecurityGroup:sg-06f2b4190bf01d261` |
| `sg-0459201826f8de5b3` | 122122642149 (workload-prod) | `cloudox-demo-atlas-prod-sg-edge` | 443 | — | `exposure:AWS::EC2::SecurityGroup:sg-0459201826f8de5b3` |

`sg-06f2b4190bf01d261` and `sg-0459201826f8de5b3` appear to front an internet-facing load balancer (`cloudox-demo-atlas-prod-alb`; finding `exposure:AWS::ElasticLoadBalancingV2::LoadBalancer:cloudox-demo-atlas-prod-alb`). The relationship data shows `sg-0457a284b8b9ead6c` allows from `sg-06f2b4190bf01d261`, and a further downstream group `sg-0831f3eae117601d2` allows from `sg-0457a284b8b9ead6c`, suggesting a layered ingress chain — validate with the customer.

`sg-054446d655de1ee7f` in the sandbox account exposes SSH (22) and PostgreSQL (5432) to the world. No load balancer relationship was detected for this group; the exposure basis is the ingress rule alone.

Five public subnets were detected via route-table entries pointing to an Internet Gateway:

- `subnet-065f522206524ab12`, `subnet-029e2cceb3d0beff7`, `subnet-016c22941a019a137`, `subnet-013a24d318ed6f3d0`, `subnet-0f64d71c952a7898a`

Account and VPC membership for these subnets should be confirmed at the workshop. No relationship between these subnets and the world-open security groups was detected in the provided data.

Of the 23 total security groups, 20 are classified as internal-only (digest `security_groups.internal_only`).

---

## IAM Shape

The digest records 72 total IAM roles across the organisation: 24 customer-managed roles, 48 service-linked roles, and 1 IAM user.

Two roles are flagged as admin-like (digest `iam.admin_like_role_refs`):

| Role ARN | Account | Name | Basis |
|---|---|---|---|
| `arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin` | 110319895932 (management) | `AWSServiceRoleForCloudFormationStackSetsOrgAdmin` | digest `admin_like_role_refs`: `AROAAAAAD3BM1QMZMPH6S` |
| `arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin` | 161388682021 (sandbox) | `cloudox-demo-sandbox-unused-admin` | digest `admin_like_role_refs`: `AROAAAAADPCL3BVEXUDTH` |

`AWSServiceRoleForCloudFormationStackSetsOrgAdmin` is a service-linked role; its admin-like permissions are expected for StackSets operation and should be validated against actual policy content. `cloudox-demo-sandbox-unused-admin` is a customer-managed role whose name suggests it may be unused — the relationship data confirms it has an `assumes_role` relationship from `cloudox-demo-sandbox-unused-admin` to `AROAAAAADPCL3BVEXUDTH`, meaning the role can be assumed. Whether it has been assumed recently is not determinable from the provided data.

No customer-managed IAM policies were detected (digest `iam.customer_managed_policies: 0`). All policy attachments appear to use AWS-managed or inline policies (69 `AWS::IAM::RolePolicyAttachment` resources, 7 `AWS::IAM::RoleInlinePolicy` resources in graph stats). Inline policy content was not provided; scope cannot be assessed.

No IAM groups were detected (digest `iam.groups: 0`). The single IAM user's identity and policy attachments are not visible in the provided data.

Notable roles by account that suggest workload function (names only — policy content not available):

- `cloudox-demo-atlas-prod-ecs-task` / `cloudox-demo-atlas-prod-ecs-exec` / `cloudox-demo-atlas-prod-lambda` / `cloudox-demo-atlas-prod-backup` / `cloudox-demo-atlas-prod-dr-replicator` — account 122122642149
- `cloudox-demo-atlas-dev-ecs-task` / `cloudox-demo-atlas-dev-ecs-exec` / `cloudox-demo-atlas-dev-lambda` — account 105769365151
- `cloudox-demo-security-audit` / `cloudox-demo-incident-response` — account 110019496666 (audit)
- `cloudox-demo-log-archive-read` — account 122980216815 (log-archive)

These names are indirect signals only. Actual trust policies and permission boundaries were not provided.

---

## Encryption (KMS)

The digest records 14 KMS keys: 9 customer-managed keys (CMKs) and 5 AWS-managed keys, with 153 aliases. No key IDs, aliases, or account assignments are individually listed in the digest beyond counts. Whether all workload data stores use CMKs versus AWS-managed keys cannot be determined from the provided data.

---

## S3 Versioning Gaps

Four of the 10 discovered S3 buckets lack versioning (digest `s3.unversioned`):

| Bucket | Account |
|---|---|
| `cloudox-demo-access-logs-122980216815-eu-central-1` | 122980216815 (log-archive) |
| `cloudox-demo-sandbox-161388682021-eu-central-1` | 161388682021 (sandbox) |
| `cloudox-demo-sandbox-abandoned-108322657170-eu-central-1` | 161388682021 (sandbox) |
| `cloudox-demo-atlas-dev-app-105769365151-eu-central-1` | 105769365151 (workload-dev) |

The access-logs bucket (`cloudox-demo-access-logs-122980216815-eu-central-1`) in the log-archive account being unversioned is worth validating — log integrity may depend on versioning or Object Lock. The `cloudox-demo-sandbox-abandoned-161388682021-eu-central-1` name suggests the bucket may be inactive; confirm whether it holds data.

No public-access block status or bucket policy details were provided for any bucket.

---

## Secrets

Four Secrets Manager secrets were detected (digest `secrets.count: 4`). Secret names, owning accounts, rotation status, and access policies are not available in the provided data.

---

## Security Services — Observed State

| Service | Observed | State |
|---|---|---|
| CloudTrail | Trail `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq` present; `multi_region: true`; `log_file_validation: true` | Trail resource properties confirmed. Whether the trail is currently logging is not determinable from the provided data. |
| GuardDuty | 1 detector resource present (`AWS::GuardDuty::Detector`; digest `guardduty_detectors: 1`) | Enabled state not available — no status field present in the provided data. |
| Security Hub | No hub resource detected (digest `securityhub_hubs: 0`) | Digest explicitly lists "No Security Hub enablement discovered." |
| AWS Config | 0 recorders, 0 rules (digest `config_recorders: 0`, `config_rules: 0`) | No Config recorder or rule resources detected. |
| Access Analyzer | 1 analyzer resource present (digest `access_analyzers: 1`) | Analyzer type, scope, and active state not available in the provided data. |

The CloudTrail trail appears in the management account (110319895932) and is marked multi-region. Whether it covers all member accounts as an organisation trail cannot be confirmed from the resource properties alone — validate at the workshop.

No explicit risk records were generated (graph stats `risk_count: 0`), but the findings above represent exposure-category evidence that warrants review.

---

## Backup

One backup plan and one backup vault were detected (digest `backups.backup_plans: 1`, `backup_vaults: 1`). The role `cloudox-demo-atlas-prod-backup` (account 122122642149) suggests backup operations in the production workload account. Coverage of other accounts and data stores is not determinable from the provided data.

---

## Unknowns / Gaps

- **Security Hub**: No hub resource detected. The digest explicitly flags this as unknown. Confirm whether Security Hub is intentionally not deployed or deployed outside the discovery scope.
- **AWS Config**: No recorder or rule resources detected. Confirm whether Config is deployed in any account.
- **GuardDuty detector state**: Detector resource present but enabled/disabled status not available. Validate in console.
- **CloudTrail logging state**: Trail resource properties are present; active logging status was not captured.
- **IAM policy content**: No managed or inline policy documents were provided. Admin-like and workload role scopes cannot be assessed without them.
- **S3 public access block and bucket policies**: Not available for any of the 10 buckets.
- **KMS key-to-resource mapping**: Which data stores use CMKs versus AWS-managed keys is not determinable from the provided data.
- **Single IAM user**: One user exists (digest `iam.users: 1`). Identity, credentials, and MFA status are not available.
- **Secrets rotation**: Rotation status for the 4 Secrets Manager secrets is not available.

---

## Follow-up Questions

1. Is `cloudox-demo-sandbox-unused-admin` (account 161388682021, `AROAAAAADPCL3BVEXUDTH`) actively used? If not, what is the decommission plan?
2. What resources are attached to `sg-054446d655de1ee7f`? SSH and PostgreSQL open to 0.0.0.0/0 in the sandbox account — is this intentional and isolated from production networks?
3. Is the CloudTrail trail `cloudox-demo-org-trail-o-aaaapzvebq` configured as an organisation trail covering all member accounts, and is it currently logging?
4. Is GuardDuty detector (account not specified in digest) currently enabled, and does it cover all accounts in the organisation?
5. Is Security Hub intentionally absent, or is it deployed in an account outside the discovery scope?
6. Is AWS Config deployed in any account? The absence of recorder resources may indicate a gap in configuration change tracking.
7. Who is the single IAM user, and does the account have MFA enforced?
8. What is the purpose of `cloudox-demo-sandbox-abandoned-161388682021-eu-central-1`? Does it contain data that should be retained or deleted?
9. Are the 4 Secrets Manager secrets subject to automatic rotation? Which workloads consume them?
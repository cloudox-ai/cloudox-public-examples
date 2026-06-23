# Operations View — Operational Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Operational Risks

Three active risks span detection coverage and identity privilege across the organisation. Two of them — missing GuardDuty and missing IAM Access Analyzer — affect the same five accounts and represent the most operationally significant gaps: incidents or policy violations in those accounts will go undetected or unanalysed until the services are enabled.

### Highest Operational Risks

#### 1. Uneven GuardDuty Threat Detection Coverage

**Risk ID:** `risk:security:aws-guardduty-detector` | **Severity:** Medium | **Priority:** 2

GuardDuty is enabled in only 1 of 6 scanned accounts. The following five accounts have no GuardDuty detector active:

| Account | Account ID |
|---|---|
| Log Archive Account | 122980216815 |
| Workload Dev Account | 105769365151 |
| Workload Prod Account | 122122642149 |
| Management Account | 110319895932 |
| Sandbox Ma Account | 161388682021 |

**Operational impact:** Threat events (port scans, credential abuse, C2 callbacks, unusual API calls) occurring in these accounts produce no GuardDuty findings. There is no automated detection signal to feed alerting pipelines or incident response workflows for those accounts.

**What to do:** Enable GuardDuty across all accounts, preferably org-wide through a delegated administrator so coverage cannot drift per-account.

> Note: The Log Archive Account (122980216815) is listed with `Likely` confidence — verify detector status directly in that account.

---

#### 2. Uneven IAM Access Analyzer Coverage

**Risk ID:** `risk:security:aws-accessanalyzer-analyzer` | **Severity:** Medium | **Priority:** 2

IAM Access Analyzer is enabled in only 1 of 6 scanned accounts. The same five accounts listed above are uncovered.

**Operational impact:** Resource policies that unintentionally grant external access (S3 buckets, KMS keys, IAM roles, Lambda functions, SQS queues) will not be flagged in these accounts. This is particularly concerning for the Workload Prod Account (122122642149) and Management Account (110319895932), where undetected external access paths carry the highest blast radius.

**What to do:** Enable IAM Access Analyzer org-wide via a delegated administrator. This ensures new findings are centralised and consistent coverage is maintained without per-account configuration.

> Note: Same confidence caveat applies to Log Archive Account (122980216815).

---

#### 3. Broadly-Privileged IAM Role: cloudox-demo-sandbox-unused-admin

**Risk ID:** `risk:security:cloudox-demo-sandbox-unused-admin` | **Severity:** Medium | **Priority:** 3

The IAM role `cloudox-demo-sandbox-unused-admin` (ID: `AROAAAAADPCL3BVEXUDTH`) exists in the **Sandbox Ma Account** (161388682021). Its name implies broad or administrative access.

**Operational impact:** An over-privileged role that is unused represents a standing identity blast radius — if assumed by a compromised principal or misconfigured trust policy, it could allow wide-ranging actions within the sandbox account.

**Confidence note (Assumed):** Privilege breadth was not directly collected; this risk is inferred from the role name. The actual attached policies must be reviewed before drawing firm conclusions.

**What to do:** Inspect the attached policies for this role. If administrative permissions are confirmed and the role is genuinely unused, remove it or scope it down to least privilege.

---

#### Coverage Gaps Affecting This Section

Two meta-collectors were unavailable during discovery, which limits confidence in the completeness of the risk inventory:

- **AWS Resource Explorer** was disabled or unavailable — the full breadth of AWS-visible resources could not be cross-checked.
- **AWS Cloud Control** meta-collector was disabled — long-tail resource types are limited to typed collector coverage only.

This means additional risks in resource categories not covered by typed collectors may not appear here.

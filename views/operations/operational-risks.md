# Operations View — Operational Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Operational Risks

Two detection-coverage gaps affect five of six scanned accounts simultaneously, leaving the majority of the estate with reduced visibility into both external access exposure and active threats. A third risk — a broadly-named admin IAM role in the Sandbox — carries lower confidence but warrants validation before it is dismissed.

> **Coverage note:** Resource Explorer and Cloud Control meta-collectors were unavailable during this scan. Long-tail resource types and AWS-visible breadth could not be cross-checked; additional risks may exist beyond typed collector coverage.

### Highest Operational Risks

#### 1. Uneven GuardDuty Threat Detection Coverage — 5 accounts uncovered
`risk:security:aws-guardduty-detector` · Severity: **Medium** · Priority 2

GuardDuty is enabled in only 1 of 6 scanned accounts. The following accounts have **no active threat detection**:

| Account | ID |
|---|---|
| Log Archive Account | 122980216815 |
| Workload Dev Account | 105769365151 |
| Workload Prod Account | 122122642149 |
| Management Account | 110319895932 |
| Sandbox Ma Account | 161388682021 |

**Operational impact:** Malicious activity — credential abuse, crypto-mining, C2 communication, lateral movement — occurring in these accounts will not generate GuardDuty findings. Incidents may go undetected until they surface through other means (billing anomalies, customer reports, etc.).

**What can break / recovery risk:** Without findings, automated or manual incident-response runbooks that depend on GuardDuty alerts will not trigger. The Workload Prod Account (`122122642149`) and Management Account (`110319895932`) are the highest-consequence gaps.

**Recommended action:** Enable GuardDuty across all accounts, preferably org-wide via a delegated administrator in the Management Account to ensure consistent, centrally-managed coverage.

---

#### 2. Uneven IAM Access Analyzer Coverage — 5 accounts uncovered
`risk:security:aws-accessanalyzer-analyzer` · Severity: **Medium** · Priority 2

IAM Access Analyzer is enabled in only 1 of 6 scanned accounts. The same five accounts listed above are uncovered.

**Operational impact:** Unintended external access to S3 buckets, IAM roles, KMS keys, SQS queues, and other resource-based policies in these accounts will not be surfaced automatically. Misconfigurations that expose resources to the internet or to external AWS principals will remain silent.

**What can break / recovery risk:** The Log Archive Account (`122980216815`) is particularly sensitive — undetected external access to log data could undermine audit integrity and compliance posture. The Management Account (`110319895932`) exposure is similarly high-consequence.

**Recommended action:** Enable IAM Access Analyzer org-wide via a delegated administrator. This provides a single pane of findings and avoids per-account enablement drift.

---

#### 3. Broadly-Privileged IAM Role: `cloudox-demo-sandbox-unused-admin`
`risk:security:cloudox-demo-sandbox-unused-admin` · Severity: **Medium** · Priority 3 · Confidence: **Assumed**

The IAM role `cloudox-demo-sandbox-unused-admin` (ID: `AROAAAAADPCL3BVEXUDTH`) exists in the **Sandbox Ma Account** (`161388682021`). Its name suggests broad or administrative access, but attached policy details were not collected — the privilege breadth is inferred from naming only and must be validated directly.

**Operational impact (if confirmed):** An over-privileged, potentially unused role represents a large identity blast radius. If the role's trust policy allows broad assumption, it could be exploited to escalate privileges or move laterally within the sandbox.

**What requires operational attention:** Confirm whether the role is actively used, review all attached managed and inline policies, and apply least-privilege scoping. If the role is genuinely unused, consider deleting it.

**Recommended action:** Review attached policies; apply least privilege. Validate before treating this as a confirmed risk — the current assessment is based on naming convention alone.

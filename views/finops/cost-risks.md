# FinOps View — Cost Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Cost Risks

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All findings below are derived from the discovered architecture, not from billing data. Dollar attributions cannot be made here; validate against AWS Cost Explorer or CUR directly.

The most material cost risk in this environment is not overspend on a single service — it is the **absence of security controls whose gaps could generate unplanned remediation and incident costs**. Three security findings with direct cost-risk implications are present across the estate.

### Cost & Commitment Risks

#### Security Control Gaps as Cost Risk

Two detective controls — GuardDuty threat detection and IAM Access Analyzer — are enabled in only **1 of 6 scanned accounts**, leaving five accounts (Log Archive `122980216815`, Workload Dev `105769365151`, Workload Prod `122122642149`, Management `110319895932`, and Sandbox Ma `161388682021`) without consistent coverage (`risk:security:aws-guardduty-detector`, `risk:security:aws-accessanalyzer-analyzer`).

For a FinOps audience, the relevance is indirect but real:

| Control | Accounts Missing | Cost-Risk Angle |
|---|---|---|
| GuardDuty (`risk:security:aws-guardduty-detector`) | 5 of 6 | Undetected compromise can lead to crypto-mining, data exfiltration egress charges, or large-scale resource spin-up — all of which appear on the bill before they are caught. |
| IAM Access Analyzer (`risk:security:aws-accessanalyzer-analyzer`) | 5 of 6 | Undetected over-exposed resources increase the blast radius of a breach, compounding incident and remediation costs. |

Enabling both controls org-wide via a delegated administrator is the recommended action for each. GuardDuty and IAM Access Analyzer do carry their own service charges, but these are typically small relative to the cost of an undetected incident.

#### Broadly-Privileged IAM Role (Requires Validation)

An IAM role named `cloudox-demo-sandbox-unused-admin` (ID: `AROAAAAADPCL3BVEXUDTH`) exists in the Sandbox Ma Account (`161388682021`) and is flagged as potentially broadly privileged based on naming (`risk:security:cloudox-demo-sandbox-unused-admin`). **Confidence on this finding is Assumed** — privilege breadth was not directly collected and must be validated by reviewing attached policies.

The cost-risk angle: an over-privileged role in a sandbox account, if misused or compromised, could be used to provision resources at scale with no guardrails. This warrants validation before it is dismissed.

> **Requires validation:** Confirm the actual policies attached to `cloudox-demo-sandbox-unused-admin` and apply least-privilege scoping if the role is broader than needed.

#### What Is Not Visible Here

The following cost-risk areas could not be assessed in this run and represent material unknowns for FinOps:

- **No spend data** — Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false`). Concentration, trend, and anomaly analysis are not possible without it.
- **No utilization metrics** — CloudWatch metrics are not collected in this version, so idle resources, underutilized compute, and right-sizing opportunities cannot be identified.
- **Collector gaps** — RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured, so cost drivers in those areas are not detected.

Enabling Cost Explorer collection in a future run would unlock spend-grounded analysis and shift confidence from Likely to Verified for cost findings.

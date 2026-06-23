# FinOps View — Cost Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Cost Risks

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All findings below are derived from the discovered architecture. Dollar attributions cannot be provided; use AWS Cost Explorer or CUR for exact figures.

The most actionable cost risk in this environment is not a billing line item — it is the absence of security controls that, if exploited, could generate unplanned spend at scale. Two detective controls (GuardDuty and IAM Access Analyzer) are missing from 5 of 6 scanned accounts, and a broadly-privileged IAM role exists in the Sandbox account. These gaps represent indirect cost exposure: a compromised or over-privileged identity can spin up resources, exfiltrate data, or trigger incident-response costs that dwarf the modest price of the controls themselves.

### Cost & Commitment Risks

#### Security gaps as cost exposure

The three architecture-derived risks surfaced in this section all carry financial consequence if left unaddressed:

| Risk | Affected Accounts | Severity | Cost Exposure Mechanism |
|---|---|---|---|
| Uneven GuardDuty coverage (`risk:security:aws-guardduty-detector`) | Log Archive, Workload Dev, Workload Prod, Management, Sandbox Ma (5 of 6) | Medium | Undetected compromise → unauthorized resource provisioning, data-transfer charges, incident response |
| Uneven IAM Access Analyzer coverage (`risk:security:aws-accessanalyzer-analyzer`) | Same 5 accounts | Medium | Undetected external access → data exfiltration, regulatory fines, forensic costs |
| Broadly-privileged IAM role `cloudox-demo-sandbox-unused-admin` (`risk:security:cloudox-demo-sandbox-unused-admin`) | Sandbox Ma Account (`161388682021`) | Medium | Over-privileged identity → large blast radius if misused or compromised |

**GuardDuty and IAM Access Analyzer** are both low-cost services relative to the incidents they prevent. GuardDuty pricing is based on data volume analyzed; for most accounts the monthly cost is modest. Enabling them org-wide via a delegated administrator (the recommended action for both) is the most cost-efficient path and eliminates per-account configuration overhead. *Requires validation: confirm current GuardDuty and Access Analyzer enrollment status in the 5 uncovered accounts before enabling, to avoid duplicate detector charges.*

**The `cloudox-demo-sandbox-unused-admin` role** (IAM role `AROAAAAADPCL3BVEXUDTH` in Sandbox Ma Account `161388682021`) is flagged on naming convention; actual privilege breadth has not been collected and must be validated. If the role carries broad permissions and is unused, it represents an unnecessary blast radius. *Requires validation: review attached policies and last-used data before modifying.*

#### What is not covered here

The following cost driver categories are outside the scope of this section due to collection gaps — they are not unknown to the environment, but are not available in this run:

- **Spend figures**: Cost Explorer collection is disabled (`CLOUDOX_COST__ENABLED=false`). No monthly totals, trends, or service-level breakdowns are available.
- **Utilization-based waste**: CloudWatch metrics are not collected in this version; idle or underutilized compute, right-sizing opportunities, and savings plan coverage gaps cannot be assessed from usage data.
- **Specific service cost drivers**: RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by current collectors.

To unlock spend-based analysis, enable Cost Explorer collection and re-run discovery.

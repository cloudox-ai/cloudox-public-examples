# FinOps View — Cost Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Cost Overview

Spend figures are not available for this analysis run — Cost Explorer collection was disabled at discovery time. What follows is architecture-derived: the accounts and resource patterns that are structurally positioned to carry charges, without dollar amounts attached. Treat this as a map of *where to look* in Cost Explorer or CUR, not a billing report.

> **Confidence: Likely** — All cost driver observations are inferred from discovered architecture. No spend data was collected. Validate against your billing data before acting.

### Total Spend Context

No spend figures can be reported. Cost Explorer collection was not enabled during this discovery run (`CLOUDOX_COST__ENABLED=false`), so no UnblendedCost data is available for any billing period.

Seven accounts are in scope across this environment:

| Account | Friendly Name | Confidence |
|---|---|---|
| 110319895932 | Management Account | Verified |
| 161388682021 | Sandbox Ma Account | Verified |
| 105769365151 | Workload Dev Account | Verified |
| 122122642149 | Workload Prod Account | Verified |
| 122980216815 | Log Archive Account | Likely |
| 110019496666 | Audit Account | Likely |
| 150982215529 | Platform Account | Likely |

The presence of dedicated **Workload Dev** and **Workload Prod** accounts alongside a **Sandbox** account suggests at least three distinct spend pools that should be tracked separately in Cost Explorer. The **Log Archive** and **Audit** accounts are typical AWS Control Tower foundational accounts and generally carry lower but non-trivial storage and API costs.

### Where Cost Concentrates

Without spend data, concentration can only be described structurally. Based on the discovered account topology, cost is most likely to concentrate in:

- **Workload Prod Account (122122642149)** — Production workload accounts are the primary driver of compute, data transfer, and managed service charges in most AWS environments. This is the highest-priority account to examine in Cost Explorer.
- **Workload Dev Account (105769365151)** — Development environments frequently carry disproportionate cost relative to business value, particularly from always-on compute and over-provisioned resources. A candidate for rightsizing and scheduling controls, but requires validation against actual usage data.
- **Log Archive Account (122980216815)** — Centralised log archiving accumulates S3 storage costs continuously. S3 storage class selection and lifecycle policies are not captured by the current collectors, so the cost profile here is not known.
- **Platform Account (150982215529)** — Shared platform services (networking, DNS, shared tooling) can carry steady-state costs that are easy to overlook. Exact services in this account are not detailed in this package.

The **Management Account (110319895932)** and **Audit Account (110019496666)** typically carry lower direct workload costs but may include AWS Organizations, CloudTrail, and Config charges.

**What is not visible in this analysis:** CloudWatch utilization metrics are not collected in this version, so idle or underutilized resource identification is not possible. RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage class details are also outside current collector coverage — these can be material cost drivers that will not appear here.

**Recommended next step (requires validation):** Enable Cost Explorer collection in CloudoX and re-run discovery to populate spend figures. In the interim, filter Cost Explorer by the seven account IDs above, grouped by account and service, to establish the actual concentration picture.

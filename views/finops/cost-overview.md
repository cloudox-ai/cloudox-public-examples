# FinOps View — Cost Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Cost Overview

Spend figures are not available for this analysis run — Cost Explorer collection was disabled at discovery time (`CLOUDOX_COST__ENABLED=false`). The observations below are derived entirely from the discovered architecture. Dollar amounts, period totals, and service-level breakdowns should be obtained directly from AWS Cost Explorer or your Cost and Usage Report (CUR).

> **Confidence: Likely** — All cost driver observations are inferred from architectural patterns, not from billing data. Treat them as hypotheses to validate against your actual spend.

### Total Spend Context

No billing data was collected for this run, so no total spend figure, trend, or period-over-period comparison can be stated. The environment spans seven accounts across two categories:

| Account | Role |
|---|---|
| Management Account (`110319895932`) | Payer / management |
| Workload Prod Account (`122122642149`) | Production workloads |
| Workload Dev Account (`105769365151`) | Development workloads |
| Sandbox Ma Account (`161388682021`) | Sandbox / experimentation |
| Platform Account (`150982215529`) | Shared platform services |
| Log Archive Account (`122980216815`) | Centralised log storage |
| Audit Account (`110019496666`) | Security audit trail |

This multi-account structure is typical of an AWS Organizations landing zone. In practice, the majority of compute and data spend tends to concentrate in the Workload Prod Account and Platform Account, while Log Archive and Audit accounts carry ongoing storage costs that grow with retention. These are patterns to confirm against your CUR — they are not dollar attributions.

To establish a baseline, enable Cost Explorer collection in CloudoX and re-run discovery, or pull a service-level CUR breakdown filtered by linked account for the accounts listed above.

### Where Cost Concentrates

Without spend data, concentration can only be described architecturally. The discovery identified **7 architectural cost driver patterns** across the environment and **0 optimization candidates** were surfaced at this time (optimization candidates require either spend data or CloudWatch utilization metrics, neither of which was collected in this run).

Key structural factors that typically drive cost concentration in an environment of this shape:

- **Production workload account** — Compute, data transfer, and managed service charges in `Workload Prod Account` are the most likely single largest cost centre. Sizing, reservation coverage, and Savings Plans attachment for this account should be the first validation target.
- **Platform shared services** — The `Platform Account` commonly hosts networking constructs (NAT Gateways, Transit Gateway attachments, VPN endpoints) and shared tooling. NAT Gateway data-processing charges in particular can be a non-obvious cost driver that does not appear large per resource but accumulates at scale.
- **Log and audit storage** — `Log Archive Account` and `Audit Account` accumulate S3 storage continuously. S3 storage class selection and lifecycle policies are not captured by current collectors (see unknowns below), so the cost impact of log retention is not yet quantified.
- **Sandbox account** — `Sandbox Ma Account` can carry uncontrolled spend if resource lifecycle governance is not enforced. This is worth filtering in Cost Explorer to confirm it is bounded.

**What to do next:**
1. Enable cost collection and re-run to get dollar-level attribution per account and service.
2. Pull a CUR or Cost Explorer breakdown by Linked Account × Service for the last three complete months to confirm which accounts and services dominate.
3. Check Savings Plans and Reserved Instance coverage for the Workload Prod Account — this is the highest-probability optimization lever before any architectural changes.

*Note: CloudWatch utilization metrics are not collected in this version, so idle/underutilized resource recommendations and right-sizing signals are not available. RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are also outside current collector scope — cost drivers for those resources are not detected.*

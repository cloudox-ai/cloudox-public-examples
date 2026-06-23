# FinOps View — Optimization Opportunities

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Optimization Opportunities

> **Confidence: Likely** — Spend data is unavailable for this run; all analysis is derived from discovered architecture. No dollar savings figures can be stated. Exact cost attribution requires AWS Cost Explorer or CUR.

One architecture-level candidate has been identified that warrants validation. No utilization-based right-sizing or idle-resource recommendations are available in this run (see gaps below).

### Optimization Candidates

| # | Resource | Finding | Recommended Action | Priority |
|---|----------|---------|-------------------|----------|
| 1 | `cloudox-demo-atlas-prod-pg` | Single-AZ RDS instance — no Multi-AZ standby configured. A zone failure could cause downtime or data loss. | Evaluate enabling a Multi-AZ standby for resilience. | 4 |

**`cloudox-demo-atlas-prod-pg`** (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`) is a production database instance running without a Multi-AZ standby. This is flagged primarily as a **resilience gap**, not a cost-saving opportunity — enabling Multi-AZ would *increase* spend (a standby instance incurs additional charges). The FinOps relevance is the inverse: the current single-AZ configuration avoids that cost, but at the expense of availability and recovery posture. Finance and engineering should align on whether the risk trade-off is intentional before treating the current configuration as a cost optimisation.

*Requires validation before any action is taken.*

### Estimated Impact

No spend data is available for this run, so no savings estimates or cost impact figures can be provided. The architectural finding above carries a **resilience impact** (reduced availability / recovery posture) rather than a direct savings opportunity. Enabling Multi-AZ on `cloudox-demo-atlas-prod-pg` would represent an incremental cost increase, not a reduction.

The following gaps limit the completeness of this section and should be addressed to unlock a fuller optimization picture:

- **Cost Explorer is disabled** (`CLOUDOX_COST__ENABLED=false`): no spend figures, trending, or cost-per-service breakdowns are available.
- **CloudWatch utilization metrics are not collected**: idle resources, underutilized instances, and right-sizing candidates based on actual usage cannot be identified in this version.
- **Several resource attributes are not captured** by current collectors (RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, S3 storage classes), so cost drivers for those services are not detected.
- **761 resources carry no Environment / Stage / Tier tag** and rely on inference for classification, which may affect the accuracy of environment-level cost attribution.

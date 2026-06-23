# Generic View — Top Findings & Recommendations

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Top Findings & Recommendations

The most actionable finding across this environment is a resilience gap in the production database tier. One architectural concern has been identified; spend-based findings are unavailable because Cost Explorer collection is disabled for this run.

> **Confidence: Likely** — findings are derived from discovered architecture; some resource attributes and cost signals are not available.

### Top Findings

#### Single-AZ Datastore: `cloudox-demo-atlas-prod-pg`

The RDS DB instance **cloudox-demo-atlas-prod-pg** is deployed without a Multi-AZ standby. In the event of an Availability Zone failure, this instance would be unavailable until AWS completes a recovery to a new zone — a process that is neither instant nor guaranteed to meet tight RTO/RPO targets. This is classified as a **modernization opportunity** (priority 4) with a verified confidence level.

**Impact:** Reduced availability and recovery posture. A zone failure could cause downtime or data loss for any workload depending on this database.

---

The following known gaps limit the completeness of this findings set:

| Gap | Effect on findings |
|---|---|
| Cost Explorer collection disabled | No spend-based findings; cost driver analysis is architectural only |
| No Security Hub enablement discovered | Managed security findings (GuardDuty, Inspector, etc.) are not surfaced |
| CloudWatch utilization metrics not collected | Idle/underutilized resource recommendations are not available |
| 761 resources lack Environment/Stage/Tier tags | Classification relies on inference; some findings may be mis-scoped |
| RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, S3 storage classes not captured | Cost drivers for these services are not detected |

### Recommended Actions

1. **Enable Multi-AZ for `cloudox-demo-atlas-prod-pg`** — Evaluate enabling a Multi-AZ standby replica for this DB instance. This is the single verified architectural action identified in this run. The decision does not require immediate escalation (no immediate decision flag), but should be reviewed against the workload's availability requirements before the next maintenance window.

2. **Enable Cost Explorer collection** — Re-run discovery with cost collection enabled to surface spend-based findings, idle resource candidates, and right-sizing opportunities. The current analysis cannot quantify financial impact.

3. **Investigate Security Hub enablement** — No Security Hub activation was discovered. Enabling it (along with GuardDuty and AWS Config) would provide a managed findings layer that CloudoX can incorporate in future runs.

4. **Apply resource tagging** — 761 resources are untagged for Environment/Stage/Tier. Consistent tagging improves finding accuracy, cost allocation, and blast-radius scoping.

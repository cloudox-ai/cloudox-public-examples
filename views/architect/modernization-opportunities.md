# Architect View — Modernization Opportunities

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Modernization Opportunities

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain. Spend data is unavailable for this run; all analysis is architecture-derived only.

The single confirmed modernization candidate in this section is a resilience gap in the production data tier. No cost-based optimization candidates were identified from the discovered architecture (spend collection is disabled; utilization metrics are not collected in this version).

### Modernization Candidates

One item warrants architect attention:

| Resource | Issue | Impact | Priority |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | Single-AZ DBInstance — no Multi-AZ standby configured | Reduced availability / recovery posture | 4 |

**`cloudox-demo-atlas-prod-pg`** is a production RDS DBInstance running without a Multi-AZ standby (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`). In its current configuration, a single Availability Zone failure could result in unplanned downtime or data loss for the production workload. This is a verified finding against the resource itself, though account and region metadata for the instance were not resolved in this discovery run.

This finding sits alongside the production DynamoDB table `cloudox-demo-atlas-prod-items` (account `122122642149`, `eu-central-1`) and the production API Gateway endpoint `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` — both of which are in the same environment tier. A zone failure affecting `cloudox-demo-atlas-prod-pg` would likely impact the broader production stack that depends on it.

> **Note:** 761 resources have no Environment / Stage / Tier tag and rely on inference for classification. The production boundary drawn here is based on inferred tagging and may not be complete.

### Recommended Improvements

**Enable Multi-AZ standby on `cloudox-demo-atlas-prod-pg`.** AWS RDS Multi-AZ provides synchronous replication to a standby instance in a second AZ, with automatic failover typically completing in 60–120 seconds. For a production datastore, this is the standard resilience baseline.

- **Action:** Evaluate enabling Multi-AZ on `cloudox-demo-atlas-prod-pg`. This can be applied to a running instance with a brief maintenance window or during the next scheduled maintenance period.
- **Design consideration:** Confirm whether the application connection layer (likely via the production API Gateway endpoint) uses the RDS-provided DNS endpoint rather than a hardcoded IP — a prerequisite for transparent failover.
- **Cost note:** Multi-AZ doubles the instance-hour cost for the DBInstance. Exact cost impact is not available (spend collection disabled), but this should be weighed against the recovery time objective (RTO) and recovery point objective (RPO) requirements for the production workload.

**Gaps limiting further recommendations:**
- CloudWatch utilization metrics are not collected in this version, so idle, underutilized, or right-sizing opportunities cannot be assessed.
- RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are outside current collector scope — cost drivers for those resource types are not detected.
- Without spend data, no dollar-value prioritization of modernization work is possible from this run.

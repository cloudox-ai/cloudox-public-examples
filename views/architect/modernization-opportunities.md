# Architect View — Modernization Opportunities

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Modernization Opportunities

The single confirmed modernization signal in this discovery run is a resilience gap in the production database tier. Addressing it is low-friction relative to its availability impact, making it the highest-priority architectural action available from current evidence.

### Modernization Candidates

One candidate was identified with **Verified** confidence:

| Resource | Issue | Impact | Priority |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | Single-AZ RDS DBInstance | Reduced availability / recovery posture | 4 |

**`cloudox-demo-atlas-prod-pg`** (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`) is a production RDS instance deployed in a single Availability Zone. A zone-level failure would result in downtime and potential data loss with no automatic failover path. Given its placement in the `atlas-prod` account alongside the DynamoDB table `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`, this instance is likely part of a production workload where availability expectations are higher than a single-AZ posture can satisfy.

No further modernization candidates are present in the current evidence package. The broader inventory — 807 resources across 7 accounts — may contain additional candidates, but CloudWatch utilization metrics are not collected in this version, so idle-resource, right-sizing, or compute-modernization signals (e.g., graviton migration, serverless offload) cannot be derived from current data.

> **Confidence note:** The single-AZ finding is Verified. The absence of other candidates reflects the limits of the current collection scope, not a clean bill of health.

### Recommended Improvements

**Enable Multi-AZ on `cloudox-demo-atlas-prod-pg`.**
Activating Multi-AZ creates a synchronous standby replica in a second AZ and enables automatic failover, directly closing the availability gap. This is an in-place RDS configuration change and does not require re-architecture of the surrounding workload. Evaluate during a low-traffic window, as the promotion step involves a brief I/O pause.

**Dependency consideration:** Before enabling Multi-AZ, confirm that any application layer connecting to this instance (likely within the `atlas-prod` account) uses the RDS endpoint DNS name rather than a hardcoded IP, so failover is transparent.

**Deferred opportunities (data gaps):** The following modernization dimensions cannot be assessed until collection gaps are closed:

- *Right-sizing / idle resources* — CloudWatch utilization metrics are not collected; no usage-based recommendations are possible.
- *DynamoDB capacity mode optimisation* — capacity mode is not captured by current collectors (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items` and peers).
- *RDS IOPS and read-replica patterns* — not captured; over-provisioned IOPS or missing read replicas cannot be detected.
- *S3 storage class tiering* — not captured; Intelligent-Tiering or lifecycle opportunities are unknown.
- *Tagging coverage* — 761 of 807 resources carry no Environment/Stage/Tier tag, which limits workload-scoped analysis and makes it harder to target modernization efforts by environment.

Enabling cost collection (`CLOUDOX_COST__ENABLED`) and CloudWatch metric collection in a future run would unlock spend-attributed and utilisation-driven recommendations across the full estate.

# Executive View — Recommended Next Actions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Recommended Next Actions

### Near-Term Actions

One verified architectural risk warrants attention before it becomes an incident: the production database **cloudox-demo-atlas-prod-pg** is running in a single Availability Zone with no standby. A zone failure would cause unplanned downtime and potential data loss for whatever workload depends on this datastore.

| Priority | Resource | Risk | Recommended Action |
|---|---|---|---|
| High | cloudox-demo-atlas-prod-pg | Zone failure → downtime / data loss | Evaluate enabling Multi-AZ standby |

**Decision needed from engineering leadership:** Confirm whether the availability and recovery requirements for `cloudox-demo-atlas-prod-pg` justify enabling a Multi-AZ standby replica. This is an architectural change with cost implications; the trade-off should be made explicitly rather than by default.

> **Confidence: Likely.** This finding is based on discovered architecture. Spend impact cannot be quantified at this time — cost collection is currently disabled for this workspace. Security Hub enablement status is also unknown, which limits visibility into additional security findings that may exist.

**Additional gaps to be aware of:**
- Security Hub is not enabled (or not discovered), reducing centralised security signal coverage.
- Cost data is unavailable; spend-based prioritisation and right-sizing recommendations cannot be made until cost collection is enabled.
- CloudWatch utilisation metrics are not collected in this version, so idle or over-provisioned resource recommendations are not available.

# Executive View — Recommended Next Actions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Recommended Next Actions

### Near-Term Actions

One confirmed architectural risk warrants attention before it becomes an incident: the production database **cloudox-demo-atlas-prod-pg** is running in a single Availability Zone. A zone-level failure would cause downtime and potential data loss with no automatic failover. Enabling Multi-AZ is the recommended remediation (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`).

| Item | Risk | Recommended Action | Decision Required? |
|---|---|---|---|
| **cloudox-demo-atlas-prod-pg** — Single-AZ production database | Downtime / data loss on zone failure | Evaluate enabling Multi-AZ | No — engineering team can proceed |

Two additional areas surfaced by discovery should be tracked by leadership, though they require further investigation before action:

- **Internet-exposed security groups** — 2 security groups are open to the internet (`sg-0459201826f8de5b3`, `sg-06f2b4190bf01d261`). Engineering should confirm whether this exposure is intentional and document the justification.
- **No Security Hub enablement detected** — There is no evidence that AWS Security Hub is active. This is a gap in centralised security posture visibility and worth a deliberate decision on whether to enable it.

**Cost visibility is currently unavailable.** Cost Explorer collection is disabled in this environment, so no spend figures or savings estimates can be provided. Enabling cost collection is recommended before the next review cycle to support data-driven prioritisation.

> **Confidence: Likely** — Findings are derived from discovered architecture. Spend data, CloudWatch utilisation metrics, and several RDS/DynamoDB cost drivers are not available in this run; the full risk and cost picture may be broader than shown here.

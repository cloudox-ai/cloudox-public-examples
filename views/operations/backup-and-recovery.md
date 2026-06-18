# Operations View — Backup & Recovery Signals

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Backup & Recovery Signals

The most operationally significant finding here is a single-AZ database instance that represents a concrete availability and recovery risk. Of 807 resources captured, no broad coverage gaps were flagged — but two meta-collector limitations mean the full AWS resource breadth could not be independently cross-checked (see Unknowns below).

### Backup Signals

No backup-specific signals (e.g., snapshot schedules, retention policies, or backup job status) are present in the available evidence for this section. Coverage of backup configuration details is limited by the collector gaps noted below.

### Recovery Readiness

One recovery-readiness issue was identified:

| Resource | Issue | Impact | Recommended Action |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | Not Multi-AZ | Reduced availability / recovery posture | Evaluate enabling Multi-AZ for resilience |

**`cloudox-demo-atlas-prod-pg`** (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`) is a DBInstance running in a single Availability Zone. A zone-level failure could cause unplanned downtime or data loss with no automatic failover. This is classified as a modernization opportunity (priority 4) rather than an immediate incident, but it is a meaningful gap in recovery posture for a production database.

**What can break:** A failure of the AZ hosting `cloudox-demo-atlas-prod-pg` would take the instance offline. Without Multi-AZ, there is no standby replica to promote automatically.

**What requires operational attention:** Evaluate whether the workload's RTO/RPO tolerances are compatible with single-AZ operation. If not, enabling Multi-AZ (or an equivalent HA mechanism) should be scheduled.

**Recovery risk:** Manual recovery from a snapshot or backup would be required following a zone failure, extending downtime beyond what Multi-AZ standby promotion would provide.

> **Coverage note:** Resource Explorer and Cloud Control meta-collectors were unavailable during discovery. Long-tail resource types and full AWS-visible inventory breadth could not be cross-checked. Additional resources with backup or recovery gaps may exist but are not visible in this dataset.

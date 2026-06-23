# Operations View — Backup & Recovery Signals

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Backup & Recovery Signals

The most operationally significant finding here is a single-AZ database deployment with no standby — meaning a zone failure directly translates to downtime or potential data loss with no automatic failover path.

### Backup Signals

No backup configuration signals (snapshot schedules, retention policies, or backup vault associations) were found in this section's package for any resource. This is a coverage gap, not a confirmed absence — two meta-collectors that would broaden resource visibility were unavailable during discovery (see unknowns below), so backup configuration data for resources outside typed collector coverage may be missing.

### Recovery Readiness

One recovery posture issue was identified:

| Resource | Issue | Impact | Recommended Action |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | No Multi-AZ standby configured | Reduced availability; zone failure can cause downtime or data loss | Evaluate enabling a Multi-AZ standby for resilience |

**`cloudox-demo-atlas-prod-pg`** (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`) is a DBInstance deployed in a single Availability Zone with no Multi-AZ standby. In the event of an AZ-level failure, there is no automatic promotion of a standby replica — recovery would depend on restoring from a snapshot or manual intervention, both of which introduce meaningful RTO and RPO exposure.

This is flagged as a modernization opportunity (priority 4) rather than an active incident, but it represents a structural gap in the recovery posture that warrants evaluation, particularly for a production workload (as the `-prod-` naming suggests).

> **Coverage note:** Resource Explorer and Cloud Control meta-collectors were both unavailable during this discovery run. Long-tail resource types and a full cross-account breadth check could not be performed. Recovery readiness signals for resource types outside typed collector coverage may be incomplete.

# Operations View — Critical Components

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Critical Components

The most operationally significant finding in this section is a **single-AZ database instance sitting in the critical path of the production API**. A zone failure would take the production data tier offline with no automatic failover. This warrants immediate resilience review.

### Critical Workloads

The following workloads are identified as key operational entities. The production workload carries the highest risk profile given its confirmed dependency concern.

| Workload | Account | Region | Confidence |
|---|---|---|---|
| Cloudox Demo Atlas Prod API (`cloudox-demo-atlas-prod-api`) | 122122642149 | eu-central-1 | Likely |
| Cloudox (`cloudox`) | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Dev API (`cloudox-demo-atlas-dev-api`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev (`cloudox-demo-atlas-dev`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch (`cloudox-demo-sandbox-scratch`) | 161388682021 | eu-central-1 | Assumed |

**Cloudox Demo Atlas Prod API** is the workload directly exposed to the resilience risk described below. The dev and sandbox workloads are noted for completeness; their risk profiles are lower, but their environment classification relies on inference rather than explicit tagging (see Unknowns).

### Shared Dependencies

**`cloudox-demo-atlas-prod-pg` — Single-AZ DBInstance (Priority 3)**

Intelligence item `dependency_concern:architecture:cloudox-demo-atlas-prod-pg` identifies that **Cloudox Demo Atlas Prod API** (`cloudox-demo-atlas-prod-api`, account 122122642149, eu-central-1) depends on the database instance `cloudox-demo-atlas-prod-pg`, which is **not configured for Multi-AZ**.

> *"A workload depends on datastore 'cloudox-demo-atlas-prod-pg', which is not Multi-AZ; a zone failure would take the dependent workload's data tier offline."*

**What can break:** A single Availability Zone failure in eu-central-1 would make `cloudox-demo-atlas-prod-pg` unavailable, taking the production API's data tier offline for the duration of the outage. There is no automatic failover path evidenced.

**What requires operational attention:**
- Confirm whether Multi-AZ is enabled or intentionally disabled on `cloudox-demo-atlas-prod-pg`.
- Verify that automated backups are configured and that the recovery point objective (RPO) is acceptable.
- Review how **Cloudox Demo Atlas Prod API** handles a data-tier failure — does it degrade gracefully, queue writes, or fail hard?

**Recovery risk:** Without Multi-AZ, recovery from a zone failure requires a restore from backup or a manual promotion, both of which introduce recovery time that may exceed production SLAs. The account ID for `cloudox-demo-atlas-prod-pg` itself could not be confirmed from available evidence — traceability to the owning account should be established before remediation is planned.

No equivalent dependency concerns are evidenced for the dev (`cloudox-demo-atlas-dev-api`) or sandbox (`cloudox-demo-sandbox-scratch`) workloads within this section's scope.

![Workload architecture](./diagrams/operations-workload-architecture.png)

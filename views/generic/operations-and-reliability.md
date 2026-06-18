# Generic View — Operations & Reliability

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Verified_

## Operations & Reliability

The environment is broadly well-captured: 807 resources are recorded across 15 VPCs and 63 subnets, with no coverage gaps identified. A single load balancer is present, and 25 internet-facing access paths have been observed — a figure worth keeping in mind when reviewing exposure and blast radius.

### Reliability Signals

The networking fabric spans **15 VPCs** and **63 subnets**, providing the structural segmentation that underpins availability and fault isolation. One load balancer is in use, distributing traffic within this topology.

Two AWS-managed prefix lists are referenced in the environment (`pl-66a5400f` and `pl-6ea54007`, both in `eu-central-1`), indicating that at least some security group or routing rules rely on AWS-maintained IP ranges — a pattern that reduces manual maintenance burden for those specific rules.

**25 internet-facing access paths** are observed. This is the primary reliability-adjacent signal to scrutinise: each path represents a potential ingress point that must remain intentional, correctly routed, and protected against disruption or misconfiguration.

### Operational Concerns

Two meta-collection gaps reduce the completeness of this picture and should be treated as material unknowns:

- **Resource Explorer was disabled or unavailable.** Without it, the full breadth of AWS-visible resources could not be cross-checked against what typed collectors found. Resources in less-common services or regions may be absent from the inventory.
- **Cloud Control was disabled.** Long-tail resource types — those not covered by dedicated typed collectors — are not represented. Operational tooling, custom configurations, or newer AWS resource types may be missing.

The practical consequence is that the **807 resources captured represent a lower bound**, not a confirmed total. Operational runbooks, alerting, and change-management processes that rely on this inventory should account for the possibility of uncatalogued resources until both meta-collectors can be enabled.

> **Confidence: Verified** — figures and signals above are derived from complete graph evidence for the networking and coverage domains. The unknowns relate to deliberate collection scope, not data quality within what was collected.

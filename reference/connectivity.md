> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Connectivity Reference

Classified connectivity links (VPC peering + security-group-permitted reachability). Each row states its evidence basis and completeness so it is never read as full end-to-end reachability.

_Displaying all **1** row(s) (no rows omitted)._

_Sorted by connectivity class, then source (stable, not by risk)._

| Source Resource | Source Type | Destination Resource | Destination Type | Connectivity Class | Evidence Basis | Completeness | Confidence | Evidence |
|---|---|---|---|---|---|---|---|---|
| cloudox-demo-atlas-prod-api | AWS::ECS::Service | cloudox-demo-atlas-prod-pg | AWS::RDS::DBInstance | Security-group-permitted reachability | Derived from a security-group allow rule (secured_by + allows_from) | Network reachability only — NOT a confirmed data flow or read/write path. | Likely | `cloudox-demo-atlas-prod-api`, `cloudox-demo-atlas-prod-pg` |

**Coverage notes**

- This table does NOT assert end-to-end reachability. Security-group-permitted rows are network reachability inferred from allow rules — not a confirmed data flow, and not proof routes or NACLs also permit the path.
- Other connectivity kinds are reported elsewhere: route-level in the Network Routing reference, internet-facing in Public Exposure, and application dependency in Service Dependencies.
- Port/protocol of derived connectivity is not resolved; an absent source→destination pair means the path is unknown / not evidenced here, never that connectivity was disproven.

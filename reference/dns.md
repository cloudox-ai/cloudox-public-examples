> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# DNS Zones Reference

Route 53 HOSTED ZONES only (zone-level summary): per-type record counts and VPC associations. Individual DNS records are NOT collected, so this is not a record/route export and resolves no specific name.

_Displaying all **1** row(s) (no rows omitted)._

_Sorted alphabetically by hosted zone name (stable, not by risk)._

| Hosted Zone | Public/Private | Record Count | Record Types | Associated VPCs | Confidence | Evidence |
|---|---|---|---|---|---|---|
| atlas.internal.demo | Private | 2 | NS:1, SOA:1 | — | Verified | `Z09280431A90P7LBIX0FB` |

**Coverage notes**

- Zones only: individual DNS records (A / CNAME / ALIAS / TXT / …) are NOT collected — only per-type counts per zone. This table cannot tell you what any specific name resolves to.
- Non-Route 53 DNS and external registrars are out of scope.

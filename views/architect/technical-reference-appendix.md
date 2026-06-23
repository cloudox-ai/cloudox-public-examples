# Architect View — Technical Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Unknown_

## Technical Reference Appendix

This appendix provides a stable friendly-name → raw-identifier mapping for every key entity referenced across this view. Use it to correlate prose references with console lookups, IaC state, or API calls.

### Entity Reference

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Note — Log Archive Account (`122980216815`):** Confidence is *Likely*, not Verified. Treat its role and membership in the organization as provisionally confirmed until corroborated by a direct account inventory or AWS Organizations API response.

#### Security Groups

| Friendly Name | Resource ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |

#### Subnets

| Friendly Name | Subnet ID | Account | Region | Confidence |
|---|---|---|---|---|
| Public Subnet (eu-central-1) | `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |

> **Naming note:** Multiple subnets share the display name "Public Subnet (eu-central-1)" across different accounts. Always use the raw subnet ID (`subnet-0…`) to disambiguate in IaC, CDK, or Terraform references.

### Evidence

No citeable evidence references are available for this section. The entity records above are drawn from the discovered inventory included in this view's context package. For deeper provenance — such as the specific AWS Config snapshots, CloudTrail events, or API calls that produced each record — refer to the raw discovery run data or request an evidence export from CloudoX.

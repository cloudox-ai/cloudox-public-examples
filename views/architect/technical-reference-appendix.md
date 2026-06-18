# Architect View — Technical Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Unknown_

## Technical Reference Appendix

This appendix provides a stable friendly-name → raw-identifier mapping for all key entities referenced across this view. Use it to correlate prose references with console lookups, IaC state, or API queries.

### Entity Reference

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Note — Log Archive Account (`122980216815`):** Confidence is **Likely**, not Verified. Treat its role and membership in the AWS Organization as unconfirmed until directly validated in the management console or via AWS Organizations API.

#### Security Groups

| Friendly Name | Resource ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |

#### Subnets

| Subnet ID | Account | Region | Confidence |
|---|---|---|---|
| `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |

All five subnets carry the friendly name **Public Subnet (eu-central-1)**; use the raw subnet ID to distinguish them in routing tables, security group rules, and IaC references.

### Evidence

No evidence references are available in this section's context package. Identifier mappings above are sourced directly from CloudoX discovery metadata.

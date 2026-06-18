# Generic View — Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Unknown_

## Reference Appendix

This appendix provides a stable mapping between the friendly names used throughout this view and the underlying raw identifiers, enabling traceability back to source resources.

### Entity Reference

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Confidence note:** The Log Archive Account (`122980216815`) is rated **Likely** — treat its presence and role as probable but not fully confirmed by direct evidence.

#### Subnets

| Friendly Name | Subnet ID | Account | Region | Confidence |
|---|---|---|---|---|
| Public Subnet (eu-central-1) | `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |

#### Other Resources

| Friendly Name | Resource ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |

### Evidence

No evidence references are available for this section. The entity mappings above are drawn directly from discovered resource metadata; no additional supporting evidence items are citeable for this appendix.

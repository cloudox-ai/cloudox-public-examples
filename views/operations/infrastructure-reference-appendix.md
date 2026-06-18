# Operations View — Infrastructure Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Unknown_

## Infrastructure Reference Appendix

This appendix provides a stable friendly-name → raw-identifier mapping for the entities observed in this environment. Use it to correlate alerts, logs, and IaC references against cloud-provider identifiers. No intelligence items were available to populate a deeper inventory narrative for this section — the tables below represent the complete closed-world evidence.

### Infrastructure Inventory

No evidence found for a structured service or resource inventory beyond the key entities listed below. Coverage gaps (observability instrumentation, backup configuration, runtime state) cannot be assessed from the available package.

### Entity Reference

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Note — Log Archive Account (`122980216815`):** Confidence is *Likely*, not Verified. Treat references to this account in runbooks with caution until ownership and purpose are confirmed.

#### Subnets (eu-central-1)

| Friendly Name | Subnet ID | Account | Confidence |
|---|---|---|---|
| Public Subnet (eu-central-1) | `subnet-016c22941a019a137` | Workload Prod (`122122642149`) | Verified |
| Public Subnet (eu-central-1) | `subnet-0f64d71c952a7898a` | Workload Dev (`105769365151`) | Verified |
| Public Subnet (eu-central-1) | `subnet-065f522206524ab12` | Workload Dev (`105769365151`) | Verified |
| Public Subnet (eu-central-1) | `subnet-013a24d318ed6f3d0` | Workload Prod (`122122642149`) | Verified |
| Public Subnet (eu-central-1) | `subnet-029e2cceb3d0beff7` | Sandbox Ma (`161388682021`) | Verified |

All observed subnets are in **eu-central-1**. No evidence found for subnets in other regions. Multiple subnets share the friendly name "Public Subnet (eu-central-1)" — use the raw subnet ID for unambiguous identification in runbooks and automation.

#### Security Groups

| Friendly Name | Security Group ID | Account | Confidence |
|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma (`161388682021`) | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod (`122122642149`) | Verified |

**Operational callout — `sg-054446d655de1ee7f` (Cloudox Demo Sandbox Sg Permissive):** The name explicitly signals permissive rules. Confirm this group is scoped strictly to the Sandbox Ma account and is not referenced by Workload Prod or Workload Dev resources. No rule-level detail is available in this package to verify blast radius.

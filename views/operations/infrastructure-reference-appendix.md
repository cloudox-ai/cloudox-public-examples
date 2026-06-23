# Operations View — Infrastructure Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Unknown_

## Infrastructure Reference Appendix

This appendix provides a stable friendly-name → raw-identifier mapping for the entities referenced across this Operations view. Use it to correlate console URLs, CLI commands, IaC references, and alert payloads back to the correct resource without ambiguity.

> **Confidence: Unknown.** The entity list is derived from discovery data; completeness of coverage across all accounts and regions cannot be confirmed from this section's package alone. Treat this as the known set, not necessarily the full set.

### Infrastructure Inventory

No evidence of detailed resource inventory (instance types, sizes, configurations, or counts) was present in this section's package. The entity reference below captures the discrete identifiers that are known. Deeper per-resource inventory is covered in other sections of this view.

### Entity Reference

The following accounts, subnets, and security groups were identified. All resources are located in **eu-central-1** where a region is applicable.

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Note:** The Log Archive Account (`122980216815`) is marked **Likely** — verify its existence and role assignment before relying on it for log retention or audit workflows.

#### Subnets

| Friendly Name | Subnet ID | Account | Region |
|---|---|---|---|
| Public Subnet (eu-central-1) | `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 |

> **Operational note:** All five known subnets are typed as **public**. No private subnets appear in this section's package. If workloads are expected to run in private subnets, verify subnet coverage in the networking section of this view.

#### Security Groups

| Friendly Name | Security Group ID | Account | Region |
|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 |

> **Operational attention — `sg-054446d655de1ee7f`:** The name **Permissive** is a strong indicator of broad ingress or egress rules. Review this security group's rule set before any production-adjacent activity in the Sandbox Ma Account. Permissive rules in a sandbox can become a lateral-movement path if VPC peering or shared services exist.

> **Operational attention — `sg-06f2b4190bf01d261`:** This security group is associated with an ALB in the Workload Prod Account. Changes to its rules directly affect production traffic ingress. Treat modifications as change-controlled.

# Security View — Evidence Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Unknown_

## Evidence Appendix

> **Confidence: Unknown** — No intelligence items or provider-native evidence references were available for this section. The entity reference below is derived from discovery metadata only. Treat this appendix as a traceability index, not a validated evidence set.

This appendix lists the key security entities identified across the in-scope accounts and provides a reference point for tracing findings discussed elsewhere in this view back to their source resources. No provider-native evidence items (e.g. AWS Config rules, Security Hub findings, CloudTrail records) were present in the package for this section.

### Entity Reference

The following entities were identified during discovery. Confidence ratings reflect the reliability of the entity identification itself, not any associated security assessment.

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Note on Log Archive Account (`122980216815`):** Confidence is rated **Likely** rather than Verified. Security & Governance teams should confirm this account's role and ensure log delivery and access controls are validated independently.

#### Security Groups

| Friendly Name | Resource ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |

> The name **Cloudox Demo Sandbox Sg Permissive** (`sg-054446d655de1ee7f`) warrants attention from a governance perspective — the "Permissive" designation in the resource name suggests broad ingress or egress rules. Security teams should cross-reference this entity with the network exposure findings covered in the relevant section of this view.

#### Public Subnets

Five public subnets were identified across three accounts, all in `eu-central-1`.

| Subnet ID | Account | Region | Confidence |
|---|---|---|---|
| `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |

All five subnets are classified as public. Security & Governance teams should verify that resources placed in these subnets are intentionally internet-reachable and that appropriate security group and network ACL controls are in place.

### Evidence

No provider-native evidence references (e.g. Security Hub findings, Config rule evaluations, CloudTrail events) were present in this section's package. This is a material gap for a security view — the entities above cannot be assessed for compliance posture, misconfiguration, or active findings from this section alone.

Governance teams should treat the absence of evidence references here as a signal to validate that:
- AWS Security Hub (or equivalent) is enabled and aggregating findings across all four accounts.
- AWS Config is recording in all relevant regions and accounts.
- Log delivery to the Log Archive Account (`122980216815`) is confirmed and auditable.

Findings and evidence discussed in other sections of this view should be cross-referenced against the entity identifiers listed above.

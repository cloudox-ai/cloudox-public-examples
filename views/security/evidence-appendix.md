# Security View — Evidence Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Unknown_

## Evidence Appendix

> **Confidence: Unknown** — No intelligence items or evidence references were available for this section. The entity reference table below is drawn from verified discovery metadata only. No provider-native evidence records were present in this package.

This appendix lists the key security entities identified across the in-scope accounts and regions. These entities are referenced throughout the Security view. No supporting evidence items or evidence references were available to cite for this section; governance and security teams should treat the absence of linked evidence as a material gap requiring direct validation.

### Entity Reference

The following entities were identified during discovery. Confidence ratings reflect the reliability of the entity identification itself, not any security assessment.

| Friendly Name | Type | Account ID | Region | Identifier | Confidence |
|---|---|---|---|---|---|
| Management Account | Account | 110319895932 | — | `110319895932` | Verified |
| Sandbox Ma Account | Account | 161388682021 | — | `161388682021` | Verified |
| Workload Dev Account | Account | 105769365151 | — | `105769365151` | Verified |
| Workload Prod Account | Account | 122122642149 | — | `122122642149` | Verified |
| Log Archive Account | Account | 122980216815 | — | `122980216815` | Likely |
| Cloudox Demo Sandbox Sg Permissive | Resource (Security Group) | 161388682021 | eu-central-1 | `sg-054446d655de1ee7f` | Verified |
| Cloudox Demo Atlas Prod Sg ALB | Resource (Security Group) | 122122642149 | eu-central-1 | `sg-06f2b4190bf01d261` | Verified |
| Public Subnet (eu-central-1) | Subnet | 122122642149 | eu-central-1 | `subnet-016c22941a019a137` | Verified |
| Public Subnet (eu-central-1) | Subnet | 105769365151 | eu-central-1 | `subnet-0f64d71c952a7898a` | Verified |
| Public Subnet (eu-central-1) | Subnet | 105769365151 | eu-central-1 | `subnet-065f522206524ab12` | Verified |
| Public Subnet (eu-central-1) | Subnet | 122122642149 | eu-central-1 | `subnet-013a24d318ed6f3d0` | Verified |
| Public Subnet (eu-central-1) | Subnet | 161388682021 | eu-central-1 | `subnet-029e2cceb3d0beff7` | Verified |

**Notes for Security & Governance teams:**

- The **Log Archive Account** (`122980216815`) carries only a *Likely* confidence rating on its identification — its role as a log archive account should be confirmed directly before relying on it for audit or compliance purposes.
- Two security groups are surfaced as key entities: `sg-054446d655de1ee7f` (named *Permissive* in the Sandbox account) and `sg-06f2b4190bf01d261` (ALB security group in the Workload Prod account). Their naming and placement are noted here for traceability; refer to the relevant findings sections of this view for any associated risk assessments.
- All subnets identified are in the **eu-central-1** region and are labelled as public subnets across three accounts (Sandbox, Workload Dev, Workload Prod).

### Evidence

No evidence found for this subsection — no provider-native evidence references were present in the package for this section. Security & Governance teams should validate entity configurations, policy attachments, and audit log coverage directly against the provider console or API outputs.

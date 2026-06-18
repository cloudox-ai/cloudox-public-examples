# Generic View — Top Findings & Recommendations

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Top Findings & Recommendations

The environment spans **807 resources across 7 accounts**, with the most actionable finding being a resilience gap on a production database. Security posture carries two internet-exposed security groups and a large IAM role surface worth monitoring. Spend analysis is unavailable for this run, so cost findings are limited to architectural observations.

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain (see below).

### Top Findings

#### Resilience: Single-AZ Production Database

The RDS instance **`cloudox-demo-atlas-prod-pg`** is deployed in a single Availability Zone. A zone-level failure would cause downtime and potential data loss for whatever workload depends on this database. This is the highest-priority architectural finding in the current dataset.

- **Affected resource:** `cloudox-demo-atlas-prod-pg`
- **Impact:** Reduced availability and recovery posture
- **Finding ID:** `modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`

#### Security: Internet-Exposed Security Groups

Two security groups are open to the internet:

| Security Group | Account |
|---|---|
| `sg-0459201826f8de5b3` | 122122642149 |
| `sg-06f2b4190bf01d261` | 122122642149 |

The resources protected (or exposed) by these groups, and whether the inbound rules are intentional, are not detailed in the available evidence. They warrant review.

A third security group (`sg-054446d655de1ee7f`, account 161388682021) is present in the evidence set but its internet-exposure status is not confirmed from the available data.

#### IAM: Large Role Surface, No Customer-Managed Policies

The environment contains **72 IAM roles** and **0 customer-managed policies**. Notable roles include:

- `cloudox-demo-sandbox-unused-admin` (account 161388682021) — the name suggests an administrative role that may no longer be in active use.
- `OrganizationAccountAccessRole` (account 161388682021) — a standard cross-account access role; its scope of use is not confirmed.
- `cloudox-demo-sandbox-scratch-lambda` (account 161388682021) — a Lambda execution role in what appears to be a sandbox account.
- `cloudox-demo-org-trail-logs` and `AWSServiceRoleForCloudFormationStackSetsOrgAdmin` (account 110319895932) — supporting org-level CloudTrail and StackSets respectively.

The absence of customer-managed policies means permissions are likely defined inline or via AWS-managed policies, which can make least-privilege auditing harder.

#### Observability: No Security Hub Evidence

No Security Hub enablement was discovered. Without it, findings from GuardDuty, Inspector, and Config are not aggregated into a single security posture view.

#### Tagging: Majority of Resources Untagged

**761 of 807 resources** (≈94%) have no `Environment`, `Stage`, or `Tier` tag. CloudoX has inferred classification for these resources, but the lack of authoritative tags limits cost allocation, access control by tag, and operational filtering.

### Recommended Actions

| Priority | Domain | Action | Affected Resource / Scope |
|---|---|---|---|
| 1 | Architecture | Evaluate enabling Multi-AZ on `cloudox-demo-atlas-prod-pg` to eliminate single-point-of-failure risk. | `cloudox-demo-atlas-prod-pg` |
| 2 | Security | Review `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261` — confirm whether internet-open inbound rules are intentional and scope them to known CIDRs or prefixes if not. | Account 122122642149 |
| 3 | Security | Audit `cloudox-demo-sandbox-unused-admin` — if the role is no longer needed, remove it to reduce standing privilege. | Account 161388682021 |
| 4 | Security | Evaluate enabling AWS Security Hub across accounts to centralise security findings. | Org-wide |
| 5 | Operations | Implement consistent `Environment` / `Stage` / `Tier` tagging across all 807 resources to enable reliable cost allocation and operational filtering. | All accounts |

> **Cost note:** Cost Explorer collection is disabled for this run (`CLOUDOX_COST__ENABLED=false`). Spend figures and dollar-attributed recommendations are unavailable. The findings above are derived from discovered architecture only.

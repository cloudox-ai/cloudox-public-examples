# Security View — Validation Questions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Validation Questions

**Confidence: Likely** — derived from graph evidence; some unknowns remain.

One open question requires owner confirmation before a remediation or retention decision can be made. It does not block other findings but carries a meaningful identity-risk signal that governance teams should resolve promptly.

### Questions to Resolve

| # | Role | Account | Question | Confidence |
|---|------|---------|----------|------------|
| 1 | `cloudox-demo-sandbox-unused-admin` | `161388682021` | Does this role require its current privilege level, and who owns it? | Assumed |

**Detail — `cloudox-demo-sandbox-unused-admin` (account `161388682021`)**

The role name contains the word *unused*, which raises the question of whether it was created temporarily and never decommissioned, or whether it is an intentional standing admin role (`validation_question:security:cloudox-demo-sandbox-unused-admin`). Because the privilege level appears administrative and no ownership or purpose metadata was found, the current state cannot be confirmed as intentional.

- **What to confirm with the environment owner:** Is the administrative privilege level still required? If the role is genuinely unused, it is a candidate for removal or privilege reduction. If it is active, ownership and a business justification should be documented.
- **Why it matters:** An unowned, potentially dormant role with administrative scope in account `161388682021` represents a latent privilege-escalation path if credentials or trust relationships are ever abused.

> **Note:** No Security Hub enablement was discovered in this environment. Centralised finding aggregation and compliance scoring are therefore not available as corroborating evidence for this or other questions.

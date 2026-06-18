# Security View — Validation Questions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Validation Questions

**Confidence: Likely** — Derived from graph evidence; some unknowns remain. The item below is flagged **Assumed** and requires owner confirmation before any remediation decision.

Two broader signals from discovery frame the urgency of these questions: 72 IAM roles exist across the environment, no customer-managed policies were found, and 2 security groups are open to the internet. With no Security Hub enablement discovered, there is no automated findings layer to catch privilege drift or exposure — making manual validation of the items below more important.

### Questions to Resolve

#### IAM Role: `cloudox-demo-sandbox-unused-admin` (Account `161388682021`)

> **Validation question:** Does IAM role `cloudox-demo-sandbox-unused-admin` require its current privilege level, and who owns it?

**Confidence: Assumed.** The role name and its presence in the sandbox account (`161388682021`) suggest administrative privileges that may not be intentional or actively needed, but this has not been confirmed through policy inspection or usage data available in this package.

| Attribute | Value |
|---|---|
| Entity type | IAM Role |
| Account | `161388682021` |
| Role ID | `AROAAAAADPCL3BVEXUDTH` |
| Friendly name | `cloudox-demo-sandbox-unused-admin` |
| Decision required | No (but validation is recommended) |

**Why this matters for Security & Governance:** An administrative role whose ownership and necessity are unconfirmed represents a standing privilege escalation path. If the role is unused or unowned, it should be disabled or scoped down. If it is legitimately required, it should be documented with a named owner and reviewed under your access governance process.

**Recommended next step:** Confirm with the sandbox account owner whether this role is actively used, by whom, and whether administrative scope is justified. Check last-used data via IAM credential reports or AWS Access Analyzer if available.

---

*No evidence found for additional validation questions in this section beyond the one item above.*

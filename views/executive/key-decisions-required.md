# Executive View — Key Decisions Required

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Key Decisions Required

### Decisions for Leadership

Three areas carry enough risk or ambiguity to warrant a leadership decision now. Each is grounded in discovered architecture; spend data and utilization metrics are not available in this run, so cost-related decisions are directional rather than dollar-precise.

**1. Internet-exposed security groups — accept or remediate?**

2 security groups are open to the internet (0.0.0.0/0 or ::/0 inbound rules). These span at least two accounts. Leadership should decide whether this exposure is intentional and reviewed, or whether a remediation sprint is warranted. Until a decision is recorded, these represent unacknowledged attack surface.

> Evidence: `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261` (account 122122642149); `sg-054446d655de1ee7f` (account 161388682021).

**2. IAM posture — 72 roles, zero customer-managed policies**

The estate carries 72 IAM roles but no customer-managed policies have been discovered. This pattern suggests permissions are either inline or relying entirely on AWS-managed policies, both of which limit the ability to enforce least-privilege consistently at scale across 7 accounts. Leadership should decide whether a centralised IAM policy governance standard is needed before the role count grows further.

**3. Security Hub — enable or formally defer?**

No Security Hub enablement has been discovered across the estate. With 807 resources across 7 accounts and an org-level CloudTrail already in place (`cloudox-demo-org-trail-o-aaaapzvebq`), the foundational logging infrastructure exists. The decision is whether to activate Security Hub now to gain a consolidated findings view, or to formally defer and accept the gap in centralised security posture monitoring.

**4. Tagging and environment classification — mandate or accept inference risk**

761 of 807 resources (94%) carry no Environment / Stage / Tier tag. CloudoX is inferring classification for these resources. Without authoritative tags, cost allocation, blast-radius analysis, and change-control boundaries are unreliable. Leadership should decide whether to mandate a tagging standard and remediation window, or accept that environment-level reporting will remain approximate.

---

> **Confidence: Likely.** All findings are derived from discovered architecture. Spend figures are unavailable (cost collection is disabled). Utilization and right-sizing data are not collected in this version. Decisions marked directional should be revisited once cost and metrics collection is enabled.

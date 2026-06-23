# Executive View — Key Decisions Required

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Key Decisions Required

### Decisions for Leadership

Four gaps in observability and governance stand out and each requires a deliberate leadership decision — not just a remediation ticket.

**1. Security Hub is not enabled (or not discoverable)**
No evidence of Security Hub enablement was found across the environment. Without it, there is no consolidated security posture score, no automated standards compliance check (CIS, AWS Foundational Security), and no single pane for cross-account findings. Leadership should decide: adopt Security Hub as the security baseline now, or document an explicit alternative control.

**2. 761 resources carry no Environment / Stage / Tier tag**
Classification of these resources relies on inference rather than authoritative tagging. This directly impairs cost allocation, blast-radius analysis, and change-control decisions (e.g., "is this production?"). A tagging policy with enforcement — via AWS Config rules or Service Control Policies — needs an owner and a deadline.

**3. Cost visibility is switched off**
Cost Explorer collection is disabled (`CLOUDOX_COST__ENABLED=false`). Spend figures are therefore unavailable; the environment carries 7 identified architectural cost drivers but no dollar attribution can be confirmed. Leadership should decide whether to enable cost collection so that the next run can surface actual spend, anomalies, and optimisation candidates. Until then, financial exposure from those drivers is unquantified.

**4. No utilisation metrics — right-sizing is blind**
CloudWatch utilisation data is not collected in this version. Idle or over-provisioned resources cannot be identified from usage evidence alone. If cost optimisation is a priority, a decision is needed on how to close this gap — either through a future CloudoX capability or a parallel tooling approach.

**Existing governance signals worth noting**
The environment does have an organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`) with an associated logging role, and CloudFormation StackSets org-admin delegation is in place. These indicate some baseline governance infrastructure is active — but the gaps above limit how much assurance that infrastructure can provide without the missing controls.

> **Confidence: Likely** — findings are derived from discovered architecture. Spend figures and utilisation data are not available for this run; decisions 3 and 4 are based on confirmed collection gaps rather than cost or usage evidence.

# Executive View

> **Demo Report** — This discovery report was produced from a real AWS environment by
> [CloudoX](https://cloudox.io). AWS account IDs and all resource identifiers (VPC,
> subnet, security group, IAM role IDs, KMS key IDs, and similar) have been replaced
> with deterministic synthetic equivalents so the report can be shared publicly.
> Architecture patterns, workload structure, findings, and service configurations are
> otherwise unmodified — this is not dummy data.

---

> **Audience:** CTO / Engineering leadership  
> **Overall confidence:** Likely  
> Business and operational risk, strategic concerns, cost overview, and the decisions that need attention.

**This view answers**

- What requires leadership attention?
- What are the highest risks?
- What decisions are needed?

**Sections**

- [Executive Summary](#executive-summary)
- [What Needs Attention](#what-needs-attention)
- [Key Decisions Required](#key-decisions-required)
- [Strategic Risks](#strategic-risks)
- [Cost & Efficiency Signals](#cost-efficiency-signals)
- [Recommended Next Actions](#recommended-next-actions)
- [Important Unknowns](#important-unknowns)
- [Evidence Appendix](#evidence-appendix)


---

## Executive Summary

The environment is operational and multi-account, but carries meaningful visibility and security gaps that warrant leadership attention before they become incidents.

### The Environment in Brief

The estate spans **7 AWS accounts** — including a Management Account, a Workload Dev Account, a Workload Prod Account, a Sandbox Ma Account, and a Log Archive Account — with **807 resources** discovered across them. CloudoX inferred **15 workloads**, of which **12 are significant** (3 were demoted as helper or governance constructs). Two public API Gateway endpoints are present and internet-reachable (`gfwaiva01f` and `xdmn5ldmif`, both in eu-central-1), and DynamoDB tables back workloads in both the Development and Production environments, as well as the Sandbox.

The account structure follows a recognisable landing-zone pattern — separate accounts for production workloads, development, sandbox, and log archiving — which is a positive structural signal.

### Overall Posture

Three issues stand out at the leadership level:

| Area | Finding | Decision needed? |
|---|---|---|
| **Security visibility** | No Security Hub enablement was discovered across any account. Centralised threat and compliance findings are absent. | Yes — enable or confirm an alternative |
| **Tagging discipline** | 761 of 807 resources (94%) carry no Environment, Stage, or Tier tag. Ownership, cost allocation, and blast-radius analysis all rely on inference rather than declared metadata. | Yes — enforce a tagging policy |
| **Discovery coverage** | Resource Explorer and Cloud Control meta-collectors were unavailable during this run. Long-tail resource types and the full AWS-visible breadth could not be cross-checked, meaning the 807-resource count is a lower bound. | Yes — re-enable collectors for a complete picture |

The two internet-facing API endpoints and the production DynamoDB table (`cloudox-demo-atlas-prod-items`) represent the highest-consequence assets in scope. Without Security Hub and with incomplete tagging, the ability to detect, attribute, and respond to an incident against these assets is reduced.

> **Confidence: Likely.** Findings are derived from graph evidence. The absence of Security Hub discovery and the meta-collector gaps mean some risks may be understated. Treat this summary as directionally reliable, not exhaustive.


---

## What Needs Attention

One critical, decision-required security exposure stands out and warrants immediate leadership action: a sandbox security group is openly exposing SSH and PostgreSQL to the entire internet. Three medium-severity internet-facing exposures round out the picture and should be confirmed as intentional.

### Highest-Priority Issues

**🔴 Critical — Action Required**

| Resource | Exposure | Ports Open to Internet | Decision Needed? |
|---|---|---|---|
| cloudox-demo-sandbox-sg-permissive | World-open ingress | SSH (22), PostgreSQL (5432) | **Yes** |

The security group **cloudox-demo-sandbox-sg-permissive** (Sandbox account, eu-central-1) allows unrestricted inbound traffic from any IP address on port 22 (SSH) and port 5432 (PostgreSQL). Exposing a database port and an administrative access port directly to the internet is a high-likelihood breach vector. Engineering leadership should confirm whether this exposure is intentional and, if not, restrict ingress rules immediately. *(Confidence: Verified)*

**🟡 Medium — Confirm Intent**

Three additional resources in the production Atlas environment carry world-open ingress rules. No immediate decision is flagged, but leadership should confirm these are deliberately internet-facing:

| Resource | Exposure | Ports |
|---|---|---|
| cloudox-demo-atlas-prod-alb (Load Balancer) | Internet-facing scheme | — |
| cloudox-demo-atlas-prod-sg-edge (Security Group) | World-open ingress | HTTPS (443) |
| cloudox-demo-atlas-prod-sg-alb (Security Group) | World-open ingress | HTTP (80) |

An internet-facing load balancer with open HTTP/HTTPS security groups is a common and often intentional pattern for public-facing services. However, the HTTP (port 80) exposure on **cloudox-demo-atlas-prod-sg-alb** should be reviewed — if traffic is expected to redirect to HTTPS, port 80 ingress may be unnecessary. *(Confidence: Verified)*

**Notable gaps to be aware of:**
- No AWS Security Hub enablement was discovered across the environment, meaning centralised security findings and compliance checks may not be active.
- 761 of 807 resources carry no Environment/Stage/Tier tag, limiting the ability to assess blast radius or classify resources by criticality with confidence.


---

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


---

## Strategic Risks

Three security gaps stand out across the organisation's AWS accounts — two of them affect the majority of the estate and warrant prompt remediation planning.

### Business & Security Risks

**Threat detection and access analysis are largely absent.** GuardDuty (threat detection) and IAM Access Analyzer (external-access visibility) are each enabled in only 1 of 6 scanned accounts. The five accounts without coverage include the Management Account, Workload Prod Account, Workload Dev Account, Log Archive Account, and Sandbox Ma Account. Running production workloads without GuardDuty means active threats — credential compromise, unusual API calls, network reconnaissance — can go undetected. The absence of IAM Access Analyzer means unintended external access to resources may not be surfaced at all.

| Risk | Accounts Uncovered | Severity |
|---|---|---|
| GuardDuty not enabled | 5 of 6 | Medium |
| IAM Access Analyzer not enabled | 5 of 6 | Medium |

The recommended path for both is org-wide enablement via a delegated administrator — a single action that closes the gap across all accounts simultaneously.

**A broadly-privileged IAM role exists in the Sandbox Ma Account.** The role `cloudox-demo-sandbox-unused-admin` carries a name strongly suggesting administrative access. Its actual policy breadth has not been collected, so the true blast radius is unconfirmed (confidence: Assumed). Until reviewed, it represents a potential over-privilege risk — particularly if the role can be assumed by unintended principals or is no longer actively needed. The recommended action is to review attached policies and apply least privilege.

**Additional context.** Two security groups are open to the internet, and Security Hub enablement has not been discovered across the estate. With 761 of 807 resources lacking environment or stage tags, classification of what is production versus non-production relies on inference rather than explicit tagging — limiting the accuracy of any automated risk prioritisation.

> **Confidence: Likely.** Findings are derived from graph evidence. The IAM role risk is Assumed and requires manual validation before acting on it.


---

## Cost & Efficiency Signals

> **Confidence: Likely** — Spend data is unavailable for this run. The signals below are derived from discovered architecture only; no dollar figures can be confirmed until Cost Explorer collection is enabled.

The single most important action here is a data gap, not a cost finding: billing data collection is currently disabled, meaning leadership has no automated spend visibility through CloudoX. Until that is resolved, cost governance relies entirely on manual Cost Explorer access.

### Where the Money Goes

No spend figures are available for any of the six accounts in scope — Management Account, Sandbox Ma Account, Workload Dev Account, Workload Prod Account, Log Archive Account, or Audit Account. Seven architectural cost drivers were identified from the discovered infrastructure, indicating active resource patterns that carry charges, but exact dollar attribution cannot be provided without billing data.

No evidence found for a breakdown of spend by account, service, or region.

### Efficiency Opportunities

No optimization candidates were surfaced in this run. This is a direct consequence of two collection gaps:

- **Billing data is off.** Cost Explorer collection is disabled, so spend-based anomalies and waste signals cannot be detected.
- **Utilization metrics are not collected.** CloudoX does not collect CloudWatch utilization data in this version, so idle resources, underutilized instances, and right-sizing recommendations based on actual usage are not available.

Additionally, several resource types — including RDS read replicas, RDS provisioned IOPS, DynamoDB capacity modes, Direct Connect, and S3 storage classes — are outside the current collector scope and will not appear as cost drivers even if present.

**Decision needed:** Enable Cost Explorer collection to unlock spend visibility and efficiency analysis across all accounts. Until then, the seven detected architectural cost drivers represent the only available signal, and no savings estimates can be validated.


---

## Recommended Next Actions

### Near-Term Actions

One confirmed architectural risk warrants attention before it becomes an incident: the production database **cloudox-demo-atlas-prod-pg** is running in a single Availability Zone. A zone-level failure would cause downtime and potential data loss with no automatic failover. Enabling Multi-AZ is the recommended remediation (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`).

| Item | Risk | Recommended Action | Decision Required? |
|---|---|---|---|
| **cloudox-demo-atlas-prod-pg** — Single-AZ production database | Downtime / data loss on zone failure | Evaluate enabling Multi-AZ | No — engineering team can proceed |

Two additional areas surfaced by discovery should be tracked by leadership, though they require further investigation before action:

- **Internet-exposed security groups** — 2 security groups are open to the internet (`sg-0459201826f8de5b3`, `sg-06f2b4190bf01d261`). Engineering should confirm whether this exposure is intentional and document the justification.
- **No Security Hub enablement detected** — There is no evidence that AWS Security Hub is active. This is a gap in centralised security posture visibility and worth a deliberate decision on whether to enable it.

**Cost visibility is currently unavailable.** Cost Explorer collection is disabled in this environment, so no spend figures or savings estimates can be provided. Enabling cost collection is recommended before the next review cycle to support data-driven prioritisation.

> **Confidence: Likely** — Findings are derived from discovered architecture. Spend data, CloudWatch utilisation metrics, and several RDS/DynamoDB cost drivers are not available in this run; the full risk and cost picture may be broader than shown here.


---

## Important Unknowns

Four evidence gaps limit the completeness of this analysis. None require an immediate decision, but two directly constrain cost visibility — which leadership should resolve before drawing conclusions about spend or optimization opportunities.

### Gaps Leadership Should Know

| Domain | Gap | Impact on This View |
|--------|-----|---------------------|
| Cost | Cost Explorer collection is disabled | No actual spend figures available; all cost analysis is architecture-derived only |
| Cost | CloudWatch utilization metrics not collected | No idle, underutilized, or right-sizing recommendations possible |
| Cost | RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes not captured | Cost drivers for these services are undetected |
| Coverage | Cloud Control meta-collector was disabled | Long-tail resource types may be missing from the inventory |

**Cost visibility is the most material gap.** Because Cost Explorer collection is disabled, this view cannot report actual spend, trends, or savings estimates — only architectural patterns that are known to carry charges. Enabling cost collection (`CLOUDOX_COST__ENABLED=true`) is the single action that would most improve the fidelity of future runs.

The absence of CloudWatch utilization data means right-sizing and idle-resource analysis cannot be performed in this version of CloudoX. This is a platform-level limitation, not a configuration choice, and should be factored into any cost-optimisation workstream.

The disabled Cloud Control meta-collector means the resource inventory covers only typed collectors. Resources of less common types may be absent from all findings in this view — security, cost, and architecture alike. The true scope of the environment may be broader than what is reflected here.

> **Confidence: Likely** — These gaps are directly evidenced by the discovery run configuration and collector state. Their downstream effect on other sections of this view is real but not fully quantifiable without the missing data.


---

## Evidence Appendix

This appendix identifies the cloud accounts and workloads that form the basis of this Executive View. No raw evidence items were available in this discovery run, so the section serves as a reference index only.

### Key Entities

The following accounts and workloads were identified within the **cloudox-demo** workspace:

| Friendly Name | Type | Region | Confidence |
|---|---|---|---|
| Management Account | Account | — | Verified |
| Sandbox Ma Account | Account | — | Verified |
| Workload Dev Account | Account | — | Verified |
| Workload Prod Account | Account | — | Verified |
| Log Archive Account | Account | — | Likely |
| Cloudox Demo Atlas Prod API | Workload | eu-central-1 | Likely |

> **Confidence note:** The Log Archive Account and the Cloudox Demo Atlas Prod API workload are rated **Likely** rather than Verified — treat any findings tied to these entities with appropriate caution until confirmed.

### Evidence

No evidence items were available for this section. Findings and conclusions elsewhere in this view are drawn from the entities listed above; no provider-native evidence references can be cited here.

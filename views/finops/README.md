# FinOps View

> **Demo Report** — This discovery report was produced from a real AWS environment by
> [CloudoX](https://cloudox.io). AWS account IDs and all resource identifiers (VPC,
> subnet, security group, IAM role IDs, KMS key IDs, and similar) have been replaced
> with deterministic synthetic equivalents so the report can be shared publicly.
> Architecture patterns, workload structure, findings, and service configurations are
> otherwise unmodified — this is not dummy data.

---

> **Audience:** FinOps / Finance  
> **Overall confidence:** Likely  
> Cost drivers, concentration, optimization opportunities, and cost unknowns — cost understood in architectural context.

**This view answers**

- What is driving cost?
- Where is the waste?
- What optimization is worth validating?

**Sections**

- [Cost Overview](#cost-overview)
- [Cost Drivers](#cost-drivers)
- [Optimization Opportunities](#optimization-opportunities)
- [Waste / Utilization Signals](#waste-utilization-signals)
- [Cost Risks](#cost-risks)
- [Missing Cost Evidence](#missing-cost-evidence)
- [Recommended Actions](#recommended-actions)
- [Cost Reference Appendix](#cost-reference-appendix)


---

## Cost Overview

Spend figures are not available for this analysis run — Cost Explorer collection was disabled at discovery time. What follows is architecture-derived: the accounts and resource patterns that are structurally positioned to carry charges, without dollar amounts attached. Treat this as a map of *where to look* in Cost Explorer or CUR, not a billing report.

> **Confidence: Likely** — All cost driver observations are inferred from discovered architecture. No spend data was collected. Validate against your billing data before acting.

### Total Spend Context

No spend figures can be reported. Cost Explorer collection was not enabled during this discovery run (`CLOUDOX_COST__ENABLED=false`), so no UnblendedCost data is available for any billing period.

Seven accounts are in scope across this environment:

| Account | Friendly Name | Confidence |
|---|---|---|
| 110319895932 | Management Account | Verified |
| 161388682021 | Sandbox Ma Account | Verified |
| 105769365151 | Workload Dev Account | Verified |
| 122122642149 | Workload Prod Account | Verified |
| 122980216815 | Log Archive Account | Likely |
| 110019496666 | Audit Account | Likely |
| 150982215529 | Platform Account | Likely |

The presence of dedicated **Workload Dev** and **Workload Prod** accounts alongside a **Sandbox** account suggests at least three distinct spend pools that should be tracked separately in Cost Explorer. The **Log Archive** and **Audit** accounts are typical AWS Control Tower foundational accounts and generally carry lower but non-trivial storage and API costs.

### Where Cost Concentrates

Without spend data, concentration can only be described structurally. Based on the discovered account topology, cost is most likely to concentrate in:

- **Workload Prod Account (122122642149)** — Production workload accounts are the primary driver of compute, data transfer, and managed service charges in most AWS environments. This is the highest-priority account to examine in Cost Explorer.
- **Workload Dev Account (105769365151)** — Development environments frequently carry disproportionate cost relative to business value, particularly from always-on compute and over-provisioned resources. A candidate for rightsizing and scheduling controls, but requires validation against actual usage data.
- **Log Archive Account (122980216815)** — Centralised log archiving accumulates S3 storage costs continuously. S3 storage class selection and lifecycle policies are not captured by the current collectors, so the cost profile here is not known.
- **Platform Account (150982215529)** — Shared platform services (networking, DNS, shared tooling) can carry steady-state costs that are easy to overlook. Exact services in this account are not detailed in this package.

The **Management Account (110319895932)** and **Audit Account (110019496666)** typically carry lower direct workload costs but may include AWS Organizations, CloudTrail, and Config charges.

**What is not visible in this analysis:** CloudWatch utilization metrics are not collected in this version, so idle or underutilized resource identification is not possible. RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage class details are also outside current collector coverage — these can be material cost drivers that will not appear here.

**Recommended next step (requires validation):** Enable Cost Explorer collection in CloudoX and re-run discovery to populate spend figures. In the interim, filter Cost Explorer by the seven account IDs above, grouped by account and service, to establish the actual concentration picture.


---

## Cost Drivers

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All cost driver analysis is derived from discovered architecture only. Dollar attributions cannot be made here; use AWS Cost Explorer or CUR for exact figures.

The most important thing to know before reading this section: **no billing data was collected**. What follows describes the architectural patterns that are known to carry charges across 807 resources, 7 accounts, and multiple workloads — giving you a map of where to look in Cost Explorer, not a spend report.

### Architectural Cost Drivers

Seven architectural cost drivers were identified from the discovered environment. These are structural patterns — not line-item costs — that typically generate ongoing charges and warrant direct investigation in your billing data.

**API Gateway exposure (internet-facing endpoints)**

Two public API Gateway endpoints are active in `eu-central-1`:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` — associated with the **Cloudox Demo Atlas Dev API** workload (Workload Dev Account `105769365151`)
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` — associated with the **Cloudox Demo Atlas Prod API** workload (Workload Prod Account `122122642149`)

API Gateway charges accrue per API call and per GB of data transferred. Both endpoints are internet-facing, meaning external traffic volumes directly drive cost. Prod and Dev endpoints running in parallel means both are incurring baseline charges regardless of traffic levels.

**DynamoDB tables across three accounts**

Three DynamoDB tables are present across different account tiers:

| Table | Account | Workload |
|---|---|---|
| `cloudox-demo-atlas-dev-items` | Workload Dev (`105769365151`) | Cloudox Demo Atlas Dev |
| `cloudox-demo-atlas-prod-items` | Workload Prod (`122122642149`) | Cloudox Demo Atlas Prod API |
| `cloudox-demo-sandbox-scratch` | Sandbox Ma (`161388682021`) | Cloudox Demo Sandbox Scratch |

DynamoDB cost is driven by capacity mode (on-demand vs. provisioned), read/write request volumes, and storage. **The capacity mode for these tables is not captured by the current collectors** — this is a material unknown for cost estimation. The sandbox table (`cloudox-demo-sandbox-scratch`) is a candidate for validation: scratch/sandbox resources can accumulate cost without active use.

**Internet Gateways across multiple accounts and regions**

Four Internet Gateways are present across two regions and three accounts:

| IGW | Account | Region |
|---|---|---|
| `igw-00ed21b9a0e6596a8` | Audit Account (`110019496666`) | us-east-1 |
| `igw-0d14f1dd4e54d5906` | Audit Account (`110019496666`) | eu-central-1 |
| `igw-0567575921f471548` | Workload Dev (`105769365151`) | us-east-1 |
| `igw-0cff0d66b4fd90803` | Log Archive Account (`122980216815`) | us-east-1 |

Internet Gateways themselves carry no hourly charge, but they enable data transfer paths that do. The presence of IGWs in the Audit Account and Log Archive Account — and a Dev account IGW in `us-east-1` while the workload appears to operate in `eu-central-1` — are worth validating. Cross-region data transfer and unexpected egress from governance/logging accounts can be silent cost contributors.

**Multi-account, multi-region footprint**

The environment spans 7 accounts (Management, Workload Dev, Workload Prod, Sandbox, Audit, Log Archive, Platform) and at least two regions (`eu-central-1` and `us-east-1`). This structure is architecturally sound for governance, but it means:
- Per-account fixed costs (e.g., AWS Config, CloudTrail, GuardDuty) multiply across accounts
- Cross-region data transfer charges apply wherever resources communicate across regions
- Cost visibility requires consolidated billing and account-level tagging discipline

**Tagging gap limits cost attribution**

761 of 807 resources (94%) have no Environment / Stage / Tier tag and rely on inference for classification. This is not a cost driver itself, but it severely limits the ability to attribute spend to workloads, environments, or teams in Cost Explorer. Any showback or chargeback model will have significant blind spots until tagging coverage improves.

### Service & Resource Drivers

No spend data is available to rank services by cost. The following service-level observations are derived from architecture discovery alone.

**Services with known per-request or per-unit cost models present in this environment:**
- **Amazon API Gateway** — two internet-facing endpoints (dev + prod); charges per million API calls and per GB data out
- **Amazon DynamoDB** — three tables across dev, prod, and sandbox; charges depend on capacity mode (unknown) and request volume
- **Data transfer** — four Internet Gateways across two regions create potential egress paths; cross-region transfer between `us-east-1` and `eu-central-1` resources would carry inter-region charges

**Optimization candidates requiring validation:**

| Candidate | Rationale | Action Required |
|---|---|---|
| `cloudox-demo-sandbox-scratch` DynamoDB table (Sandbox `161388682021`) | Sandbox/scratch resource; may be idle or low-use | Validate activity in CloudWatch; confirm if still needed |
| IGW `igw-0567575921f471548` in `us-east-1` (Workload Dev `105769365151`) | Dev workload appears to be `eu-central-1`-primary; `us-east-1` IGW may be unused | Confirm whether any resources route through this IGW; check for associated NAT or data transfer charges |
| IGW `igw-0d14f1dd4e54d5906` and `igw-00ed21b9a0e6596a8` in Audit Account (`110019496666`) | Governance accounts with internet gateways are unusual; egress from audit/log infrastructure can be unexpected | Validate whether these IGWs carry active traffic; review data transfer costs for this account |
| Dev API endpoint running in parallel with Prod | Parallel API Gateway stages incur independent call and data transfer charges | Confirm Dev endpoint traffic levels; assess whether Dev can be throttled or decommissioned between active development cycles |

> **Note:** CloudWatch utilization metrics are not collected in this version. Idle or underutilized resource identification (e.g., right-sizing, unused capacity) requires separate analysis using CloudWatch or AWS Compute Optimizer.


---

## Optimization Opportunities

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All analysis is derived from discovered architecture only; no dollar savings figures can be stated.

The one identified candidate in this section is an **architectural risk item, not a cost-saving opportunity**. There are currently **no optimization candidates with a direct cost-reduction angle** surfaced from the discovered architecture. The absence of spend data and utilization metrics (see Unknowns below) materially limits what can be validated at this time.

### Optimization Candidates

No cost-reduction optimization candidates were identified from the discovered architecture in this run. The summary confirms **0 optimization candidates** across 807 resources in 7 accounts.

One architectural finding was surfaced that carries an indirect cost relevance — unplanned downtime from a single-AZ database can generate incident costs and recovery effort:

| Resource | Finding | Recommended Action | Requires Validation |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | Single-AZ DBInstance — a zone failure could cause downtime or data loss | Evaluate enabling Multi-AZ for resilience | Yes |

*Source: `modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`*

Enabling Multi-AZ for `cloudox-demo-atlas-prod-pg` would **increase** the RDS line item (standby instance charges apply), but reduces the risk of unplanned recovery costs. This is a resilience trade-off, not a savings action — finance and engineering should evaluate it together against the workload's recovery requirements.

### Estimated Impact

No estimated savings impact can be stated. Spend data is unavailable, and CloudoX does not collect CloudWatch utilization metrics in this version, so idle-resource or right-sizing recommendations based on actual usage are not available.

The following gaps prevent a complete optimization picture and should be resolved before drawing conclusions:

- **Cost Explorer is disabled** (`CLOUDOX_COST__ENABLED=false`). Enabling it would unlock spend-attributed driver analysis and dollar-level opportunity sizing.
- **No utilization metrics** are collected. Idle EC2, underutilized RDS, and right-sizing candidates cannot be detected without CloudWatch data.
- **Untagged resources**: 761 of 807 resources carry no Environment / Stage / Tier tag, making workload-level cost attribution unreliable. Tag coverage is a prerequisite for meaningful FinOps segmentation.
- **Collector gaps**: RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured — cost drivers for these services are not detected in this run.


---

## Waste / Utilization Signals

> **Confidence: Likely** — Spend data is unavailable for this run. All observations below are derived from discovered architecture only, not from billing records or utilization metrics. Treat this section as a starting point for investigation, not a confirmed waste inventory.

No optimization candidates were identified from the current discovery pass. The 7 architectural cost drivers noted elsewhere in this view represent patterns that *can* carry charges, but without spend data or utilization metrics, CloudoX cannot confirm whether any specific resource is idle, oversized, or wasteful.

### Idle & Unused Resources

No evidence found for confirmed idle or unused resources in this run. Identifying idle resources — such as unattached EBS volumes, stopped EC2 instances still incurring charges, unused Elastic IPs, or orphaned load balancers — requires either Cost Explorer data or CloudWatch utilization metrics, neither of which is available here.

**What to validate manually:**
- Unattached or snapshot-only EBS volumes
- EC2 instances in a stopped state (storage charges continue)
- Elastic IPs not associated with a running instance
- Load balancers with no registered targets

### Utilization Signals

No evidence found for utilization-based signals in this run. CloudoX does not collect CloudWatch metrics in this version, so CPU, memory, network, and storage utilization data are not available. Right-sizing recommendations and underutilization flags cannot be generated without this input.

**Known collection gaps that limit this section:**
- Cost Explorer is disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`); no spend figures are available to correlate with resource inventory.
- CloudWatch utilization metrics are not collected in this version; idle and underutilization signals are not detectable.
- RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are outside current collector coverage — cost drivers for these services will not appear here.

Enabling Cost Explorer collection and, where available, integrating AWS Compute Optimizer or Trusted Advisor findings would materially improve the signal quality in this section on the next run.


---

## Cost Risks

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All findings below are derived from the discovered architecture, not from billing data. Dollar attributions cannot be made here; validate against AWS Cost Explorer or CUR directly.

The most material cost risk in this environment is not overspend on a single service — it is the **absence of security controls whose gaps could generate unplanned remediation and incident costs**. Three security findings with direct cost-risk implications are present across the estate.

### Cost & Commitment Risks

#### Security Control Gaps as Cost Risk

Two detective controls — GuardDuty threat detection and IAM Access Analyzer — are enabled in only **1 of 6 scanned accounts**, leaving five accounts (Log Archive `122980216815`, Workload Dev `105769365151`, Workload Prod `122122642149`, Management `110319895932`, and Sandbox Ma `161388682021`) without consistent coverage (`risk:security:aws-guardduty-detector`, `risk:security:aws-accessanalyzer-analyzer`).

For a FinOps audience, the relevance is indirect but real:

| Control | Accounts Missing | Cost-Risk Angle |
|---|---|---|
| GuardDuty (`risk:security:aws-guardduty-detector`) | 5 of 6 | Undetected compromise can lead to crypto-mining, data exfiltration egress charges, or large-scale resource spin-up — all of which appear on the bill before they are caught. |
| IAM Access Analyzer (`risk:security:aws-accessanalyzer-analyzer`) | 5 of 6 | Undetected over-exposed resources increase the blast radius of a breach, compounding incident and remediation costs. |

Enabling both controls org-wide via a delegated administrator is the recommended action for each. GuardDuty and IAM Access Analyzer do carry their own service charges, but these are typically small relative to the cost of an undetected incident.

#### Broadly-Privileged IAM Role (Requires Validation)

An IAM role named `cloudox-demo-sandbox-unused-admin` (ID: `AROAAAAADPCL3BVEXUDTH`) exists in the Sandbox Ma Account (`161388682021`) and is flagged as potentially broadly privileged based on naming (`risk:security:cloudox-demo-sandbox-unused-admin`). **Confidence on this finding is Assumed** — privilege breadth was not directly collected and must be validated by reviewing attached policies.

The cost-risk angle: an over-privileged role in a sandbox account, if misused or compromised, could be used to provision resources at scale with no guardrails. This warrants validation before it is dismissed.

> **Requires validation:** Confirm the actual policies attached to `cloudox-demo-sandbox-unused-admin` and apply least-privilege scoping if the role is broader than needed.

#### What Is Not Visible Here

The following cost-risk areas could not be assessed in this run and represent material unknowns for FinOps:

- **No spend data** — Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false`). Concentration, trend, and anomaly analysis are not possible without it.
- **No utilization metrics** — CloudWatch metrics are not collected in this version, so idle resources, underutilized compute, and right-sizing opportunities cannot be identified.
- **Collector gaps** — RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured, so cost drivers in those areas are not detected.

Enabling Cost Explorer collection in a future run would unlock spend-grounded analysis and shift confidence from Likely to Verified for cost findings.


---

## Missing Cost Evidence

The cost picture for this environment is incomplete. Spend figures are unavailable because Cost Explorer collection was disabled during the discovery run (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`). Everything in this FinOps view is therefore derived from discovered architecture alone — no dollar amounts, no trend lines, and no validated optimization savings can be stated. Treat all cost driver observations as directional until actual billing data is collected.

> **Confidence: Likely** — Architecture-derived drivers are grounded in real discovered resources, but without spend data, no cost attribution or prioritization by dollar impact is possible.

### Cost Data Gaps

Three distinct gaps limit what can be answered right now:

| Gap | What it blocks |
|---|---|
| Cost Explorer collection disabled | All spend figures, trend analysis, and service-level cost breakdowns (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`) |
| CloudWatch utilization metrics not collected | Idle resource detection, underutilization flags, and right-sizing recommendations based on actual usage (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`) |
| Specific service attributes not captured | RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes — cost drivers for these services are not detected (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`) |

Two additional coverage gaps mean the resource inventory itself may be incomplete, which compounds the cost blind spots: the Cloud Control meta-collector was disabled (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`) and Resource Explorer was unavailable for cross-checking (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`). Resources not discovered cannot be costed.

### What to Enable

To move from architecture-derived observations to a fully evidenced FinOps view, the following collection capabilities should be enabled on the next discovery run:

1. **Cost Explorer collection** — Re-run with `CLOUDOX_COST__ENABLED=true` (or without `--no-collect-costs`). This is the single highest-priority change; it unlocks spend figures, service breakdowns, and dollar-weighted prioritization of every other finding.
2. **CloudWatch utilization metrics** — Once available in a future CloudoX version, this will enable idle and underutilized resource identification and right-sizing candidates grounded in actual usage rather than configuration alone.
3. **Cloud Control and Resource Explorer meta-collectors** — Enabling these will broaden resource coverage and allow cross-checking of inventory completeness, reducing the risk that billable resources are missing from the analysis.

No optimization candidates can be validated or dollar-quantified until at least Cost Explorer collection is active. The architectural cost drivers identified elsewhere in this view should be treated as a prioritization queue to revisit once billing data is available.


---

## Recommended Actions

### Recommended Cost Actions

**Spend data is unavailable for this run.** Cost Explorer collection was disabled at discovery time (`CLOUDOX_COST__ENABLED=false`), so no dollar figures, trends, or per-service breakdowns can be presented. The actions below are derived entirely from the discovered architecture — treat them as *validation candidates*, not confirmed savings.

> **Confidence: Likely** — Architectural patterns are observed; their cost impact requires confirmation in AWS Cost Explorer or CUR once collection is enabled.

---

#### Enable Cost Data Collection — Highest Priority

Before any other action, enable Cost Explorer collection in CloudoX. Without it, none of the 7 detected architectural cost drivers can be dollar-quantified, and no optimization can be sized or prioritised. This is a prerequisite for all downstream FinOps work.

---

#### Validate Internet Gateway Sprawl

Four internet gateways are present across multiple accounts and regions:

| Resource | Account | Region |
|---|---|---|
| `igw-00ed21b9a0e6596a8` | 110019496666 | us-east-1 |
| `igw-0d14f1dd4e54d5906` | 110019496666 | eu-central-1 |
| `igw-0567575921f471548` | 105769365151 | us-east-1 |
| `igw-0cff0d66b4fd90803` | 122980216815 | us-east-1 |

Internet gateways themselves carry no hourly charge, but they are a proxy for data-transfer cost: traffic routed through them incurs AWS data-transfer egress fees. **Validate** whether all four gateways are actively routing production traffic, or whether any serve idle or decommissioned VPCs. Unused gateways attached to live VPCs still expose egress paths that may be generating unintended transfer charges.

---

#### Review DynamoDB Tables Across Environments

Three DynamoDB tables are present across three separate accounts:

| Table ARN | Account | Environment (inferred) |
|---|---|---|
| `…:table/cloudox-demo-atlas-dev-items` | 105769365151 | Dev |
| `…:table/cloudox-demo-atlas-prod-items` | 122122642149 | Prod |
| `…:table/cloudox-demo-sandbox-scratch` | 161388682021 | Sandbox |

DynamoDB cost is driven by capacity mode (on-demand vs. provisioned), read/write throughput, and storage. **Note:** DynamoDB capacity mode is not captured by the current collectors, so it is unknown whether these tables are on-demand or provisioned. **Validate** the following:
- Is the sandbox `scratch` table actively used, or can it be deleted or paused?
- Are dev and prod tables sized appropriately for their actual traffic?
- Confirm capacity mode in the AWS console; on-demand tables with low, spiky traffic may be cheaper on provisioned, and vice versa.

---

#### Confirm API Gateway Endpoints Are Intentionally Public

Two public API Gateway endpoints are exposed to the internet:

- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`

API Gateway charges per API call and per GB of data transfer. **Validate** that both endpoints are actively used and that request volumes are within expected ranges. Unexpected public exposure can lead to unbudgeted call volume (e.g., from scanners or misconfigured clients).

---

#### Address Tagging Gaps Before the Next Billing Review

761 of 807 resources (94%) carry no `Environment`, `Stage`, or `Tier` tag and rely on inference for classification. Without tags, cost allocation reports in Cost Explorer cannot reliably attribute spend to teams, environments, or workloads. **Recommended action:** define a mandatory tagging policy for at minimum `Environment` and `Owner`, and remediate existing untagged resources. This is a prerequisite for meaningful showback or chargeback.

---

#### What Cannot Be Assessed Yet

The following optimization categories require either cost data or utilization metrics that are not currently available:

- **Right-sizing / idle resources** — CloudWatch utilization metrics are not collected in this version; CPU, memory, and network idle analysis is not possible.
- **RDS cost drivers** — Read replicas, provisioned IOPS, and storage class are not captured by current collectors.
- **S3 storage class optimisation** — S3 storage classes are not captured; Intelligent-Tiering or lifecycle policy recommendations cannot be made.
- **Direct Connect** — Not captured; any associated port-hour or data charges are invisible to this analysis.

Once Cost Explorer collection is enabled and a full billing period is collected, re-run this view to surface dollar-quantified drivers and ranked optimization candidates.


---

## Cost Reference Appendix

This appendix catalogues the accounts and workloads that form the cost boundary for this FinOps view. Use it to cross-reference entities mentioned elsewhere in the view against their account IDs and confidence levels. No provider-native cost evidence (billing exports, Cost Explorer data, or equivalent) was available to CloudoX at the time this view was generated — see the Evidence sub-section below.

### Cost Entity Reference

The following accounts and workloads were identified within the `cloudox-demo` workspace. Confidence reflects how firmly each entity's identity and boundary have been established.

**Accounts**

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | 110319895932 | Verified |
| Workload Prod Account | 122122642149 | Verified |
| Workload Dev Account | 105769365151 | Verified |
| Sandbox Ma Account | 161388682021 | Verified |
| Log Archive Account | 122980216815 | Likely |
| Platform Account | 150982215529 | Likely |
| Audit Account | 110019496666 | Likely |

**Workloads**

| Friendly Name | Workload ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox | cloudox | Workload Prod Account (122122642149) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | cloudox-demo-atlas-prod-api | Workload Prod Account (122122642149) | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | cloudox-demo-atlas-dev | Workload Dev Account (105769365151) | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | cloudox-demo-atlas-dev-api | Workload Dev Account (105769365151) | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | cloudox-demo-sandbox-scratch | Sandbox Ma Account (161388682021) | eu-central-1 | Assumed |

> **Confidence note:** Entities marked *Likely* have been inferred with reasonable confidence but have not been fully verified. The *Assumed* entry (Cloudox Demo Sandbox Scratch) should be treated as provisional — validate its account association and cost attribution before acting on any figures tied to it.

### Evidence

No provider-native cost evidence was available to CloudoX for this section. No billing records, cost allocation tags, Cost Explorer exports, or equivalent data sources were present in the package. As a result:

- Spend figures, cost breakdowns, and savings estimates **cannot be derived** from this appendix alone.
- The entity reference above is suitable for use as a lookup table when reconciling account IDs with workload names, but it does not carry monetary values.

**Recommended action (requires validation):** Confirm that Cost and Usage Report (CUR) or equivalent billing data is being exported and is accessible to CloudoX. Once cost data is ingested, this appendix will be populated with per-account and per-workload spend context to support the cost driver and optimization sections of this view.

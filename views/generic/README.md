# Generic View

> **Demo Report** — This discovery report was produced from a real AWS environment by
> [CloudoX](https://cloudox.io). AWS account IDs and all resource identifiers (VPC,
> subnet, security group, IAM role IDs, KMS key IDs, and similar) have been replaced
> with deterministic synthetic equivalents so the report can be shared publicly.
> Architecture patterns, workload structure, findings, and service configurations are
> otherwise unmodified — this is not dummy data.

---

> **Audience:** Any technical reader  
> **Overall confidence:** Likely  
> Balanced, full-depth understanding of the environment; the default when no audience is selected.

**This view answers**

- What is this environment and what runs in it?
- How is it structured, connected, and secured?
- What is uncertain or unknown?

**Sections**

- [Environment Overview](#environment-overview)
- [Architecture Summary](#architecture-summary)
- [Key Workloads](#key-workloads)
- [Security & Exposure](#security-exposure)
- [Operations & Reliability](#operations-reliability)
- [Cost Signals](#cost-signals)
- [Top Findings & Recommendations](#top-findings-recommendations)
- [Unknowns & Validation Questions](#unknowns-validation-questions)
- [Reference Appendix](#reference-appendix)


---

## Environment Overview

This is a multi-account AWS organisation structured around a clear separation of concerns: dedicated accounts for management, audit, log archiving, platform services, workload delivery (dev and prod), and sandboxing. Across these seven accounts, **807 resources** have been discovered, hosting **12 significant workloads** (out of 15 inferred; 3 classified as helper/governance).

> **Confidence: Likely** — Evidence is derived from graph analysis. Some classification relies on inference rather than explicit tagging, and two meta-collectors were unavailable during discovery, meaning the full resource breadth may be wider than what is represented here.

### Scope & Accounts

Seven accounts make up this environment:

| Friendly Name | Account ID | Confidence | Role |
|---|---|---|---|
| Management Account | 110319895932 | Verified | Org root / management |
| Workload Dev Account | 105769365151 | Verified | Development workloads |
| Workload Prod Account | 122122642149 | Verified | Production workloads |
| Sandbox Ma Account | 161388682021 | Verified | Sandbox / experimentation |
| Audit Account | 110019496666 | Likely | Audit trail |
| Log Archive Account | 122980216815 | Likely | Centralised log storage |
| Platform Account | 150982215529 | Likely | Shared platform services |

Three accounts — Audit, Log Archive, and Platform — are classified at **Likely** confidence, meaning their roles are inferred from evidence rather than confirmed by explicit tagging or naming conventions. The Log Archive account's environment stage is currently **Unknown** (`122980216815-unknown`), indicating no stage tag or strong signal was found to classify it further.

Three workloads were demoted from the significant workload list as helper or governance constructs rather than primary application workloads.

### Regions & Footprint

Active resource evidence spans at least two AWS regions:

- **eu-central-1 (Frankfurt)** — API Gateway endpoints (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`), DynamoDB tables in the Dev account (`arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`), Prod account (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`), and Sandbox account (`arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`), plus internet gateways in the Audit account (`arn:aws:ec2:eu-central-1:110019496666:internet-gateway/igw-0d14f1dd4e54d5906`).
- **us-east-1 (N. Virginia)** — Internet gateways present in the Audit account (`arn:aws:ec2:us-east-1:110019496666:internet-gateway/igw-00ed21b9a0e6596a8`), Workload Dev account (`arn:aws:ec2:us-east-1:105769365151:internet-gateway/igw-0567575921f471548`), and Log Archive account (`arn:aws:ec2:us-east-1:122980216815:internet-gateway/igw-0cff0d66b4fd90803`).

The presence of internet gateways in multiple accounts — including Audit and Log Archive — indicates that VPCs with public routing exist beyond just the workload accounts. The practical significance of those gateways (whether actively used or residual) is not determinable from available evidence.

> **Note:** The discovery summary reports **0 regions** in the structured scope metadata, which conflicts with the regional evidence above. This is likely a metadata gap rather than a true zero-region footprint. The Resource Explorer meta-collector was disabled or unavailable, which means AWS-visible regional breadth could not be independently cross-checked.

### What This Environment Is

This is a **workload-hosting AWS organisation** built around the `cloudox-demo` workspace. The naming conventions (`cloudox-demo-atlas-dev-items`, `cloudox-demo-atlas-prod-items`, `cloudox-demo-sandbox-scratch`) point to an application called **Atlas** with at least two promoted stages — development and production — and an active sandbox for experimentation.

The environment follows a recognisable landing-zone pattern: separate accounts for management, audit, and log archiving sit alongside workload and platform accounts. Internet-facing components are confirmed in eu-central-1 (public API Gateway endpoints) and internet gateways exist across multiple accounts and regions, indicating intentional public connectivity in at least the workload tier.

A material tagging gap exists: **761 of 807 resources carry no Environment, Stage, or Tier tag**, meaning workload classification throughout this view relies on inference. Readers should treat workload boundaries and environment assignments as indicative rather than authoritative until tagging is improved. Additionally, the absence of the Cloud Control meta-collector means long-tail or newer AWS resource types may be under-represented in the 807-resource count.

![AWS Organizations account structure](./diagrams/generic-org-account-structure.png)


---

## Architecture Summary

This environment spans **807 resources across 7 AWS accounts**, with the primary workloads concentrated in **eu-central-1** and at least one account extending into **us-east-1**. The estate is organised around a clear dev/prod/sandbox pattern, with internet-facing API surfaces and DynamoDB-backed storage visible across the main workload accounts.

> **Confidence: Likely** — Architecture is derived from graph evidence. Some relationships and workload boundaries rely on inference rather than explicit tagging (see Unknowns below).

### Systems & Workloads

CloudoX identified **12 significant workloads** out of 15 inferred; 3 were demoted as helper or governance workloads and are not detailed here. The significant workloads resolve into **3 systems**.

| Workload | Account | Region | Confidence |
|---|---|---|---|
| Cloudox | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | 122122642149 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | 161388682021 | eu-central-1 | Assumed |

Two workloads — **Cloudox** (`cloudox`) and the **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`) — carry Verified confidence, meaning their identity and boundaries are directly evidenced. The **Cloudox Demo Sandbox Scratch** workload (`cloudox-demo-sandbox-scratch`) is Assumed; treat its scope and membership with caution until confirmed.

Three VPCs are present with Unknown confidence and no friendly names resolved — `vpc-0fd81e106f24b4e85` (account 161388682021, us-east-1), `vpc-02298727f33db8908` (account 122980216815, eu-central-1), and `vpc-0038de9730553c915` (account 110019496666, eu-central-1). Their workload membership and purpose are not established from available evidence.

### How It Fits Together

**Internet exposure** is confirmed across multiple accounts. Two API Gateway endpoints are publicly reachable:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`

Four internet gateways are evidenced across accounts 110019496666 (eu-central-1 and us-east-1), 105769365151 (us-east-1), and 122980216815 (us-east-1), confirming that several VPCs have direct internet paths.

**Persistent storage** is provided by DynamoDB tables in each of the three primary workload accounts:
- `cloudox-demo-atlas-dev-items` (account 105769365151, eu-central-1)
- `cloudox-demo-atlas-prod-items` (account 122122642149, eu-central-1)
- `cloudox-demo-sandbox-scratch` (account 161388682021, eu-central-1)

The naming pattern suggests a consistent data model replicated across dev, prod, and sandbox stages, with each stage isolated in its own AWS account — a common account-per-environment separation strategy. This is an inference from naming and account structure; no explicit cross-account relationship evidence is in scope for this section.

**VPC topology** follows the same account boundary: a dedicated VPC per environment — **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`), **Cloudox Demo Atlas Prod VPC** (`vpc-0aaa6d4f2f981e945`), and **Cloudox Demo Sandbox VPC** (`vpc-0c97d53850c027a14`) — all in eu-central-1. The three unnamed VPCs noted above fall outside this pattern and remain uncharacterised.

> **Unknown:** 761 of 807 resources carry no Environment / Stage / Tier tag. Workload and system boundaries for those resources are inferred rather than declared, which means the workload count and membership should be treated as approximate until tagging is improved.

![Workload architecture](./diagrams/generic-workload-architecture.png)


---

## Key Workloads

The environment hosts **12 significant workloads** (out of 15 inferred) spread across **7 accounts** and **807 resources**, with the majority anchored in **eu-central-1**. The production tier carries a notable resilience concern that warrants attention before any availability incident occurs.

> **Confidence: Likely** — workload boundaries and classifications are graph-inferred; 761 resources carry no Environment/Stage/Tier tag and rely on inference rather than explicit tagging.

### Significant Workloads

Five workloads are identified as key entities across the workspace:

| Friendly Name | Account | Region | Confidence |
|---|---|---|---|
| Cloudox | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | 122122642149 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | 161388682021 | eu-central-1 | Assumed |

**Cloudox** (ref: `cloudox`, account `122122642149`) is the only Verified workload entity, indicating the strongest evidence for its boundary and composition. **Cloudox Demo Sandbox Scratch** (ref: `cloudox-demo-sandbox-scratch`, account `161388682021`) carries only Assumed confidence — its classification should be treated as provisional.

Three additional workloads were demoted as helper/governance workloads and are not listed here.

### Composition & Tiers

Evidence points to at least two tiers of deployment — **production** and **development** — mirrored across separate AWS accounts:

- **Production** workloads run in account `122122642149`, with an API surface exposed via `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` and a DynamoDB table at `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`.
- **Development** workloads run in account `105769365151`, with a corresponding API at `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and a DynamoDB table at `arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`.
- **Sandbox** activity is isolated to account `161388682021`, backed by `arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`.

Internet gateways are present across multiple accounts and regions (eu-central-1 and us-east-1), indicating that at least some workloads have internet-facing exposure:

- `arn:aws:ec2:eu-central-1:110019496666:internet-gateway/igw-0d14f1dd4e54d5906`
- `arn:aws:ec2:us-east-1:110019496666:internet-gateway/igw-00ed21b9a0e6596a8`
- `arn:aws:ec2:us-east-1:105769365151:internet-gateway/igw-0567575921f471548`
- `arn:aws:ec2:us-east-1:122980216815:internet-gateway/igw-0cff0d66b4fd90803`

The presence of us-east-1 gateways is noted; no workloads in the key entities list are attributed to us-east-1, so the relationship between those gateways and the named workloads is not established from this package.

### Key Dependencies

**⚠️ Priority concern: Single-AZ database in the production critical path**

The **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account `122122642149`, eu-central-1) depends on a DBInstance named **`cloudox-demo-atlas-prod-pg`** that is not configured for Multi-AZ. A single availability zone failure would take the production data tier offline for this workload.

| Detail | Value |
|---|---|
| Intelligence item | `dependency_concern:architecture:cloudox-demo-atlas-prod-pg` |
| Affected datastore | `cloudox-demo-atlas-prod-pg` |
| Affected workload | Cloudox Demo Atlas Prod API |
| Impact | Single-AZ datastore in a workload's critical path |
| Confidence | Verified |

**Recommended action:** Confirm whether Multi-AZ is enabled and whether automated backups are in place for `cloudox-demo-atlas-prod-pg`. Additionally, review how the Cloudox Demo Atlas Prod API handles a data-tier failure — for example, whether it degrades gracefully or fails hard. No decision is flagged as immediately required, but this is a priority 3 finding with verified evidence.


---

## Security & Exposure

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain. Treat gaps noted below as areas requiring manual confirmation.

The most urgent finding is a sandbox security group with world-open SSH and database ports — a decision is required. Alongside that, detective controls (GuardDuty and IAM Access Analyzer) are inconsistently deployed across the organisation, leaving four of five non-sandbox accounts with reduced visibility into threats and external access. The production workload carries expected internet exposure through a load balancer and edge security groups, but those rules warrant confirmation.

### Internet Exposure

Three security groups and one load balancer present internet-facing exposure across two accounts.

| Resource | Account | Open Ports / Scheme | Severity | Decision Required? |
|---|---|---|---|---|
| `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) | Sandbox Ma (161388682021) | 22 (SSH), 5432 (PostgreSQL) — `0.0.0.0/0` and `::/0` | **Critical** | **Yes** |
| `cloudox-demo-atlas-prod-alb` | Unknown (account not resolved) | Internet-facing scheme | Medium | No |
| `cloudox-demo-atlas-prod-sg-edge` (`sg-0459201826f8de5b3`) | Workload Prod (122122642149) | 443 — `0.0.0.0/0` / `::/0` | Medium | No |
| `cloudox-demo-atlas-prod-sg-alb` (`sg-06f2b4190bf01d261`) | Workload Prod (122122642149) | 80 — `0.0.0.0/0` / `::/0` | Medium | No |

**Critical — action required:** `cloudox-demo-sandbox-sg-permissive` exposes SSH (22) and PostgreSQL (5432) to the entire internet. There is no evidence this is intentional. The recommended action is to confirm whether the exposure is required and, if not, restrict or remove those ingress rules immediately (`arn:aws:ec2:eu-central-1:161388682021:security-group/sg-054446d655de1ee7f`).

The production ALB (`cloudox-demo-atlas-prod-alb`) and its associated security groups (`cloudox-demo-atlas-prod-sg-edge` on port 443, `cloudox-demo-atlas-prod-sg-alb` on port 80) are internet-facing, which is consistent with serving external traffic. These are flagged as medium severity; confirm the exposure is intentional and that port 80 is either redirecting to HTTPS or required for a specific reason.

In total, 25 internet-facing access paths were observed across the environment.

### Identity & Access

72 IAM roles are present across the organisation. No customer-managed IAM policies were discovered — all role permissions appear to use AWS-managed or inline policies.

Several roles are notable from a security perspective:

- **`cloudox-demo-sandbox-unused-admin`** (Sandbox Ma, 161388682021 — `arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin`): The name indicates administrative permissions that may be unused. This warrants review to confirm whether the role is still needed.
- **`OrganizationAccountAccessRole`** exists in multiple member accounts (161388682021, 105769365151, 122122642149, and others). This is the standard AWS Organizations cross-account access role, granting the management account administrative access to each member. Its presence is expected but should be governed carefully.
- **`cloudox-demo-sandbox-scratch-lambda`** (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-scratch-lambda`) and **`stacksets-exec-899a82a2cb437e970934262f85c7628a`** (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`) are present in the sandbox account. The StackSets execution role is consistent with CloudFormation StackSets deployment from the management account (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`).
- Workload roles in the Dev account (`cloudox-demo-atlas-dev-ecs-exec`, `cloudox-demo-atlas-dev-ecs-task`, `cloudox-demo-atlas-dev-lambda`) and Prod account (`cloudox-demo-atlas-prod-backup`, `cloudox-demo-atlas-prod-dr-replicator`, `cloudox-demo-atlas-prod-ecs-exec`) follow a consistent naming convention, suggesting structured role separation by function.

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the management account, with a dedicated log delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This provides a centralised audit record across the organisation.

### Notable Risks

Two medium-severity, organisation-wide risks stand out because they affect the same five accounts and reduce the environment's ability to detect threats and unintended access.

**Uneven GuardDuty coverage** (`risk:security:aws-guardduty-detector`): GuardDuty threat detection is enabled in only 1 of 6 scanned accounts. The following accounts lack coverage: Log Archive (122980216815), Workload Dev (105769365151), Workload Prod (122122642149), Management (110319895932), and Sandbox Ma (161388682021). The recommended remediation is to enable GuardDuty org-wide via a delegated administrator.

**Uneven IAM Access Analyzer coverage** (`risk:security:aws-accessanalyzer-analyzer`): IAM Access Analyzer is similarly enabled in only 1 of 6 accounts, with the same five accounts uncovered. Without it, externally accessible resources (roles, S3 buckets, KMS keys, etc.) in those accounts will not be automatically flagged. The recommended remediation is to enable Access Analyzer org-wide via a delegated administrator.

Enabling both controls centrally from the management account would close these gaps across all member accounts simultaneously.

> **Gap:** No Security Hub enablement was discovered in this environment. Security Hub would aggregate findings from GuardDuty, Access Analyzer, and other services into a single pane — its absence means there is currently no centralised security findings aggregation.


---

## Operations & Reliability

The environment is broadly well-captured: 807 resources are recorded across 15 VPCs and 63 subnets, with no coverage gaps identified. A single load balancer is present, and 25 internet-facing access paths have been observed — a figure worth keeping in mind when reviewing exposure and blast radius.

### Reliability Signals

The networking fabric spans **15 VPCs** and **63 subnets**, providing the structural segmentation that underpins availability and fault isolation. One load balancer is in use, distributing traffic within this topology.

Two AWS-managed prefix lists are referenced in the environment (`pl-66a5400f` and `pl-6ea54007`, both in `eu-central-1`), indicating that at least some security group or routing rules rely on AWS-maintained IP ranges — a pattern that reduces manual maintenance burden for those specific rules.

**25 internet-facing access paths** are observed. This is the primary reliability-adjacent signal to scrutinise: each path represents a potential ingress point that must remain intentional, correctly routed, and protected against disruption or misconfiguration.

### Operational Concerns

Two meta-collection gaps reduce the completeness of this picture and should be treated as material unknowns:

- **Resource Explorer was disabled or unavailable.** Without it, the full breadth of AWS-visible resources could not be cross-checked against what typed collectors found. Resources in less-common services or regions may be absent from the inventory.
- **Cloud Control was disabled.** Long-tail resource types — those not covered by dedicated typed collectors — are not represented. Operational tooling, custom configurations, or newer AWS resource types may be missing.

The practical consequence is that the **807 resources captured represent a lower bound**, not a confirmed total. Operational runbooks, alerting, and change-management processes that rely on this inventory should account for the possibility of uncatalogued resources until both meta-collectors can be enabled.

> **Confidence: Verified** — figures and signals above are derived from complete graph evidence for the networking and coverage domains. The unknowns relate to deliberate collection scope, not data quality within what was collected.


---

## Cost Signals

> **Confidence: Likely** — Spend data is unavailable for this run. All signals below are derived from the discovered architecture; no dollar figures or exact cost attributions are available.

Seven architectural patterns have been identified across this environment that are known to carry AWS charges. No optimization candidates were surfaced from the current discovery pass. Because Cost Explorer collection is disabled, the analysis cannot be tied to actual billing amounts — treat the drivers below as structural indicators of where spend is likely concentrated, not as measured cost breakdowns.

### Cost Drivers

The following accounts are in scope and each contributes to the overall cost surface of the environment:

| Account | ID | Confidence |
|---|---|---|
| Management Account | 110319895932 | Verified |
| Sandbox Ma Account | 161388682021 | Verified |
| Workload Dev Account | 105769365151 | Verified |
| Workload Prod Account | 122122642149 | Verified |
| Log Archive Account | 122980216815 | Likely |
| Audit Account | 110019496666 | Likely |
| Platform Account | 150982215529 | Likely |

Seven architectural cost drivers have been identified from the discovered resources across these accounts. Exact service-level attribution requires AWS Cost Explorer or a Cost and Usage Report (CUR) query; CloudoX has not collected that data in this run.

Note that several common cost driver categories are outside the current collector scope and therefore not detected: RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes.

### Optimization Signals

No optimization candidates were identified in this run.

Two capability gaps limit optimization coverage:

- **No spend data** — Cost Explorer collection is disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`). Enabling it would allow CloudoX to surface spend-ranked drivers and savings estimates.
- **No utilization metrics** — CloudWatch utilization data is not collected in this version, so idle-resource and right-sizing recommendations based on actual usage are not available.

Enabling both capabilities in a future run would materially improve the signal quality here.


---

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


---

## Unknowns & Validation Questions

Two categories of limitation shape how much confidence to place in this view: **evidence gaps** — things the discovery run could not collect — and **validation questions** — things that were observed but need a human to confirm intent. Readers should treat conclusions in affected domains as directional rather than definitive until these gaps are closed.

> **Confidence: Likely** — findings are derived from graph evidence; the gaps below are the primary reason full confidence cannot be claimed.

### What We Couldn't Confirm

#### Cost visibility is absent

Spend figures are entirely unavailable for this run. Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`), so no dollar amounts, trends, or per-service breakdowns can be reported. All cost-related observations in this view are inferred from the discovered architecture — resource patterns known to carry charges — not from actual billing data. (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`)

In addition, CloudWatch utilization metrics were not collected in this version, so idle, underutilized, or right-sizing recommendations based on actual usage are not available. (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`)

Several cost-relevant resource attributes are also outside current collector scope: RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured, so cost drivers for those services are not detected. (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`)

#### Discovery breadth may be incomplete

Two meta-collectors that would have broadened resource coverage were not active during this run:

| Meta-collector | Status | Effect |
|---|---|---|
| Cloud Control | Disabled | Long-tail resource types are limited to typed collector coverage only |
| Resource Explorer | Disabled or unavailable | AWS-visible resource breadth could not be cross-checked |

This means resources of types not covered by a dedicated typed collector may be absent from the graph entirely. (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`, `evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`)

#### Tagging and classification gaps

761 resources carry no Environment / Stage / Tier tag and rely on inference for classification. Environment assignments for those resources should be treated as approximate. No Security Hub enablement was discovered, so security posture findings from that service are not available in this view.

### Questions to Validate

#### IAM role with apparent administrative privileges — sandbox account

An IAM role named **cloudox-demo-sandbox-unused-admin** (role ID `AROAAAAADPCL3BVEXUDTH`) exists in account `161388682021` and appears to carry administrative privileges. Its name suggests it may be unused, but that cannot be confirmed from discovery data alone.

> **Validation question:** Does IAM role `cloudox-demo-sandbox-unused-admin` require its current privilege level, and who owns it?

This is flagged as **Assumed** confidence — the privilege level is inferred, not directly verified. No immediate action is mandated by this view, but the role warrants confirmation from the team responsible for the sandbox account. (`validation_question:security:cloudox-demo-sandbox-unused-admin`)


---

## Reference Appendix

This appendix provides a stable mapping between the friendly names used throughout this view and the underlying raw identifiers, enabling traceability back to source resources.

### Entity Reference

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Confidence note:** The Log Archive Account (`122980216815`) is rated **Likely** — treat its presence and role as probable but not fully confirmed by direct evidence.

#### Subnets

| Friendly Name | Subnet ID | Account | Region | Confidence |
|---|---|---|---|---|
| Public Subnet (eu-central-1) | `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| Public Subnet (eu-central-1) | `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |

#### Other Resources

| Friendly Name | Resource ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |

### Evidence

No evidence references are available for this section. The entity mappings above are drawn directly from discovered resource metadata; no additional supporting evidence items are citeable for this appendix.

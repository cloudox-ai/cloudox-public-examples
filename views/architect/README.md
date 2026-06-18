# Architect View

> **Demo Report** — This discovery report was produced from a real AWS environment by
> [CloudoX](https://cloudox.io). AWS account IDs and all resource identifiers (VPC,
> subnet, security group, IAM role IDs, KMS key IDs, and similar) have been replaced
> with deterministic synthetic equivalents so the report can be shared publicly.
> Architecture patterns, workload structure, findings, and service configurations are
> otherwise unmodified — this is not dummy data.

---

> **Audience:** Solutions / Cloud Architects  
> **Overall confidence:** Likely  
> Architecture, workloads, dependencies, networking, security architecture, design risks, and modernization opportunities.

**This view answers**

- What design issues exist?
- What dependencies matter?
- What modernization opportunities exist?

**Sections**

- [Architecture Overview](#architecture-overview)
- [Workloads & Systems](#workloads-systems)
- [Dependencies](#dependencies)
- [Networking & Connectivity](#networking-connectivity)
- [Security Architecture](#security-architecture)
- [Design Risks](#design-risks)
- [Modernization Opportunities](#modernization-opportunities)
- [Assumptions & Unknowns](#assumptions-unknowns)
- [Technical Reference Appendix](#technical-reference-appendix)


---

## Architecture Overview

The environment is a multi-account AWS estate centred on **eu-central-1**, running a small number of clearly separated workloads across distinct lifecycle stages. The dominant pattern is serverless API + managed NoSQL storage, with internet-facing endpoints exposed through API Gateway. Four internet gateways across multiple accounts indicate that VPC-based compute also exists alongside the serverless tier, though the relationship between those two layers is not fully resolved from available evidence.

### Architectural Shape

The estate spans **7 accounts** hosting **807 resources**, with **12 significant workloads** inferred from graph evidence (3 further workloads were demoted as helper or governance constructs, and 3 systems were also identified). The primary workload pattern visible in the evidence is:

- **API Gateway → DynamoDB** — two public HTTPS endpoints (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) front workloads that persist to DynamoDB tables in eu-central-1. Both endpoints are reachable from `internet`, making them externally accessible surfaces.
- **VPC / IGW presence** — four internet gateways are present across accounts `110019496666` (both eu-central-1 and us-east-1), `105769365151` (us-east-1), and `122980216815` (us-east-1). This signals VPC-based workloads in at least two regions, though no specific compute or service detail is available in this section's evidence.
- **us-east-1 footprint** — three of the four internet gateways are in us-east-1, suggesting a secondary regional presence that is not reflected in the primary eu-central-1 workload definitions. The relationship between the us-east-1 infrastructure and the named workloads is not established from available evidence.

> **Confidence: Likely.** Shape is derived from graph inference. 761 of 807 resources carry no Environment/Stage/Tier tag, so workload boundaries and environment assignments rely on inference rather than explicit tagging.

### Environments & Accounts

Three lifecycle environments are distinguishable from the key entities and evidence:

| Environment | Account | Primary Region | Key Evidence |
|---|---|---|---|
| **Production** | `122122642149` | eu-central-1 | Workloads: *Cloudox Demo Atlas Prod API* (`cloudox-demo-atlas-prod-api`), *Cloudox* (`cloudox`); DynamoDB table `cloudox-demo-atlas-prod-items` |
| **Development** | `105769365151` | eu-central-1 | Workloads: *Cloudox Demo Atlas Dev* (`cloudox-demo-atlas-dev`), *Cloudox Demo Atlas Dev API* (`cloudox-demo-atlas-dev-api`); DynamoDB table `cloudox-demo-atlas-dev-items`; API Gateway endpoint `xdmn5ldmif`; IGW `igw-0567575921f471548` (us-east-1) |
| **Sandbox** | `161388682021` | eu-central-1 | Workload: *Cloudox Demo Sandbox Scratch* (`cloudox-demo-sandbox-scratch`); DynamoDB table `cloudox-demo-sandbox-scratch` |

Account `110019496666` hosts two internet gateways (eu-central-1 and us-east-1) but does not map to a named workload in this section's evidence — its role is unknown. Account `122980216815` similarly has an IGW in us-east-1 with no workload attribution available.

The *Cloudox* workload (`cloudox`, account `122122642149`) is the only **Verified**-confidence entity; all others are **Likely** or **Assumed**.

### Key Patterns

**Serverless-first, internet-exposed API tier.** The Atlas workloads (dev and prod) follow a consistent pattern: API Gateway endpoint → (implied compute/integration layer) → DynamoDB. Both the dev API endpoint (`xdmn5ldmif`) and at least one prod-side endpoint (`gfwaiva01f`) are directly reachable from the internet. There is no evidence in this section of a WAF, CloudFront distribution, or private API configuration in front of these endpoints — that is a gap worth validating.

**Account-per-environment isolation.** Production, development, and sandbox each occupy a dedicated AWS account, which is a sound isolation boundary. The sandbox account (`161388682021`) has its own DynamoDB table (`cloudox-demo-sandbox-scratch`), consistent with an experimental or scratch-pad use.

**Tagging debt is a structural risk.** 761 of 807 resources (≈94%) have no Environment/Stage/Tier tag. This undermines cost allocation, automated governance, and any tooling that relies on tag-based targeting. Until tagging is remediated, workload boundaries and environment assignments across the estate remain inferred rather than authoritative.

**Unresolved us-east-1 footprint.** Three IGWs in us-east-1 across accounts `110019496666`, `105769365151`, and `122980216815` indicate active VPC infrastructure in a region where no named workload is defined in available evidence. Whether this represents active workloads, legacy infrastructure, or DR capacity is not known from this section's evidence and warrants direct investigation.

![AWS Organizations account structure](./diagrams/architect-org-account-structure.png)


---

## Workloads & Systems

The estate resolves to **3 systems** and **12 significant workloads** (3 further workloads were demoted as helper/governance artefacts), spread across 7 accounts and 807 resources — the majority in **eu-central-1**. Classification confidence is mixed: a large proportion of resources carry no Environment/Stage/Tier tags, so workload boundaries are inferred rather than declared. Architects should treat workload membership as a working model to validate, not a ground truth.

### Systems

One system is explicitly identified from the evidence:

| System | Confidence | Notes |
|---|---|---|
| **Cloudox Demo Atlas Dev** | Assumed | No account or region binding confirmed; inferred from workload grouping |

Two further systems are implied by the prod/dev split of the Atlas workloads and the presence of a sandbox account, but the package does not surface them as named system entities. No evidence found for additional named systems beyond the above.

> **Confidence note (Assumed):** The Cloudox Demo Atlas Dev system boundary is inferred. Its account and region attribution are not confirmed in the available evidence.

### Workloads

Six workloads are identified across three accounts, all anchored in **eu-central-1**:

| Workload | Account | Region | Confidence | Key Evidence |
|---|---|---|---|---|
| **Cloudox** | 122122642149 | eu-central-1 | Verified | — |
| **Cloudox Demo Atlas Prod API** | 122122642149 | eu-central-1 | Likely | API GW endpoint `xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`; DynamoDB table `cloudox-demo-atlas-prod-items` |
| **Cloudox Demo Atlas Dev** | 105769365151 | eu-central-1 | Likely | DynamoDB table `cloudox-demo-atlas-dev-items` |
| **Cloudox Demo Atlas Dev API** | 105769365151 | eu-central-1 | Likely | API GW endpoint `gfwaiva01f.execute-api.eu-central-1.amazonaws.com` |
| **Cloudox Demo Sandbox Scratch** | 161388682021 | eu-central-1 | Assumed | DynamoDB table `cloudox-demo-sandbox-scratch` |

The **Cloudox** workload (account 122122642149) is the only Verified entity — it shares an account with the Atlas Prod API, suggesting a potential co-tenancy or platform dependency worth confirming. The sandbox workload (`cloudox-demo-sandbox-scratch`) is Assumed confidence; its scope and lifecycle are not established from the available evidence.

Internet gateways are present across multiple accounts and regions (eu-central-1 and us-east-1), indicating that at least some workloads have or had internet-facing paths:

| IGW | Account | Region |
|---|---|---|
| `igw-0d14f1dd4e54d5906` | 110019496666 | eu-central-1 |
| `igw-00ed21b9a0e6596a8` | 110019496666 | us-east-1 |
| `igw-0567575921f471548` | 105769365151 | us-east-1 |
| `igw-0cff0d66b4fd90803` | 122980216815 | us-east-1 |

The us-east-1 IGWs appear in accounts not associated with the named workloads above. No evidence found for active workloads in us-east-1 — these may represent dormant VPC infrastructure or accounts outside the current workload inventory.

### Components & Tiers

From the evidence, two component tiers are visible for the Atlas workloads:

- **API tier:** Amazon API Gateway public endpoints serve as the front door for both the dev API (`gfwaiva01f.execute-api.eu-central-1.amazonaws.com`) and the prod API (`xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`). Both are internet-reachable (`internet` ingress path confirmed).
- **Data tier:** Amazon DynamoDB tables back each environment — `cloudox-demo-atlas-dev-items` (account 105769365151), `cloudox-demo-atlas-prod-items` (account 122122642149), and `cloudox-demo-sandbox-scratch` (account 161388682021).

No evidence found for compute tiers (e.g., Lambda, ECS, EC2) within this section's package — the integration layer between API Gateway and DynamoDB is not confirmed here and should be treated as an unknown.

**Design observations for architects:**
- Dev and prod workloads are isolated into separate AWS accounts, which is a sound boundary pattern, but the co-location of **Cloudox** and **Cloudox Demo Atlas Prod API** in the same account (122122642149) warrants a blast-radius review.
- 761 of 807 resources lack Environment/Stage/Tier tags, making automated governance, cost allocation, and workload-boundary enforcement unreliable at scale. Tag remediation is a prerequisite for confident architecture analysis.
- The us-east-1 IGWs in accounts with no identified workloads represent unaccounted network exposure that should be investigated.

![Workload architecture](./diagrams/architect-workload-architecture.png)


---

## Dependencies

The most actionable finding in this section is a verified single-AZ database dependency sitting in the critical path of a production workload — a resilience gap that warrants immediate architectural review. Beyond that, the dependency graph spans multiple workloads across at least three accounts in eu-central-1, with internet-facing access paths observed across several VPCs.

*Overall confidence: Likely — derived from graph evidence; some classification relies on inference where tagging is absent.*

### Workload Dependencies

Five significant workloads are identified across the environment, anchored in eu-central-1:

| Workload | Account | Region | Confidence |
|---|---|---|---|
| Cloudox Demo Atlas Prod API (`cloudox-demo-atlas-prod-api`) | 122122642149 | eu-central-1 | Likely |
| Cloudox (`cloudox`) | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Dev (`cloudox-demo-atlas-dev`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API (`cloudox-demo-atlas-dev-api`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch (`cloudox-demo-sandbox-scratch`) | 161388682021 | eu-central-1 | Assumed |

Two API Gateway endpoints are observed as internet-facing entry points: `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`. These are consistent with the Atlas Dev and Prod API workloads, though the exact mapping between endpoints and workloads is not explicitly confirmed in the available evidence.

Internet gateways are present across multiple accounts and regions (`igw-00ed21b9a0e6596a8` and `igw-0d14f1dd4e54d5906` in account 110019496666 across us-east-1 and eu-central-1; `igw-0567575921f471548` in account 105769365151 us-east-1; `igw-0cff0d66b4fd90803` in account 122980216815 us-east-1), indicating that internet-facing exposure is not confined to a single account or region. The summary records 25 internet-facing access paths across 15 VPCs — a surface area worth mapping against intended public exposure.

### Data Dependencies

Three DynamoDB tables are identified as data-tier dependencies:

| Table | Account | Workload Association |
|---|---|---|
| `cloudox-demo-atlas-dev-items` | 105769365151 | Atlas Dev workload |
| `cloudox-demo-atlas-prod-items` | 122122642149 | Atlas Prod workload |
| `cloudox-demo-sandbox-scratch` | 161388682021 | Sandbox Scratch workload |

The more critical data dependency is the RDS DBInstance **cloudox-demo-atlas-prod-pg**, which is not Multi-AZ. The **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account 122122642149, eu-central-1) has a verified dependency on this instance. A single availability zone failure would take the production data tier offline. This is the highest-priority architectural concern in this section (`dependency_concern:architecture:cloudox-demo-atlas-prod-pg`, priority 3, significance 0.57).

> **Recommended action:** Confirm whether Multi-AZ is enabled and automated backups are configured for `cloudox-demo-atlas-prod-pg`. Review how the Cloudox Demo Atlas Prod API handles a data-tier failure — connection retry logic, circuit-breaking, and graceful degradation should all be validated.

Note: the account ID and region for `cloudox-demo-atlas-prod-pg` itself are not confirmed in the available evidence (confidence: Unknown on the resource entity).

### Coupling Risks

The primary coupling risk is the tight, unguarded dependency between the production API workload and a non-resilient relational datastore:

- **Single-AZ RDS in a production critical path** — `cloudox-demo-atlas-prod-pg` is not Multi-AZ. There is no evidence of a standby, read replica, or failover mechanism in the package. A zone failure is a full data-tier outage for the Cloudox Demo Atlas Prod API.
- **Multi-account, multi-region internet gateway spread** — Internet gateways appear in accounts 110019496666, 105769365151, and 122980216815 across both eu-central-1 and us-east-1. Without explicit evidence of centralised egress or ingress controls, this distribution increases the blast radius of any misconfiguration.
- **Tagging gaps limit dependency tracing** — 761 resources have no Environment/Stage/Tier tag and rely on inference for classification. This means some dependency paths may be misattributed or invisible to the graph. Architects should treat any inferred workload boundary (particularly `cloudox-demo-sandbox-scratch`, confidence: Assumed) as unconfirmed until validated against actual resource tags or IaC definitions.
- **Dev/Prod account separation** — Atlas Dev (account 105769365151) and Atlas Prod (account 122122642149) are in separate accounts, which is a positive isolation pattern. However, the presence of a shared `cloudox` workload (Verified, account 122122642149) alongside the Prod API in the same account warrants a review of whether blast-radius isolation between those two workloads is sufficient.


---

## Networking & Connectivity

The environment spans **15 VPCs and 63 subnets** across eu-central-1, with **25 internet-facing access paths** observed — a surface area that warrants careful review for any workloads carrying sensitive data. Three named VPCs anchor the topology: **Cloudox Demo Atlas Prod VPC** (`vpc-0aaa6d4f2f981e945`, account `122122642149`), **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`, account `105769365151`), and **Cloudox Demo Sandbox VPC** (`vpc-0c97d53850c027a14`, account `161388682021`), all in eu-central-1. One load balancer is present in the environment.

### VPCs & Subnets

All three named VPCs are single-region (eu-central-1) and each carries at least one **Public Subnet** in the key-entity set:

| VPC | Account | Public Subnets (observed) |
|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | `122122642149` | `subnet-016c22941a019a137`, `subnet-013a24d318ed6f3d0` |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | `105769365151` | `subnet-0f64d71c952a7898a`, `subnet-065f522206524ab12` |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | `161388682021` | `subnet-029e2cceb3d0beff7` |

The package surfaces only public subnets in the key-entity set. Whether private or isolated subnets exist within these VPCs — and how workloads are distributed across them — is not evidenced here, though the total subnet count of 63 across 15 VPCs implies additional subnet tiers exist beyond what is named.

**Design note:** The presence of public subnets in both Prod and Dev VPCs, combined with 25 internet-facing access paths, suggests workloads or endpoints are directly reachable from the internet. Architects should confirm whether all public-subnet placements are intentional and whether security group / NACL controls are appropriately scoped.

### Routing & Egress

Each of the three named VPCs has a dedicated **Internet Gateway** attached:

| VPC | Internet Gateway |
|---|---|
| Cloudox Demo Atlas Dev VPC | Cloudox Demo Atlas Dev Igw (`igw-0155958a0a5e9c500`) |
| Cloudox Demo Atlas Prod VPC | Cloudox Demo Atlas Prod Igw (`igw-01dd5625ec1ea7a54`) |
| Cloudox Demo Sandbox VPC | Sandbox Internet Gateway (`igw-08017528fa36ccb4d`) |

All three environments — including Prod — have direct internet egress/ingress capability. No NAT Gateway, Transit Gateway, or VPN/Direct Connect entity is evidenced in this package. Whether those constructs exist in the remaining 12 VPCs or serve as the egress path for private subnets is unknown from this context.

Two AWS-managed prefix lists are referenced in the environment: `pl-66a5400f` and `pl-6ea54007` (both `arn:aws:ec2:eu-central-1:aws:prefix-list/…`). These are standard AWS-managed prefix lists for eu-central-1 (typically S3 and DynamoDB service CIDRs used in route tables or security groups), indicating some routing or security-group rules reference AWS service ranges — consistent with the VPC endpoint observed below.

**Design consideration:** With internet gateways on all three named VPCs and 25 internet-facing paths, the blast radius of a misconfigured security group or NACL is elevated. Architects should validate whether a hub-and-spoke or shared-egress model (e.g., centralised NAT or inspection VPC) would reduce exposure, particularly for the Prod environment.

### Private Connectivity

One **DynamoDB Interface Endpoint** (`vpce-02ddbd8d63a6ab447`) is present in the Prod VPC account (`122122642149`, eu-central-1). This routes DynamoDB traffic from that VPC over the AWS private network rather than the public internet, which aligns with the AWS-managed prefix list references noted above.

No evidence is found in this package for:
- VPC peering connections between the Prod, Dev, or Sandbox VPCs
- Transit Gateway attachments
- PrivateLink endpoints beyond the single DynamoDB interface endpoint
- VPN or Direct Connect for on-premises connectivity

**Modernization opportunity:** Only one VPC endpoint is evidenced. If workloads in Dev or Sandbox also access DynamoDB or other AWS services (S3, SSM, Secrets Manager, etc.), adding equivalent endpoints would eliminate unnecessary internet-path exposure and reduce data-transfer costs. The absence of observed inter-VPC private connectivity (peering or Transit Gateway) means Dev-to-Prod or Sandbox-to-Prod traffic, if it exists, likely traverses the public internet — a dependency worth validating and potentially replacing with private routing.

![Workload VPC network topology](./diagrams/architect-network-topology.png)


---

## Security Architecture

> **Confidence: Likely** — Derived from graph evidence; some unknowns remain (see below).

The most pressing architectural concern is an unrestricted security group in the sandbox account that exposes SSH (port 22) and PostgreSQL (port 5432) directly to the internet — this requires an immediate design decision. Beyond that critical item, the production Atlas workload carries an expected internet-facing edge (ALB + public subnets) whose exposure is verified but intentional by design; the architecture question is whether the current segmentation is sufficient.

---

### Trust & Identity

The environment surfaces **72 IAM roles** across accounts and **0 customer-managed IAM policies** — all permissions are expressed through inline or AWS-managed policies, which limits reuse and central auditability of permission boundaries.

Key roles observed across accounts:

| Role | Account | Architectural Significance |
|---|---|---|
| `cloudox-demo-sandbox-unused-admin` | 161388682021 | Admin-scoped role with no evidence of active use — high-privilege, low-justification |
| `OrganizationAccountAccessRole` | 122980216815, 110019496666, 105769365151, 122122642149, 161388682021 | Standard cross-account break-glass role present in all member accounts |
| `cloudox-demo-atlas-dev-ecs-exec` / `ecs-task` | 105769365151 | ECS execution and task identity for the dev Atlas workload |
| `cloudox-demo-atlas-dev-lambda` | 105769365151 | Lambda execution identity in dev |
| `cloudox-demo-atlas-prod-ecs-exec` | 122122642149 | ECS execution identity in prod |
| `cloudox-demo-atlas-prod-backup` | 122122642149 | Backup automation role in prod |
| `cloudox-demo-atlas-prod-dr-replicator` | 122122642149 | DR replication role — implies cross-region or cross-account data movement |
| `cloudox-demo-sandbox-scratch-lambda` | 161388682021 | Scratch Lambda role in sandbox — scope unknown |
| `stacksets-exec-*` | 161388682021 | CloudFormation StackSets execution role, managed by org admin |

Org-level CloudTrail (`cloudox-demo-org-trail-o-aaaapzvebq`) is present in `eu-central-1` under account `110319895932`, with a dedicated log-delivery role (`cloudox-demo-org-trail-logs`). CloudFormation StackSets org-admin delegation is also active (`AWSServiceRoleForCloudFormationStackSetsOrgAdmin`), indicating centrally-managed baseline deployments.

**Design concern:** `cloudox-demo-sandbox-unused-admin` (`AROAAAAADPCL3BVEXUDTH`) carries admin-level permissions and shows no evidence of active use. An unused admin role in a sandbox account is a standing privilege escalation path that should be removed or scoped down.

**Unknown:** No Security Hub enablement was discovered. There is no evidence of centralised findings aggregation or compliance posture scoring across accounts.

---

### Exposure & Boundaries

Four security groups and one load balancer carry verified internet-facing exposure. One is **critical** and requires a decision; the others are **medium** severity and consistent with an intentional public-edge pattern.

#### Critical — Sandbox permissive group

`cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`, account `161388682021`, `eu-central-1`) allows `0.0.0.0/0` and `::/0` ingress on:
- **Port 22 (SSH)** — direct shell access from the internet
- **Port 5432 (PostgreSQL)** — database port directly reachable from the internet

This is a **Priority 2 / Critical** finding (`security_exposure:security:sg-054446d655de1ee7f`). A decision is required: confirm whether this exposure is intentional (e.g., a temporary scratch environment) and restrict or remove it if not. The combination of SSH and a database port open to the world in the same group is architecturally indefensible in any persistent environment.

#### Medium — Production Atlas edge exposure

The following resources form the internet-facing edge of the Atlas production workload (account `122122642149`, `eu-central-1`) and are consistent with a standard public ALB pattern:

| Resource | ID | Open Port(s) | Finding |
|---|---|---|---|
| `cloudox-demo-atlas-prod-alb` | `cloudox-demo-atlas-prod-alb` | — (internet-facing scheme) | `security_exposure:security:cloudox-demo-atlas-prod-alb` |
| `cloudox-demo-atlas-prod-sg-edge` | `sg-0459201826f8de5b3` | 443 | `security_exposure:security:sg-0459201826f8de5b3` |
| `cloudox-demo-atlas-prod-sg-alb` | `sg-06f2b4190bf01d261` | 80 | `security_exposure:security:sg-06f2b4190bf01d261` |

Port 80 open on `cloudox-demo-atlas-prod-sg-alb` is worth confirming: if the ALB is expected to redirect HTTP→HTTPS, the security group rule is technically required but the redirect behaviour should be validated at the listener level.

---

### Segmentation

Four public subnets are confirmed across the prod and dev Atlas environments, all in `eu-central-1`. Their route tables forward to an Internet Gateway, making them internet-reachable by definition.

| Subnet | Account | Environment | Finding |
|---|---|---|---|
| `cloudox-demo-atlas-prod-public-a` (`subnet-013a24d318ed6f3d0`) | 122122642149 | Prod | `security_exposure:security:subnet-013a24d318ed6f3d0` |
| `cloudox-demo-atlas-prod-public-b` (`subnet-016c22941a019a137`) | 122122642149 | Prod | `security_exposure:security:subnet-016c22941a019a137` |
| `cloudox-demo-atlas-dev-public-a` (`subnet-065f522206524ab12`) | 105769365151 | Dev | `security_exposure:security:subnet-065f522206524ab12` |
| `cloudox-demo-atlas-dev-public-b` (`subnet-0f64d71c952a7898a`) | 105769365151 | Dev | `security_exposure:security:subnet-0f64d71c952a7898a` |

The presence of named public subnets in both prod and dev is consistent with a multi-tier VPC design (public subnets for the ALB/edge, private subnets for compute and data). However, the package does not contain evidence of private subnet configuration, NAT gateway placement, or network ACL rules — so the depth of the private-tier segmentation cannot be confirmed from available evidence.

**Modernization opportunity:** With 0 customer-managed IAM policies observed, there is a clear opportunity to introduce permission boundaries and reusable managed policies — particularly to constrain the `OrganizationAccountAccessRole` in each member account and to replace any inline policies on the ECS task/exec roles with auditable, version-controlled managed policies.


---

## Design Risks

The most consequential finding across this environment is a **systematic gap in detective security controls**: both GuardDuty and IAM Access Analyzer are enabled in only 1 of 6 scanned accounts, leaving the majority of the estate — including production, dev, management, and log archive accounts — without consistent threat detection or external-access analysis. This is an architectural control-plane gap, not an isolated misconfiguration, and it affects the blast radius of any future compromise.

### Architectural Risks

Two medium-severity, org-wide risks dominate the design-risk picture. They share the same affected account set and the same remediation pattern, which makes them efficient to address together.

| Risk | Severity | Accounts Uncovered | Recommended Action |
|---|---|---|---|
| Uneven GuardDuty coverage (`risk:security:aws-guardduty-detector`) | Medium | Log Archive (122980216815), Workload Dev (105769365151), Workload Prod (122122642149), Management (110319895932), Sandbox Ma (161388682021) | Enable org-wide via delegated administrator |
| Uneven IAM Access Analyzer coverage (`risk:security:aws-accessanalyzer-analyzer`) | Medium | Same 5 accounts as above | Enable org-wide via delegated administrator |

**GuardDuty gap** (`risk:security:aws-guardduty-detector`): Threat detection is absent from 5 of 6 scanned accounts, including the Workload Prod Account (122122642149) and the Management Account (110319895932). The management account is a particularly sensitive gap — compromise there has org-wide reach. The recommended architectural fix is to designate a delegated administrator account and push GuardDuty enablement org-wide, rather than enabling it account-by-account.

**IAM Access Analyzer gap** (`risk:security:aws-accessanalyzer-analyzer`): External-access analysis is similarly absent from the same 5 accounts. Without Access Analyzer, resource policies that inadvertently grant cross-account or public access (on S3 buckets, KMS keys, API Gateway endpoints, DynamoDB tables, etc.) will go undetected. Given that internet-facing API Gateway endpoints are present in eu-central-1 (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) and internet gateways exist across multiple accounts and regions, the absence of Access Analyzer in the Workload Dev and Workload Prod accounts is a meaningful exposure. The same delegated-administrator pattern applies.

**No Security Hub enablement discovered**: The package notes that Security Hub has not been found in this environment. Combined with the GuardDuty and Access Analyzer gaps, this means there is no centralised aggregation point for security findings — a structural gap for any architect designing a detection and response capability.

**Internet gateway spread**: Internet gateways are present across at least four accounts and two regions (us-east-1 and eu-central-1), including in the Log Archive Account (igw-0cff0d66b4fd90803) and Workload Dev Account (igw-0567575921f471548). The architectural intent behind internet gateway presence in a log archive account is not evident from available evidence and warrants review.

### Technical Debt Signals

**Broadly-privileged IAM role in Sandbox** (`risk:security:cloudox-demo-sandbox-unused-admin`): The IAM role `cloudox-demo-sandbox-unused-admin` (AROAAAAADPCL3BVEXUDTH) in the Sandbox Ma Account (161388682021) carries a name that suggests broad administrative access. *Confidence on this item is Assumed* — privilege breadth was not directly collected and is inferred from naming alone. Validation against the attached policies is required before acting. If the name is accurate, the role represents an over-privilege risk with a large identity blast radius. Recommended action: review attached policies and apply least privilege.

**Tagging debt**: 761 of 807 resources carry no Environment / Stage / Tier tag, meaning workload classification across this estate relies on inference rather than explicit metadata. For architects, this limits the reliability of any environment-boundary analysis, cost allocation, or automated policy enforcement that depends on resource tags. Addressing tagging at the infrastructure-as-code level is the durable fix.

> **Confidence note (Likely):** The GuardDuty and Access Analyzer findings are derived from graph evidence and are assessed as Likely. The IAM role risk is Assumed and must be validated. Security Hub absence is an unknown (not confirmed disabled — simply not discovered).


---

## Modernization Opportunities

The single confirmed modernization signal in this discovery run is a resilience gap in the production database tier. Addressing it is low-friction relative to its availability impact, making it the highest-priority architectural action available from current evidence.

### Modernization Candidates

One candidate was identified with **Verified** confidence:

| Resource | Issue | Impact | Priority |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | Single-AZ RDS DBInstance | Reduced availability / recovery posture | 4 |

**`cloudox-demo-atlas-prod-pg`** (`modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`) is a production RDS instance deployed in a single Availability Zone. A zone-level failure would result in downtime and potential data loss with no automatic failover path. Given its placement in the `atlas-prod` account alongside the DynamoDB table `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`, this instance is likely part of a production workload where availability expectations are higher than a single-AZ posture can satisfy.

No further modernization candidates are present in the current evidence package. The broader inventory — 807 resources across 7 accounts — may contain additional candidates, but CloudWatch utilization metrics are not collected in this version, so idle-resource, right-sizing, or compute-modernization signals (e.g., graviton migration, serverless offload) cannot be derived from current data.

> **Confidence note:** The single-AZ finding is Verified. The absence of other candidates reflects the limits of the current collection scope, not a clean bill of health.

### Recommended Improvements

**Enable Multi-AZ on `cloudox-demo-atlas-prod-pg`.**
Activating Multi-AZ creates a synchronous standby replica in a second AZ and enables automatic failover, directly closing the availability gap. This is an in-place RDS configuration change and does not require re-architecture of the surrounding workload. Evaluate during a low-traffic window, as the promotion step involves a brief I/O pause.

**Dependency consideration:** Before enabling Multi-AZ, confirm that any application layer connecting to this instance (likely within the `atlas-prod` account) uses the RDS endpoint DNS name rather than a hardcoded IP, so failover is transparent.

**Deferred opportunities (data gaps):** The following modernization dimensions cannot be assessed until collection gaps are closed:

- *Right-sizing / idle resources* — CloudWatch utilization metrics are not collected; no usage-based recommendations are possible.
- *DynamoDB capacity mode optimisation* — capacity mode is not captured by current collectors (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items` and peers).
- *RDS IOPS and read-replica patterns* — not captured; over-provisioned IOPS or missing read replicas cannot be detected.
- *S3 storage class tiering* — not captured; Intelligent-Tiering or lifecycle opportunities are unknown.
- *Tagging coverage* — 761 of 807 resources carry no Environment/Stage/Tier tag, which limits workload-scoped analysis and makes it harder to target modernization efforts by environment.

Enabling cost collection (`CLOUDOX_COST__ENABLED`) and CloudWatch metric collection in a future run would unlock spend-attributed and utilisation-driven recommendations across the full estate.


---

## Assumptions & Unknowns

Two collector gaps and one unresolved IAM question materially limit the confidence of this view. Architects should treat any workload classification, cost estimate, or security posture statement as **Likely** rather than Verified until the gaps below are closed.

### Assumptions Made

No explicit assumptions were recorded in the discovery run for this section. However, the following inference-driven conditions shape the entire view and should be treated as working assumptions:

- **Environment classification by inference.** 761 of the 807 discovered resources carry no `Environment`, `Stage`, or `Tier` tag. CloudoX has inferred environment membership (dev / prod / sandbox) from naming conventions and account context rather than authoritative tags. Workload groupings and environment-boundary analysis downstream depend on this inference being correct.
- **Architecture-only cost analysis.** Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`). All cost-related observations in this view are derived from the discovered architecture shape alone — no actual spend figures are available (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`). Treat any cost commentary as directional.
- **Collector coverage is typed, not exhaustive.** The Cloud Control meta-collector was disabled, meaning long-tail or less-common AWS resource types are only visible if a dedicated typed collector exists for them (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`). Resources outside typed collector scope are silently absent from the graph.

### Open Questions

The following gaps and validation questions remain unresolved and are material to architectural decisions:

#### Discovery Coverage

| Gap | Impact on this view |
|---|---|
| Resource Explorer meta-collector disabled or unavailable | AWS-visible resource breadth could not be cross-checked; undiscovered resources may exist (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`) |
| Cloud Control meta-collector disabled | Long-tail resource types (e.g. less-common managed services) may be absent from the dependency graph |

#### Security Posture

- **Security Hub not discovered.** No Security Hub enablement was found across any account in scope (`evidence_gap:security:no-security-hub-enablement-discovered`). It is unknown whether Security Hub is absent, disabled, or simply outside collector reach. This means no centralised findings aggregation or compliance standard status can be confirmed from this data.
- **IAM role privilege validation required.** The role `cloudox-demo-sandbox-unused-admin` (id: `AROAAAAADPCL3BVEXUDTH`, account `161388682021`) carries what appears to be administrative privileges. It is unconfirmed whether this scope is intentional and who owns the role (`validation_question:security:cloudox-demo-sandbox-unused-admin`). This is flagged as **Assumed** — the concern is inferred from the role name and privilege level, not from a policy document audit.

#### Cost Drivers

Several cost-relevant dimensions are outside current collector coverage and cannot be assessed (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`):

- RDS read replicas and provisioned IOPS
- DynamoDB capacity mode (on-demand vs. provisioned)
- Direct Connect
- S3 storage classes

Right-sizing and idle-resource recommendations are also unavailable because CloudWatch utilisation metrics are not collected in this version (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`). Any compute or database sizing decisions should be validated against actual CloudWatch data before acting.

#### Recommended Next Steps to Close Gaps

1. Enable Resource Explorer and re-run discovery to cross-check resource breadth.
2. Enable the Cloud Control meta-collector to surface long-tail resource types.
3. Enable Cost Explorer collection (`CLOUDOX_COST__ENABLED=true`) to replace architecture-inferred cost commentary with actuals.
4. Confirm Security Hub enablement status across all accounts — or enable it if absent.
5. Resolve ownership and intended privilege scope of `cloudox-demo-sandbox-unused-admin` with the sandbox account owner.
6. Apply consistent `Environment` / `Stage` / `Tier` tags to the 761 untagged resources to replace inference-based classification.


---

## Technical Reference Appendix

This appendix provides a stable friendly-name → raw-identifier mapping for all key entities referenced across this view. Use it to correlate prose references with console lookups, IaC state, or API queries.

### Entity Reference

#### Accounts

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | `110319895932` | Verified |
| Sandbox Ma Account | `161388682021` | Verified |
| Workload Dev Account | `105769365151` | Verified |
| Workload Prod Account | `122122642149` | Verified |
| Log Archive Account | `122980216815` | Likely |

> **Note — Log Archive Account (`122980216815`):** Confidence is **Likely**, not Verified. Treat its role and membership in the AWS Organization as unconfirmed until directly validated in the management console or via AWS Organizations API.

#### Security Groups

| Friendly Name | Resource ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox Demo Sandbox Sg Permissive | `sg-054446d655de1ee7f` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod Sg ALB | `sg-06f2b4190bf01d261` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |

#### Subnets

| Subnet ID | Account | Region | Confidence |
|---|---|---|---|
| `subnet-016c22941a019a137` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| `subnet-013a24d318ed6f3d0` | Workload Prod Account (`122122642149`) | eu-central-1 | Verified |
| `subnet-0f64d71c952a7898a` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| `subnet-065f522206524ab12` | Workload Dev Account (`105769365151`) | eu-central-1 | Verified |
| `subnet-029e2cceb3d0beff7` | Sandbox Ma Account (`161388682021`) | eu-central-1 | Verified |

All five subnets carry the friendly name **Public Subnet (eu-central-1)**; use the raw subnet ID to distinguish them in routing tables, security group rules, and IaC references.

### Evidence

No evidence references are available in this section's context package. Identifier mappings above are sourced directly from CloudoX discovery metadata.

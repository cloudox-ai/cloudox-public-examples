# Security View

> **Demo Report** — This discovery report was produced from a real AWS environment by
> [CloudoX](https://cloudox.io). AWS account IDs and all resource identifiers (VPC,
> subnet, security group, IAM role IDs, KMS key IDs, and similar) have been replaced
> with deterministic synthetic equivalents so the report can be shared publicly.
> Architecture patterns, workload structure, findings, and service configurations are
> otherwise unmodified — this is not dummy data.

---

> **Audience:** Security & Governance teams  
> **Overall confidence:** Likely  
> Exposure, identity and access, governance signals, security findings, and the evidence gaps that need validation.

**This view answers**

- What is exposed, and to whom?
- Where is identity / access over-privileged or unclear?
- What governance or evidence gaps exist?

**Sections**

- [Security Overview](#security-overview)
- [Exposure Summary](#exposure-summary)
- [Identity & Access Risks](#identity-access-risks)
- [Network Access Paths](#network-access-paths)
- [Governance & Coverage Gaps](#governance-coverage-gaps)
- [Security Findings](#security-findings)
- [Validation Questions](#validation-questions)
- [Evidence Appendix](#evidence-appendix)


---

## Security Overview

> **Confidence: Likely** — Evidence is derived from graph discovery; some unknowns and collector limitations remain. Treat findings as a strong working picture, not a complete audit.

The most decision-relevant signal for this assessment: **two security groups are confirmed open to the internet**, an unused administrative role exists in the Sandbox account, and **Security Hub enablement could not be confirmed** — meaning there is no verified centralised findings aggregator in scope. These three facts together represent the highest-priority items for Security & Governance review.

---

### Security Posture

#### Network Exposure

Two security groups with internet-facing rules were identified across the Workload Prod Account (`122122642149`):

| Security Group | Account | Account Friendly Name |
|---|---|---|
| `sg-0459201826f8de5b3` | `122122642149` | Workload Prod Account |
| `sg-06f2b4190bf01d261` | `122122642149` | Workload Prod Account |

A third security group (`sg-054446d655de1ee7f`) is present in the Sandbox Ma Account (`161388682021`). Its internet-exposure status is not separately confirmed in this package beyond its presence as a citeable evidence reference.

**What is exposed, and to whom?** Two production security groups have rules permitting inbound access from the internet. The specific ports, protocols, and attached resources are not detailed in this package and should be validated directly.

#### Identity & Access

72 IAM roles are in scope. **0 customer-managed IAM policies** were discovered — all policy attachment is via AWS-managed or inline policies, which limits the ability to audit least-privilege centrally.

Roles of specific concern identified in evidence:

| Role | Account | Risk Signal |
|---|---|---|
| `cloudox-demo-sandbox-unused-admin` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin`) | Sandbox Ma Account (`161388682021`) | Name indicates unused administrative privilege |
| `OrganizationAccountAccessRole` (`arn:aws:iam::161388682021:role/OrganizationAccountAccessRole`) | Sandbox Ma Account (`161388682021`) | Standard cross-account break-glass role; access path should be reviewed |
| `cloudox-demo-sandbox-scratch-lambda` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-scratch-lambda`) | Sandbox Ma Account (`161388682021`) | Scratch/experimental role; scope of permissions not confirmed |
| `stacksets-exec-899a82a2cb437e970934262f85c7628a` (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`) | Sandbox Ma Account (`161388682021`) | StackSets execution role; lateral movement path if over-permissioned |
| `cloudox-demo-org-trail-logs` (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`) | Management Account (`110319895932`) | Trail log delivery role; integrity depends on its policy scope |
| `AWSServiceRoleForCloudFormationStackSetsOrgAdmin` (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`) | Management Account (`110319895932`) | Org-wide StackSets admin; high blast radius if misused |

**Where is identity over-privileged or unclear?** The `cloudox-demo-sandbox-unused-admin` role is the clearest signal — its name explicitly suggests administrative privilege that is not actively used, which is a remediation candidate. The `OrganizationAccountAccessRole` in the Sandbox account provides a cross-account access path from the Management Account and warrants confirmation that its trust policy is appropriately restricted. The absence of customer-managed policies means least-privilege posture cannot be assessed from policy documents alone.

#### Audit Trail

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the Management Account (`110319895932`) in `eu-central-1`. This is a positive governance signal — org-level trails cover member accounts. Whether log integrity validation, log file validation, or CloudWatch Logs integration is enabled is **not confirmed** in this package.

#### Centralised Security Findings

**No Security Hub enablement was discovered.** This is a material governance gap: without a confirmed aggregator, there is no verified single pane for findings across the organisation's accounts. This should be validated as a priority — it is possible Security Hub is enabled but was not reachable by the collector.

---

### Scope of Assessment

**807 resources** were captured across the following accounts:

| Account | Friendly Name | Confidence | Environment |
|---|---|---|---|
| `110319895932` | Management Account | Verified | — |
| `161388682021` | Sandbox Ma Account | Verified | Sandbox |
| `105769365151` | Workload Dev Account | Verified | Development |
| `122122642149` | Workload Prod Account | Verified | Production |
| `122980216815` | Log Archive Account | Likely | Unknown |
| `110019496666` | Audit Account | Likely | — |
| `150982215529` | Platform Account | Likely | Production |

The Log Archive Account (`122980216815`), Audit Account (`110019496666`), and Platform Account (`150982215529`) are identified with **Likely** confidence — their roles are inferred, not directly confirmed from account metadata in this package.

**0 coverage gaps** were identified within the typed collector coverage. However, two collector limitations constrain the completeness of this picture:

- **Resource Explorer meta-collector was disabled or unavailable** — the AWS-visible resource breadth could not be cross-checked against what was collected. Resources visible to AWS but outside typed collector coverage may be missing.
- **Cloud Control meta-collector was disabled** — long-tail and newer resource types are limited to what typed collectors explicitly support. Novel or less-common resource types may not appear in the 807-resource count.

**What governance or evidence gaps exist?** The combination of no confirmed Security Hub, disabled Resource Explorer cross-check, and disabled Cloud Control meta-collector means the true attack surface breadth is not fully verified. The 807-resource figure should be treated as a lower bound. Enabling these collectors and confirming Security Hub status are the highest-priority evidence gaps to close.


---

## Exposure Summary

**Confidence: Likely** — derived from graph evidence; some unknowns and limitations remain.

The most urgent finding requiring a decision is a sandbox security group with world-open ingress on SSH (port 22) and PostgreSQL (port 5432) — protocols that should never be exposed to the internet. Beyond that critical item, the production Atlas environment carries intentional but unconfirmed internet-facing exposure across a load balancer, two security groups, and public subnets. Four additional public subnets exist across the dev and prod Atlas accounts. All findings are evidence-grounded; account and region metadata are noted where available.

---

### Internet-Reachable Resources

#### Critical — Requires Decision

| Resource | Account | Ports Exposed | Severity |
|---|---|---|---|
| `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) | 161388682021 (Sandbox) | 22 (SSH), 5432 (PostgreSQL) | **Critical** |

Security group `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) has ingress rules permitting `0.0.0.0/0` and/or `::/0` on port 22 (SSH) and port 5432 (PostgreSQL). **This is the only finding in this section flagged `decision_required`.** Unrestricted SSH and database access from the internet represents a direct exploitation path. Confirm whether this exposure is intentional; if not, restrict ingress immediately.

#### Medium — Confirm Intentional

The following resources are internet-reachable and carry medium severity. No immediate decision flag has been raised, but each should be confirmed as intentionally public.

**Load Balancer**

| Resource | Account | Region | Notes |
|---|---|---|---|
| `cloudox-demo-atlas-prod-alb` | Unknown | Unknown | Scheme is internet-facing |

Account and region metadata for `cloudox-demo-atlas-prod-alb` are not available in the current evidence. **Confidence: Unknown** for the account/region attribution of this resource.

**Security Groups (Production Atlas — account 122122642149, eu-central-1)**

| Resource | Ports Exposed |
|---|---|
| `cloudox-demo-atlas-prod-sg-edge` (`sg-0459201826f8de5b3`) | 443 (HTTPS) |
| `cloudox-demo-atlas-prod-sg-alb` (`sg-06f2b4190bf01d261`) | 80 (HTTP) |

Ports 443 and 80 open to `0.0.0.0/0`/`::/0` are consistent with a public-facing web application, but should be confirmed as intentional and reviewed for whether HTTP (port 80) should redirect rather than accept traffic.

**Public Subnets**

Four subnets have route tables forwarding traffic to an Internet Gateway, making them internet-reachable:

| Subnet | Friendly Name | Account | Region |
|---|---|---|---|
| `subnet-013a24d318ed6f3d0` | cloudox-demo-atlas-prod-public-a | 122122642149 | eu-central-1 |
| `subnet-016c22941a019a137` | cloudox-demo-atlas-prod-public-b | 122122642149 | eu-central-1 |
| `subnet-065f522206524ab12` | cloudox-demo-atlas-dev-public-a | 105769365151 | eu-central-1 |
| `subnet-0f64d71c952a7898a` | cloudox-demo-atlas-dev-public-b | 105769365151 | eu-central-1 |

Public subnets in both prod and dev Atlas accounts are present. Resources placed in these subnets inherit internet reachability; the risk depends on what is deployed into them.

---

### Exposure Paths

Three VPCs with confirmed Internet Gateways are in scope:

| VPC | Account | IGW | Region |
|---|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | 122122642149 | `igw-01dd5625ec1ea7a54` | eu-central-1 |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | 105769365151 | `igw-0155958a0a5e9c500` | eu-central-1 |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | 161388682021 | `igw-08017528fa36ccb4d` | eu-central-1 |

The sandbox VPC (`vpc-0c97d53850c027a14`) is the highest-risk path: its Internet Gateway combined with `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) creates a direct internet route to SSH and PostgreSQL.

The Atlas prod and dev VPCs each have Internet Gateways and public subnets. The prod path includes the internet-facing ALB (`cloudox-demo-atlas-prod-alb`) fronted by security groups permitting ports 80 and 443 from any source — consistent with a public web tier, but requiring confirmation.

**Additional VPCs with unknown posture:** Several VPCs were discovered with **Unknown** confidence — including unnamed VPCs in accounts 122980216815, 110019496666, and additional VPCs in accounts 161388682021 and 122122642149 (eu-central-1 and us-east-1). Their exposure posture is not established from current evidence. This is a material governance gap.

> **Governance gap:** No Security Hub enablement was discovered. Without Security Hub, continuous security findings aggregation and standards compliance checks are absent, meaning the exposures above may not be surfaced through any automated detection channel.


---

## Identity & Access Risks

**Section confidence: Likely** — derived from graph evidence; some unknowns and limitations remain. Treat Assumed-confidence findings as requiring manual validation before acting.

The most pressing governance gap across this environment is the near-absence of automated identity and threat monitoring: both IAM Access Analyzer and GuardDuty are enabled in only 1 of 6 scanned accounts, leaving the Management Account, both Workload accounts, the Sandbox, and the Log Archive account operating without these detective controls. Alongside this, a broadly-named administrative IAM role in the Sandbox account warrants immediate review.

### Privilege & Trust

The environment contains 72 IAM roles across the scanned accounts. No customer-managed IAM policies were discovered, which limits visibility into how permissions are composed and governed.

**OrganizationAccountAccessRole presence across member accounts**

The `OrganizationAccountAccessRole` — a high-trust role created by AWS Organizations that grants the Management Account (`110319895932`) full administrative access to member accounts — is present in at least four member accounts:

| Account | Friendly Name | Role ID |
|---|---|---|
| `122980216815` | Log Archive Account | `AROAAAAAB33DHRZ72LLFE` |
| `110019496666` | Audit Account | `AROAAAAACLQX0PDP245PQ` |
| `105769365151` | Workload Dev Account | `AROAAAAAC6A0RHGHCRIWC` |
| `122122642149` | Workload Prod Account | `AROAAAAADGV42TTJ3H83B` |

This is standard AWS Organizations behaviour, but it means any principal in the Management Account that can assume this role has unrestricted access to those member accounts. Governance teams should confirm that access to the Management Account itself is tightly controlled and that use of `OrganizationAccountAccessRole` is logged and reviewed.

**Broadly-privileged IAM role in Sandbox — confidence: Assumed ⚠️**

IAM role `cloudox-demo-sandbox-unused-admin` (`AROAAAAADPCL3BVEXUDTH`) in the Sandbox Ma Account (`161388682021`) carries a name strongly suggesting broad or administrative privilege. The actual attached policies were not collected, so the privilege breadth is inferred from naming alone and **must be validated** before drawing conclusions. If the role does carry admin-level permissions, the blast radius in the Sandbox account could be significant — particularly if the role is assumable from outside the account or has been unused (as the name implies).

> **Recommended action (`risk:security:cloudox-demo-sandbox-unused-admin`):** Review attached policies on `cloudox-demo-sandbox-unused-admin`; apply least-privilege scoping. Confirm whether the role is still needed; if not, remove it.

Additional roles present in the Sandbox account include `OrganizationAccountAccessRole`, `cloudox-demo-sandbox-scratch-lambda`, and `stacksets-exec-899a82a2cb437e970934262f85c7628a`. The StackSets execution role (`stacksets-exec-...`) and the CloudFormation StackSets org-admin service-linked role in the Management Account (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`) indicate that StackSets-based deployment is in use across the organisation — a trust path that should be in scope for governance review.

**Network exposure context**

Two security groups are open to the internet: `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261` in the Workload Prod Account (`122122642149`), and `sg-054446d655de1ee7f` in the Sandbox Ma Account (`161388682021`). The resources attached to these groups are not detailed in this section's evidence; exposure should be assessed in conjunction with the network posture view.

### Access Risks

**IAM Access Analyzer — coverage gap (medium severity, Likely)**

IAM Access Analyzer, which identifies resources shared with external principals, is enabled in only **1 of 6** scanned accounts. The five accounts without coverage are:

| Account ID | Friendly Name |
|---|---|
| `122980216815` | Log Archive Account |
| `105769365151` | Workload Dev Account |
| `122122642149` | Workload Prod Account |
| `110319895932` | Management Account |
| `161388682021` | Sandbox Ma Account |

The absence of Access Analyzer in the Management Account and both Workload accounts is particularly significant: unintended external resource sharing (S3 buckets, IAM roles, KMS keys, etc.) in these accounts would go undetected. The Log Archive Account confidence is Likely rather than Verified.

> **Recommended action (`risk:security:aws-accessanalyzer-analyzer`):** Enable IAM Access Analyzer consistently across all accounts, ideally org-wide via a delegated administrator in the Management Account.

**GuardDuty — coverage gap (medium severity, Likely)**

GuardDuty threat detection is similarly enabled in only **1 of 6** scanned accounts. The same five accounts listed above lack coverage. Without GuardDuty, anomalous API activity, credential compromise, and network-level threats in those accounts produce no automated findings.

> **Recommended action (`risk:security:aws-guardduty-detector`):** Enable GuardDuty org-wide via a delegated administrator. This can be done centrally from the Management Account with minimal per-account effort.

**CloudTrail**

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`) exists in `eu-central-1` under the Management Account (`110319895932`), with a dedicated log-delivery role (`cloudox-demo-org-trail-logs`). This provides a baseline audit record, but the absence of GuardDuty and Access Analyzer means that trail data is not being actively analysed for threats or external-access anomalies in most accounts.

**Security Hub**

No evidence of Security Hub enablement was discovered across any account. This is a material governance gap: without Security Hub, there is no centralised aggregation of findings from GuardDuty, Access Analyzer, or other security services, and no automated compliance standard scoring (e.g. AWS Foundational Security Best Practices, CIS Benchmarks).


---

## Network Access Paths

**Confidence: Verified** — Derived from complete graph evidence for this domain.

The most security-relevant finding here is the combination of internet-facing exposure across multiple environments and a production data tier that sits in a single availability zone. With 25 internet-facing access paths observed and 2 security groups open to the internet, the attack surface is non-trivial and warrants structured review. The sections below break this down by ingress and lateral connectivity.

---

### Ingress Paths

Three VPCs — all in `eu-central-1` — carry internet gateways, meaning each has a direct on-ramp from the public internet:

| VPC | Account | Internet Gateway |
|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | 122122642149 | `igw-01dd5625ec1ea7a54` |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | 105769365151 | `igw-0155958a0a5e9c500` |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | 161388682021 | `igw-08017528fa36ccb4d` |

All three environments — production, development, and sandbox — are internet-reachable. For Security & Governance teams, the presence of an internet gateway on the Dev and Sandbox VPCs alongside Production is a governance signal: if these environments share any credentials, data, or peering paths, the lower-assurance environments could become a stepping stone toward production.

Five public subnets are confirmed across these three VPCs (accounts 122122642149, 105769365151, and 161388682021), all in `eu-central-1`. Resources placed in these subnets are directly reachable from the internet subject to security group and NACL rules.

**Security groups open to the internet:** 2 are confirmed. The package does not identify which resources these groups are attached to, so the precise exposure surface cannot be fully characterised from this evidence alone — this is a material gap (see Unknowns).

**25 internet-facing access paths** are observed in total across the estate. The package does not enumerate each path individually, but the combination of three internet gateways, five public subnets, and two permissive security groups is consistent with this count.

One **DynamoDB Interface Endpoint** (`vpce-02ddbd8d63a6ab447`, account 122122642149, `eu-central-1`) is present in the Prod environment. This routes DynamoDB traffic privately within the VPC rather than over the public internet — a positive signal for that specific service path. No other VPC endpoints are evidenced in this package.

AWS-managed prefix lists `pl-66a5400f` and `pl-6ea54007` (eu-central-1) are referenced in the evidence graph. These are standard AWS-managed prefix lists (typically used for S3 and DynamoDB gateway routing); their presence indicates route table or security group rules reference them, but the package does not confirm the specific rules or their scope.

---

### Lateral Connectivity

The package does not contain direct evidence of VPC peering, Transit Gateway attachments, or other explicit cross-VPC routing between the three environments. Absence of evidence here is not confirmation of isolation — this is a material unknown that should be validated (see Unknowns).

The most significant lateral risk evidenced is the dependency of the **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account 122122642149) on the datastore **`cloudox-demo-atlas-prod-pg`** (`dependency_concern:architecture:cloudox-demo-atlas-prod-pg`). This datastore is not Multi-AZ. While this is primarily a resilience concern, it has a security-governance dimension: a zone failure or targeted disruption of the data tier would take the production API's data layer offline. The recommended action is to confirm whether Multi-AZ or backup controls are in place and to review how the API handles a data-tier failure.

> **Confidence on the workload dependency:** Verified for the dependency relationship. The datastore entity `cloudox-demo-atlas-prod-pg` carries Unknown confidence on its account/region attributes, meaning its precise placement cannot be confirmed from current evidence.

No evidence of Security Hub enablement was discovered. This means centralised, cross-account security findings aggregation cannot be confirmed, and the 25 internet-facing paths and 2 open security groups may not be under active automated monitoring.

![Workload VPC network topology](./diagrams/security-network-topology.png)


---

## Governance & Coverage Gaps

**Confidence: Likely** — Derived from graph evidence; some unknowns and collector limitations remain. Treat gaps below as material until resolved.

The most decision-relevant signal for Security & Governance teams is this: **no AWS Security Hub enablement was discovered** across any of the seven accounts in scope, and two meta-collectors that would allow cross-checking the completeness of this discovery run were both disabled. This means the findings in this view may be incomplete, and the organisation currently lacks a centralised, native AWS security findings aggregator.

### Org Guardrails

Seven accounts are in scope across this organisation:

| Account | Friendly Name | Confidence |
|---|---|---|
| 110319895932 | Management Account | Verified |
| 161388682021 | Sandbox Ma Account | Verified |
| 105769365151 | Workload Dev Account | Verified |
| 122122642149 | Workload Prod Account | Verified |
| 122980216815 | Log Archive Account | Likely |
| 110019496666 | Audit Account | Likely |
| 150982215529 | Platform Account | Likely |

The presence of a dedicated Log Archive Account (`122980216815`) and Audit Account (`110019496666`) is consistent with an AWS Control Tower or Landing Zone structure, though their roles are assessed as **Likely** rather than Verified — governance teams should confirm these designations are enforced by policy.

A CloudTrail organisation trail (`arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the Management Account, with a dedicated logging role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This provides a baseline audit record across the organisation.

The summary data indicates **72 IAM roles** and **0 customer-managed IAM policies** across the discovered scope. The absence of customer-managed policies is notable: it may indicate reliance on AWS-managed policies or inline policies, both of which reduce visibility into the precise permissions granted. Security teams should validate whether this reflects actual policy design or a collector coverage gap.

Two security groups are identified as open to the internet:
- `arn:aws:ec2:eu-central-1:122122642149:security-group/sg-0459201826f8de5b3` (Workload Prod Account)
- `arn:aws:ec2:eu-central-1:122122642149:security-group/sg-06f2b4190bf01d261` (Workload Prod Account)

A third security group (`arn:aws:ec2:eu-central-1:161388682021:security-group/sg-054446d655de1ee7f`) is present in the Sandbox Ma Account. The internet-exposure status of this group is not stated in the available evidence.

**No Security Hub enablement was discovered** (`evidence_gap:security:no-security-hub-enablement-discovered`). This is a significant governance gap: without Security Hub, there is no centralised aggregation of findings from GuardDuty, Inspector, Macie, or third-party integrations. The absence could mean Security Hub is not enabled, or that the collector did not reach it — either outcome requires validation.

### Coverage Gaps

Three collector-level gaps directly limit the completeness of this security view. All three carry **Unknown** confidence, meaning they cannot be resolved from current evidence alone.

**1. No Security Hub data (`evidence_gap:security:no-security-hub-enablement-discovered`)**
No Security Hub enablement was found. If Security Hub is active but was not collected, security findings aggregated there are invisible to this analysis. If it is genuinely not enabled, the organisation lacks a native AWS findings plane.

**2. Resource Explorer meta-collector disabled (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`)**
AWS Resource Explorer was disabled or unavailable during collection. This means the 807 resources captured cannot be cross-checked against what AWS itself considers visible in these accounts. Resources that exist but were not reached by typed collectors would be silently absent.

**3. Cloud Control meta-collector disabled (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`)**
Cloud Control API was not used. Long-tail or newer AWS resource types that lack a dedicated typed collector are therefore not represented. For a security review, this means niche or recently introduced resource types — which may carry their own exposure surface — are out of scope.

Three additional cost-domain gaps are recorded but are not directly relevant to security posture:
- Cost Explorer collection is disabled; spend figures are unavailable (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`).
- CloudWatch utilization metrics are not collected (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`).
- RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`).

These are noted for completeness; they do not affect the security findings in this section but do limit any cost-based anomaly detection that might otherwise surface shadow or rogue resources.

**Recommended actions for governance teams:**
1. Confirm whether Security Hub is enabled across all accounts and, if not, prioritise enablement with delegated admin in the Management or Audit Account.
2. Enable Resource Explorer and re-run discovery to cross-check resource breadth.
3. Enable Cloud Control collection to surface long-tail resource types.
4. Validate the 0 customer-managed IAM policies finding — confirm whether permissions are managed via AWS-managed policies, inline policies, or permission boundaries, and assess whether that meets your governance standard.


---

## Security Findings

**Confidence: Likely** — findings are derived from graph evidence; some unknowns and limitations remain. Treat items below as strong signals requiring validation, not a complete audit.

Two issues stand out as immediate governance concerns: internet-exposed security groups and an unused administrative IAM role. No Security Hub enablement was discovered, which means there is no centralised finding aggregation to rely on — the gaps below may be the tip of the iceberg.

### Findings

#### Internet-Exposed Security Groups

Two security groups in account `122122642149` (eu-central-1) are open to the internet and warrant immediate review:

| Security Group | ARN |
|---|---|
| sg-0459201826f8de5b3 | `arn:aws:ec2:eu-central-1:122122642149:security-group/sg-0459201826f8de5b3` |
| sg-06f2b4190bf01d261 | `arn:aws:ec2:eu-central-1:122122642149:security-group/sg-06f2b4190bf01d261` |

A third security group (`sg-054446d655de1ee7f`) exists in account `161388682021` (eu-central-1). Its internet-exposure status is not confirmed in the available evidence — it is listed as a known entity but no open-to-internet signal was recorded for it.

**What is exposed, and to whom?** The two groups in `122122642149` have rules permitting inbound traffic from `0.0.0.0/0` or `::/0`. The specific ports and protocols, and the resources attached to these groups, are not captured in this package and must be validated directly.

#### IAM Role: Unused Admin Privileges

Account `161388682021` contains a role named `cloudox-demo-sandbox-unused-admin` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin`). The name and evidence signal indicate this role carries administrative permissions and is unused. An unused admin role represents a standing privilege escalation path — any principal that can assume it gains broad access without generating routine activity that would surface in monitoring.

#### IAM Role: Broad Organisational Access

The `OrganizationAccountAccessRole` (`arn:aws:iam::161388682021:role/OrganizationAccountAccessRole`) is present in account `161388682021`. This is the standard AWS cross-account role created when an account is added to an AWS Organization, and it is assumable by the management account. Its presence is expected, but governance teams should confirm: (a) which principals in the management account can assume it, and (b) whether that trust is intentional and scoped appropriately.

#### CloudTrail: Organisation Trail Present

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) exists in account `110319895932`. The associated log-delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`) is also present. This is a positive governance signal — organisation-wide API activity should be captured. However, whether the trail is currently active, whether log file validation is enabled, and whether logs are being monitored or alerted on is **not confirmed** in this package.

#### Other IAM Roles in Scope

The following additional roles were identified and are noted for completeness:

- `cloudox-demo-sandbox-scratch-lambda` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-scratch-lambda`) — a Lambda execution role in the sandbox account. Scope of permissions is not captured here.
- `stacksets-exec-899a82a2cb437e970934262f85c7628a` (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`) — the StackSets execution role in the sandbox account, paired with the org-admin service-linked role (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`) in the management account. Together these enable organisation-wide CloudFormation StackSets deployments. Confirm that the trust policy on the execution role restricts assumption to the expected StackSets service principal only.

The summary indicates **72 IAM roles** exist across the environment. Only the roles listed above are within the evidence boundary of this package. The remaining roles are not characterised here.

#### No Security Hub Enablement Discovered

No evidence of AWS Security Hub being enabled was found in any in-scope account or region. Without Security Hub (or an equivalent), there is no centralised aggregation of findings from GuardDuty, Inspector, Macie, or Config. This is a material governance gap: the findings above are derived from infrastructure graph analysis, not from a live security monitoring pipeline.

### Recommended Mitigations

1. **Restrict or remove internet-open security groups.** For `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261` in account `122122642149`: identify the attached resources, determine whether any inbound rule from `0.0.0.0/0`/`::/0` is intentional, and replace broad rules with source-scoped CIDRs or prefix lists. If no legitimate use case exists, remove the rules.

2. **Delete or disable `cloudox-demo-sandbox-unused-admin`.** An unused role with admin permissions should be removed. If a future use case is anticipated, the role should be created on demand with least-privilege permissions rather than left standing.

3. **Audit `OrganizationAccountAccessRole` trust.** Confirm which IAM principals in the management account (`110319895932`) can assume this role in `161388682021`, and whether MFA or condition keys are enforced on the trust policy.

4. **Validate CloudTrail trail health.** Confirm that `cloudox-demo-org-trail-o-aaaapzvebq` is active, that log file validation is enabled, and that an alerting or SIEM pipeline consumes the logs. The trail and log role exist — operational status is unconfirmed.

5. **Enable AWS Security Hub organisation-wide.** Without a centralised finding service, coverage gaps are invisible. Enabling Security Hub with auto-enrolment for all member accounts, and delegating administration to a dedicated security account, would close the most significant governance gap identified here.

6. **Review StackSets execution role trust.** Confirm that `stacksets-exec-899a82a2cb437e970934262f85c7628a` in `161388682021` trusts only the CloudFormation StackSets service principal and not overly broad principals.

7. **Characterise the remaining 60+ IAM roles.** Only a subset of the 72 discovered roles are covered by this package. A full IAM access review — ideally using IAM Access Analyzer and last-accessed data — is needed to identify further unused or over-privileged roles.


---

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


---

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

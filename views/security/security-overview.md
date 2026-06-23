# Security View — Security Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Security Overview

> **Confidence: Likely** — Derived from graph evidence across discovered resources; some unknowns and collector limitations remain. See gaps noted below before treating this as a complete picture.

The most decision-relevant signal for this assessment: **three security groups are confirmed open to the internet**, an unused administrative role exists in the Sandbox account, and **no Security Hub enablement was discovered** — meaning there is no centralised, AWS-native findings aggregator confirmed to be active across this organisation. These three facts together represent the highest-priority governance concerns surfaced by this view.

---

### Security Posture

#### Network Exposure

Three security groups with internet-facing rules were identified across two accounts:

| Security Group | Account | Account Friendly Name |
|---|---|---|
| `sg-0459201826f8de5b3` | `122122642149` | Workload Prod Account |
| `sg-06f2b4190bf01d261` | `122122642149` | Workload Prod Account |
| `sg-054446d655de1ee7f` | `161388682021` | Sandbox Ma Account |

Two of the three internet-open security groups reside in the **Production environment** (`122122642149-production`). The third is in the **Sandbox environment** (`161388682021-sandbox`). Security & Governance teams should validate the business justification for each open rule, confirm that any exposed ports are intentional, and verify that compensating controls (e.g. WAF, NACLs, host-based firewalling) are in place for the production instances.

#### Identity & Access

**72 IAM roles** were captured across the assessed accounts. No customer-managed IAM policies were discovered — all policy attachment appears to be via AWS-managed or inline policies, which limits the ability to audit custom permission boundaries centrally.

The following roles warrant specific attention:

| Role | Account | Risk Signal |
|---|---|---|
| `cloudox-demo-sandbox-unused-admin` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin`) | Sandbox Ma Account (`161388682021`) | Name indicates an unused administrative role — high-privilege, potentially dormant |
| `OrganizationAccountAccessRole` (`arn:aws:iam::161388682021:role/OrganizationAccountAccessRole`) | Sandbox Ma Account (`161388682021`) | Standard AWS cross-account break-glass role; confirm trust policy restricts assumption to the Management Account only |
| `cloudox-demo-org-trail-logs` (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`) | Management Account (`110319895932`) | Supports the org-level CloudTrail; confirm least-privilege and no unintended trust relationships |
| `AWSServiceRoleForCloudFormationStackSetsOrgAdmin` (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`) | Management Account (`110319895932`) | AWS service-linked role for StackSets org administration; verify StackSets usage is intentional and scoped |
| `cloudox-demo-sandbox-scratch-lambda` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-scratch-lambda`) | Sandbox Ma Account (`161388682021`) | "Scratch" naming suggests an ad-hoc or temporary role; confirm it is still required and appropriately scoped |
| `stacksets-exec-899a82a2cb437e970934262f85c7628a` (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`) | Sandbox Ma Account (`161388682021`) | StackSets execution role in a member account; confirm it is scoped to expected StackSets operations only |

**Priority action:** The `cloudox-demo-sandbox-unused-admin` role should be reviewed immediately. If it is genuinely unused, it represents an unnecessary standing privilege that should be removed or at minimum have its trust policy tightened.

#### Audit & Logging

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the Management Account (`110319895932`), operating from `eu-central-1`. This is a positive governance signal — a single org trail provides consistent API activity logging across member accounts. The associated log-delivery role (`cloudox-demo-org-trail-logs`) should be validated to confirm logs are being delivered to the Log Archive Account (`122980216815`) and that the destination bucket is protected against deletion or modification.

**No Security Hub enablement was discovered.** This is a material gap: without Security Hub, there is no confirmed centralised aggregation of findings from GuardDuty, Config, Inspector, or Macie across the organisation. This should be validated — it is possible Security Hub is enabled but was not reachable by the collector (see gaps below).

---

### Scope of Assessment

#### Accounts in Scope

The following accounts were identified within the assessed organisation. Confidence varies by account:

| Account ID | Friendly Name | Confidence |
|---|---|---|
| `110319895932` | Management Account | Verified |
| `161388682021` | Sandbox Ma Account | Verified |
| `105769365151` | Workload Dev Account | Verified |
| `122122642149` | Workload Prod Account | Verified |
| `122980216815` | Log Archive Account | Likely |
| `110019496666` | Audit Account | Likely |
| `150982215529` | Platform Account | Likely |

Four accounts are **Verified** (directly observed evidence). Three accounts — Log Archive (`122980216815`), Audit (`110019496666`), and Platform (`150982215529`) — are assessed as **Likely**, meaning their presence is inferred but not fully confirmed by direct resource discovery. Governance teams should confirm that the Audit and Log Archive accounts have appropriate SCPs and access controls preventing modification by workload accounts.

#### Resource Coverage

**807 resources** were captured across the assessed scope. No coverage gaps were identified within the typed collector coverage. However, two collector limitations constrain the completeness of this picture:

- **Resource Explorer meta-collector was disabled or unavailable** — the AWS-visible breadth of resources could not be cross-checked against what CloudoX discovered. There may be resource types or regions not covered by typed collectors that are invisible to this assessment.
- **Cloud Control meta-collector was disabled** — long-tail and newer AWS resource types are limited to what typed collectors explicitly support. Niche or recently-launched services may be absent.

These limitations mean the 807-resource count should be treated as a **lower bound**, not a complete inventory. Security teams should independently verify coverage for any services known to be in use that are not reflected in other sections of this view.

#### Known Governance Gaps

| Gap | Implication |
|---|---|
| No Security Hub enablement discovered | No confirmed centralised findings aggregation; detective controls may be fragmented |
| Resource Explorer meta-collector unavailable | Cannot cross-check full AWS resource breadth; unknown resource types may exist |
| Cloud Control meta-collector disabled | Long-tail resource types not covered; newer AWS services may be unassessed |
| No customer-managed IAM policies discovered | Custom permission boundaries cannot be centrally audited; all 72 roles rely on AWS-managed or inline policies |
| Log Archive and Audit account confidence is Likely, not Verified | Full discovery of these accounts was not confirmed; their protective controls cannot be fully assessed from this data |

# Security View — Security Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

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

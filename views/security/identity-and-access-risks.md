# Security View — Identity & Access Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

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

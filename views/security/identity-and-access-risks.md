# Security View — Identity & Access Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Identity & Access Risks

**Confidence: Likely** — findings are derived from graph evidence; some unknowns and naming-based inferences remain. See assumptions and gaps below.

The most pressing governance concern across this environment is the absence of consistent detective controls: both IAM Access Analyzer and GuardDuty are enabled in only **1 of 6 scanned accounts**, leaving the majority of the estate — including the Management Account and Workload Prod Account — without external-access analysis or threat detection. Alongside this, a broadly-named administrative IAM role in the Sandbox account warrants immediate privilege review.

---

### Privilege & Trust

Several IAM roles with elevated or broad trust scope are present across the environment.

**`OrganizationAccountAccessRole` — cross-account administrative trust**

Instances of `OrganizationAccountAccessRole` are present in four member accounts: Log Archive Account (`122980216815`, `AROAAAAAB33DHRZ72LLFE`), Audit Account (`110019496666`, `AROAAAAACLQX0PDP245PQ`), Workload Dev Account (`105769365151`, `AROAAAAAC6A0RHGHCRIWC`), and Workload Prod Account (`122122642149`, `AROAAAAADGV42TTJ3H83B`). This role is the standard AWS Organizations cross-account access mechanism and typically grants full administrative access from the Management Account (`110319895932`). Its presence in production and log-archive accounts means the Management Account is a high-value lateral-movement target — compromise of the Management Account implies administrative access to all four.

Evidence: `arn:aws:iam::161388682021:role/OrganizationAccountAccessRole`

**`cloudox-demo-sandbox-unused-admin` — broadly-privileged role, Sandbox account**

> **Confidence: Assumed** — privilege breadth is inferred from naming only; attached policies have not been collected.

The IAM role `cloudox-demo-sandbox-unused-admin` (`AROAAAAADPCL3BVEXUDTH`) in the Sandbox Ma Account (`161388682021`) carries a name that strongly suggests broad or administrative permissions. The "unused" component of the name may indicate the role is dormant, but an unreviewed administrative role — even if currently inactive — represents a standing blast-radius risk: it can be assumed by any principal with the right trust relationship at any time.

Recommended action: Inspect attached and inline policies to confirm actual permission scope; apply least-privilege scoping or remove the role if it is genuinely unused.

Evidence: `arn:aws:iam::161388682021:role/cloudox-demo-sandbox-unused-admin` | Item: `risk:security:cloudox-demo-sandbox-unused-admin`

**Supporting roles in scope (Management Account)**

Two additional roles in the Management Account (`110319895932`) are present in the evidence set:
- `cloudox-demo-org-trail-logs` (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`) — associated with the organisation-level CloudTrail trail (`arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`), indicating log-delivery trust.
- `AWSServiceRoleForCloudFormationStackSetsOrgAdmin` — the AWS-managed service-linked role for StackSets org-admin delegation.

These roles appear consistent with their named purposes; no anomaly is flagged for them in this section.

**Sandbox scratch Lambda role**

`cloudox-demo-sandbox-scratch-lambda` (`arn:aws:iam::161388682021:role/cloudox-demo-sandbox-scratch-lambda`) is present in the Sandbox Ma Account. No privilege anomaly is flagged for this role in the current evidence set; it is noted for completeness.

---

### Access Risks

Two medium-severity, organisation-wide detective-control gaps are identified. Both affect the same five accounts and share the same recommended remediation path.

| Risk | Severity | Confidence | Accounts Affected |
|---|---|---|---|
| Uneven IAM Access Analyzer coverage | Medium | Likely | 5 of 6 scanned |
| Uneven GuardDuty threat detection coverage | Medium | Likely | 5 of 6 scanned |

**IAM Access Analyzer — 5 accounts uncovered**

IAM Access Analyzer is enabled in 1 of 6 scanned accounts. The five accounts without coverage are: Log Archive Account (`122980216815`), Workload Dev Account (`105769365151`), Workload Prod Account (`122122642149`), Management Account (`110319895932`), and Sandbox Ma Account (`161388682021`). Without Access Analyzer, externally-shared resources (S3 buckets, IAM roles, KMS keys, etc.) in these accounts will not be automatically flagged — reducing the team's ability to detect unintended public or cross-account exposure.

Recommended action: Enable IAM Access Analyzer org-wide via a delegated administrator to ensure consistent, centralised coverage.

Item: `risk:security:aws-accessanalyzer-analyzer`

**GuardDuty — 5 accounts uncovered**

GuardDuty threat detection is enabled in 1 of 6 scanned accounts. The same five accounts listed above are uncovered. This means anomalous API calls, credential exfiltration signals, and network-level threats in those accounts — including the Management Account and Workload Prod Account — will not generate GuardDuty findings.

Recommended action: Enable GuardDuty org-wide via a delegated administrator to ensure consistent threat detection and centralised finding aggregation.

Item: `risk:security:aws-guardduty-detector`

**Security group exposure (Workload Prod Account)**

Two security groups are present in the evidence set for the Workload Prod Account (`122122642149`): `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261`. Their rule configurations are not detailed in this section's package; network exposure analysis is covered in the relevant network/exposure section of this view.

A third security group, `sg-054446d655de1ee7f`, is present in the Sandbox Ma Account (`161388682021`).

> **Governance gap — Security Hub:** No Security Hub enablement was discovered across any scanned account. Security Hub would provide a normalised, aggregated view of findings from GuardDuty, Access Analyzer, and other services. Its absence means there is no single-pane governance layer for security findings today.

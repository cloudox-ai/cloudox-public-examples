# Security View — Security Findings

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

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

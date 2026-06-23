# Security View — Governance & Coverage Gaps

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Governance & Coverage Gaps

**Confidence: Likely** — Derived from graph evidence; some unknowns and collector limitations remain. Treat absences as potential gaps, not confirmed clean states.

The most consequential finding for governance teams is that **no AWS Security Hub enablement was discovered** across the organisation. Combined with two disabled meta-collectors, the picture of what is actually running — and what controls are in place — is incomplete. These gaps should be resolved before drawing conclusions from any other section of this view.

### Org Guardrails

The following accounts are in scope for this assessment (confidence as noted):

| Account | Friendly Name | Confidence |
|---|---|---|
| 110319895932 | Management Account | Verified |
| 161388682021 | Sandbox Ma Account | Verified |
| 105769365151 | Workload Dev Account | Verified |
| 122122642149 | Workload Prod Account | Verified |
| 122980216815 | Log Archive Account | Likely |
| 110019496666 | Audit Account | Likely |
| 150982215529 | Platform Account | Likely |

An organisation-level CloudTrail trail (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) is present in the Management Account (110319895932), with a dedicated delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This indicates that API-level audit logging is configured at the org level — a foundational guardrail.

A CloudFormation StackSets service-linked role (`arn:aws:iam::110319895932:role/aws-service-role/stacksets.cloudformation.amazonaws.com/AWSServiceRoleForCloudFormationStackSetsOrgAdmin`) is present in the Management Account, suggesting that centralised infrastructure deployment via StackSets is in use. A corresponding StackSets execution role is present in the Sandbox Ma Account (`arn:aws:iam::161388682021:role/stacksets-exec-899a82a2cb437e970934262f85c7628a`), consistent with cross-account stack deployment.

The `OrganizationAccountAccessRole` (`arn:aws:iam::161388682021:role/OrganizationAccountAccessRole`) is present in the Sandbox Ma Account. This is the standard AWS-vended break-glass role created when an account is added to an organisation; its continued existence and who can assume it warrants validation.

**Security Hub — Unknown:** No Security Hub enablement was discovered in any account. **Confidence: Unknown.** This means there is no confirmed centralised aggregation of security findings (GuardDuty, Config, Inspector, etc.) visible to CloudoX. This is a material governance gap: either Security Hub is not enabled, or it exists in accounts or regions not reached by the current collection run. This must be validated directly. See `evidence_gap:security:no-security-hub-enablement-discovered`.

### Coverage Gaps

Three collector-level gaps limit the completeness of this assessment. Security and governance teams should treat the findings in all sections as a lower bound, not a complete inventory.

**1. Resource Explorer meta-collector disabled or unavailable** (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`)

AWS Resource Explorer was not available to cross-check the breadth of discovered resources. Resources present in AWS but not reached by typed collectors may be invisible to this view. The true attack surface and resource count could be larger than what is documented here.

**2. Cloud Control meta-collector disabled** (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`)

Long-tail resource types (those not covered by CloudoX's typed collectors) were not collected. Niche or newer AWS service types may be entirely absent from this assessment.

**3. Cost-domain collector gaps** (noted for completeness)

CloudWatch utilisation metrics (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`), Cost Explorer (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`), and several resource-level cost attributes (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`) were not collected. These are primarily cost-domain gaps but are noted here because they indicate the collection run was not fully instrumented — reinforcing that coverage should not be assumed complete.

**Recommended actions for governance teams:**
1. Validate Security Hub status directly in each account, particularly the Management Account (110319895932), Audit Account (110019496666), and Workload Prod Account (122122642149).
2. Review who can assume `OrganizationAccountAccessRole` in the Sandbox Ma Account (161388682021) and confirm it is restricted to authorised principals only.
3. Enable Resource Explorer and Cloud Control collection in a subsequent CloudoX run to close the inventory breadth gap before drawing environment-wide conclusions.

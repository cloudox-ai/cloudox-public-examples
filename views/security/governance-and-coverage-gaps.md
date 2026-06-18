# Security View — Governance & Coverage Gaps

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

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

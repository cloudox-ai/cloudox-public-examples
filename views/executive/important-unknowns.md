# Executive View — Important Unknowns

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Important Unknowns

Four evidence gaps limit the completeness of this view. None require an immediate decision, but two directly constrain cost visibility and should be resolved before any spend optimisation work begins.

### Gaps Leadership Should Know

**Cost data is unavailable.** Cost Explorer collection was disabled during this discovery run, so no spend figures, trends, or savings estimates are present anywhere in this view. All cost-related observations are derived from the discovered architecture only — not from actual billing data. To unlock spend analysis, re-run discovery with cost collection enabled.
(`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`)

**Right-sizing and idle-resource analysis is not possible.** CloudWatch utilisation metrics are not collected in this version of CloudoX, so there are no usage-based recommendations for downsizing or decommissioning resources. The actual utilisation of the 807 discovered resources is unknown.
(`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`)

**Several cost drivers are outside collector coverage.** RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by the current collectors. If any of these are in use, their cost contribution is not reflected in this view.
(`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`)

**Long-tail resource types may be missing.** The Cloud Control meta-collector was disabled, so discovery is limited to explicitly typed collectors. Resource types outside that set will not appear in any section of this view.
(`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`)

> **Confidence: Likely.** The gaps above are confirmed collection limitations, not inferences. Their downstream impact on other sections of this view is real but bounded — security and architecture findings rely on direct resource discovery, which is unaffected by the cost and CloudWatch gaps.

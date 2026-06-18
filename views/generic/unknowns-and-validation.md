# Generic View — Unknowns & Validation Questions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Unknowns & Validation Questions

Two categories of limitation shape how much confidence to place in this view: **evidence gaps** — things the discovery run could not collect — and **validation questions** — things that were observed but need a human to confirm intent. Readers should treat conclusions in affected domains as directional rather than definitive until these gaps are closed.

> **Confidence: Likely** — findings are derived from graph evidence; the gaps below are the primary reason full confidence cannot be claimed.

### What We Couldn't Confirm

#### Cost visibility is absent

Spend figures are entirely unavailable for this run. Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`), so no dollar amounts, trends, or per-service breakdowns can be reported. All cost-related observations in this view are inferred from the discovered architecture — resource patterns known to carry charges — not from actual billing data. (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`)

In addition, CloudWatch utilization metrics were not collected in this version, so idle, underutilized, or right-sizing recommendations based on actual usage are not available. (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`)

Several cost-relevant resource attributes are also outside current collector scope: RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured, so cost drivers for those services are not detected. (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`)

#### Discovery breadth may be incomplete

Two meta-collectors that would have broadened resource coverage were not active during this run:

| Meta-collector | Status | Effect |
|---|---|---|
| Cloud Control | Disabled | Long-tail resource types are limited to typed collector coverage only |
| Resource Explorer | Disabled or unavailable | AWS-visible resource breadth could not be cross-checked |

This means resources of types not covered by a dedicated typed collector may be absent from the graph entirely. (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`, `evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`)

#### Tagging and classification gaps

761 resources carry no Environment / Stage / Tier tag and rely on inference for classification. Environment assignments for those resources should be treated as approximate. No Security Hub enablement was discovered, so security posture findings from that service are not available in this view.

### Questions to Validate

#### IAM role with apparent administrative privileges — sandbox account

An IAM role named **cloudox-demo-sandbox-unused-admin** (role ID `AROAAAAADPCL3BVEXUDTH`) exists in account `161388682021` and appears to carry administrative privileges. Its name suggests it may be unused, but that cannot be confirmed from discovery data alone.

> **Validation question:** Does IAM role `cloudox-demo-sandbox-unused-admin` require its current privilege level, and who owns it?

This is flagged as **Assumed** confidence — the privilege level is inferred, not directly verified. No immediate action is mandated by this view, but the role warrants confirmation from the team responsible for the sandbox account. (`validation_question:security:cloudox-demo-sandbox-unused-admin`)

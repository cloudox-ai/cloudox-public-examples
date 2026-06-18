# FinOps View — Missing Cost Evidence

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Missing Cost Evidence

The cost picture for this environment is incomplete. Spend figures are unavailable because Cost Explorer collection was disabled during the discovery run (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`). Everything in this FinOps view is therefore derived from discovered architecture alone — no dollar amounts, no trend lines, and no validated optimization savings can be stated. Treat all cost driver observations as directional until actual billing data is collected.

> **Confidence: Likely** — Architecture-derived drivers are grounded in real discovered resources, but without spend data, no cost attribution or prioritization by dollar impact is possible.

### Cost Data Gaps

Three distinct gaps limit what can be answered right now:

| Gap | What it blocks |
|---|---|
| Cost Explorer collection disabled | All spend figures, trend analysis, and service-level cost breakdowns (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`) |
| CloudWatch utilization metrics not collected | Idle resource detection, underutilization flags, and right-sizing recommendations based on actual usage (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`) |
| Specific service attributes not captured | RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes — cost drivers for these services are not detected (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`) |

Two additional coverage gaps mean the resource inventory itself may be incomplete, which compounds the cost blind spots: the Cloud Control meta-collector was disabled (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`) and Resource Explorer was unavailable for cross-checking (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`). Resources not discovered cannot be costed.

### What to Enable

To move from architecture-derived observations to a fully evidenced FinOps view, the following collection capabilities should be enabled on the next discovery run:

1. **Cost Explorer collection** — Re-run with `CLOUDOX_COST__ENABLED=true` (or without `--no-collect-costs`). This is the single highest-priority change; it unlocks spend figures, service breakdowns, and dollar-weighted prioritization of every other finding.
2. **CloudWatch utilization metrics** — Once available in a future CloudoX version, this will enable idle and underutilized resource identification and right-sizing candidates grounded in actual usage rather than configuration alone.
3. **Cloud Control and Resource Explorer meta-collectors** — Enabling these will broaden resource coverage and allow cross-checking of inventory completeness, reducing the risk that billable resources are missing from the analysis.

No optimization candidates can be validated or dollar-quantified until at least Cost Explorer collection is active. The architectural cost drivers identified elsewhere in this view should be treated as a prioritization queue to revisit once billing data is available.

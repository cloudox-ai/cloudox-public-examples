# FinOps View — Missing Cost Evidence

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Missing Cost Evidence

The cost picture for this environment is incomplete. Spend figures are entirely unavailable for this run, and several architectural cost drivers fall outside current collector coverage. Every cost observation elsewhere in this view is derived from discovered architecture, not from billing data — treat it as directional, not financial.

*Confidence: Likely — architecture-derived analysis only; no spend data collected.*

### Cost Data Gaps

Three distinct gaps limit what can be answered today:

| Gap | What it means for FinOps |
|---|---|
| **No spend data** — Cost Explorer collection is disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`) | Actual dollar figures, month-over-month trends, and service-level cost breakdowns are unavailable. All cost reasoning in this view is architectural inference only. |
| **No utilization metrics** — CloudWatch metrics are not collected in this version | Idle or underutilised resources cannot be identified from usage data. Right-sizing and waste candidates cannot be validated against actual consumption. |
| **Collector blind spots** — RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured | These are often significant line items. Their cost drivers are not detected and are therefore absent from this analysis. |

Two additional coverage gaps affect the completeness of the resource inventory itself, which in turn limits cost attribution confidence:

- The **Cloud Control meta-collector** was disabled, so long-tail resource types beyond typed collector coverage may be missing from the inventory.
- The **Resource Explorer meta-collector** was disabled or unavailable, meaning the full breadth of AWS-visible resources could not be cross-checked against what was discovered.

The practical consequence: the environment may contain billable resources that are not reflected anywhere in this view.

### What to Enable

To move from architecture-derived inference to evidence-grounded cost analysis, the following collection capabilities should be enabled before the next run:

1. **Enable Cost Explorer collection** — set `CLOUDOX_COST__ENABLED=true` (or remove `--no-collect-costs`). This unlocks actual spend figures, service breakdowns, and trend data across complete billing periods.
2. **Enable CloudWatch utilization metric collection** (when available in your CloudoX version) — required to identify idle resources, validate right-sizing candidates, and quantify waste with confidence.
3. **Enable the Cloud Control and Resource Explorer meta-collectors** — broadens resource inventory coverage and allows cross-checking, reducing the risk of undetected billable resources.

Until these gaps are resolved, cost optimization candidates identified elsewhere in this view should be treated as requiring validation against actual billing data before any action is taken.

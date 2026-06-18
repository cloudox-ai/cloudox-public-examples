# Executive View — Important Unknowns

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Important Unknowns

Four evidence gaps limit the completeness of this analysis. None require an immediate decision, but two directly constrain cost visibility — which leadership should resolve before drawing conclusions about spend or optimization opportunities.

### Gaps Leadership Should Know

| Domain | Gap | Impact on This View |
|--------|-----|---------------------|
| Cost | Cost Explorer collection is disabled | No actual spend figures available; all cost analysis is architecture-derived only |
| Cost | CloudWatch utilization metrics not collected | No idle, underutilized, or right-sizing recommendations possible |
| Cost | RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes not captured | Cost drivers for these services are undetected |
| Coverage | Cloud Control meta-collector was disabled | Long-tail resource types may be missing from the inventory |

**Cost visibility is the most material gap.** Because Cost Explorer collection is disabled, this view cannot report actual spend, trends, or savings estimates — only architectural patterns that are known to carry charges. Enabling cost collection (`CLOUDOX_COST__ENABLED=true`) is the single action that would most improve the fidelity of future runs.

The absence of CloudWatch utilization data means right-sizing and idle-resource analysis cannot be performed in this version of CloudoX. This is a platform-level limitation, not a configuration choice, and should be factored into any cost-optimisation workstream.

The disabled Cloud Control meta-collector means the resource inventory covers only typed collectors. Resources of less common types may be absent from all findings in this view — security, cost, and architecture alike. The true scope of the environment may be broader than what is reflected here.

> **Confidence: Likely** — These gaps are directly evidenced by the discovery run configuration and collector state. Their downstream effect on other sections of this view is real but not fully quantifiable without the missing data.

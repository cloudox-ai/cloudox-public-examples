# Executive View — Cost & Efficiency Signals

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Cost & Efficiency Signals

> **Confidence: Likely** — Spend data is unavailable for this run. The signals below are derived from discovered architecture only; no dollar figures can be confirmed until Cost Explorer collection is enabled.

The single most important action here is a data gap, not a cost finding: billing data collection is currently disabled, meaning leadership has no automated spend visibility through CloudoX. Until that is resolved, cost governance relies entirely on manual Cost Explorer access.

### Where the Money Goes

No spend figures are available for any of the six accounts in scope — Management Account, Sandbox Ma Account, Workload Dev Account, Workload Prod Account, Log Archive Account, or Audit Account. Seven architectural cost drivers were identified from the discovered infrastructure, indicating active resource patterns that carry charges, but exact dollar attribution cannot be provided without billing data.

No evidence found for a breakdown of spend by account, service, or region.

### Efficiency Opportunities

No optimization candidates were surfaced in this run. This is a direct consequence of two collection gaps:

- **Billing data is off.** Cost Explorer collection is disabled, so spend-based anomalies and waste signals cannot be detected.
- **Utilization metrics are not collected.** CloudoX does not collect CloudWatch utilization data in this version, so idle resources, underutilized instances, and right-sizing recommendations based on actual usage are not available.

Additionally, several resource types — including RDS read replicas, RDS provisioned IOPS, DynamoDB capacity modes, Direct Connect, and S3 storage classes — are outside the current collector scope and will not appear as cost drivers even if present.

**Decision needed:** Enable Cost Explorer collection to unlock spend visibility and efficiency analysis across all accounts. Until then, the seven detected architectural cost drivers represent the only available signal, and no savings estimates can be validated.

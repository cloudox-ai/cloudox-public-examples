# FinOps View — Waste / Utilization Signals

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Waste / Utilization Signals

> **Confidence: Likely** — Spend data is unavailable for this run. All observations below are derived from discovered architecture only, not from billing records or utilization metrics. Treat this section as a starting point for investigation, not a confirmed waste inventory.

No optimization candidates were identified from the current discovery pass. The 7 architectural cost drivers noted elsewhere in this view represent patterns that *can* carry charges, but without spend data or utilization metrics, CloudoX cannot confirm whether any specific resource is idle, oversized, or wasteful.

### Idle & Unused Resources

No evidence found for confirmed idle or unused resources in this run. Identifying idle resources — such as unattached EBS volumes, stopped EC2 instances still incurring charges, unused Elastic IPs, or orphaned load balancers — requires either Cost Explorer data or CloudWatch utilization metrics, neither of which is available here.

**What to validate manually:**
- Unattached or snapshot-only EBS volumes
- EC2 instances in a stopped state (storage charges continue)
- Elastic IPs not associated with a running instance
- Load balancers with no registered targets

### Utilization Signals

No evidence found for utilization-based signals in this run. CloudoX does not collect CloudWatch metrics in this version, so CPU, memory, network, and storage utilization data are not available. Right-sizing recommendations and underutilization flags cannot be generated without this input.

**Known collection gaps that limit this section:**
- Cost Explorer is disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`); no spend figures are available to correlate with resource inventory.
- CloudWatch utilization metrics are not collected in this version; idle and underutilization signals are not detectable.
- RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are outside current collector coverage — cost drivers for these services will not appear here.

Enabling Cost Explorer collection and, where available, integrating AWS Compute Optimizer or Trusted Advisor findings would materially improve the signal quality in this section on the next run.

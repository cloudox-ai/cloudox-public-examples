# Generic View — Cost Signals

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Cost Signals

> **Confidence: Likely** — Spend data is unavailable for this run. All signals below are derived from the discovered architecture; no dollar figures or exact cost attributions are presented.

Seven architectural patterns have been identified across the environment that are known to carry AWS charges. No optimization candidates were detected from the available data. Because Cost Explorer collection is disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`), spend totals, trends, and per-service breakdowns cannot be shown — those require enabling cost collection in a future run.

The environment spans seven accounts: **Management Account** (`110319895932`), **Sandbox Ma Account** (`161388682021`), **Workload Dev Account** (`105769365151`), **Workload Prod Account** (`122122642149`), **Log Archive Account** (`122980216815`), **Audit Account** (`110019496666`), and **Platform Account** (`150982215529`). Cost drivers may exist across any of these accounts; attribution to individual accounts is not possible without spend data.

### Cost Drivers

The following architectural cost driver categories were identified from the discovered resources. These describe resource patterns known to carry charges; they are not dollar attributions.

| # | Driver Category | Notes |
|---|----------------|-------|
| 1 | Compute (EC2 / containers) | Running instances and container workloads incur continuous hourly charges based on instance family and size. |
| 2 | Managed databases (RDS / Aurora) | Provisioned database instances accrue instance-hour and storage charges regardless of utilization. |
| 3 | Object storage (S3) | Storage volume, request rates, and data retrieval all contribute to S3 costs; storage class breakdown is not captured by current collectors. |
| 4 | Data transfer | Cross-region, cross-AZ, and internet egress traffic generate transfer charges; exact volumes are not available. |
| 5 | Logging and observability | CloudWatch Logs ingestion and retention, and log delivery to the Log Archive Account (`122980216815`), are recurring cost contributors. |
| 6 | Networking (NAT Gateway / VPC endpoints) | NAT Gateway data-processing and hourly charges are a common hidden cost driver in multi-account VPC architectures. |
| 7 | Security and compliance services | Services such as GuardDuty, Security Hub, and Config (likely active given the presence of an Audit Account (`110019496666`)) carry per-event or per-resource charges. |

> **Note:** RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by the current collectors and are therefore not reflected above.

### Optimization Signals

No optimization candidates were detected from the available architectural data.

This is a data-coverage limitation, not a confirmation that the environment is fully optimized. Two key gaps prevent meaningful optimization analysis:

- **No spend data** — Cost Explorer collection is disabled; there are no cost anomalies, top-spend services, or waste signals to surface.
- **No utilization metrics** — CloudoX does not collect CloudWatch utilization metrics in this version, so idle resources, underutilized instances, and right-sizing opportunities based on actual usage cannot be identified.

Enabling cost collection and utilization metric gathering in a future run will unlock both spend-based and usage-based optimization signals.

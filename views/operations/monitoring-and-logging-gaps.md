# Operations View — Monitoring & Logging Gaps

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Monitoring & Logging Gaps

Three collector-level blind spots limit what this Operations view can assert with confidence: Security Hub enablement is unconfirmed, two meta-collectors were disabled during discovery, and no CloudWatch utilisation data was ingested. Each gap directly affects what can break without warning and what recovery risks remain invisible.

### Logging Coverage

One organisation-level CloudTrail trail is present (`cloudox-demo-org-trail-o-aaaapzvebq`, `arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`), with a dedicated log-delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This is the only logging signal confirmed in the discovery data.

**Security Hub** — No Security Hub enablement was discovered across any account in scope (`evidence_gap:security:no-security-hub-enablement-discovered`). Without Security Hub, there is no centralised aggregation of GuardDuty, Config, Inspector, or IAM Access Analyzer findings. Operational teams have no single pane for security-event triage, and automated remediation workflows that depend on Security Hub findings cannot be assumed to exist.

**Discovery breadth caveat** — Two meta-collectors that would cross-check the completeness of the above picture were not active:
- The **Resource Explorer** meta-collector was disabled or unavailable, so the full AWS-visible resource breadth could not be independently verified (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`).
- The **Cloud Control** meta-collector was disabled, meaning long-tail resource types (those not covered by typed collectors) may be absent from the inventory (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`).

The headline figure of 807 resources captured should be read as a lower-bound estimate, not a complete inventory. Resources outside typed-collector coverage are not represented.

### Monitoring Gaps

**CloudWatch utilisation metrics** — CloudoX did not collect CloudWatch metrics in this run (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`). This means:
- No CPU, memory, network, or storage utilisation baselines are available.
- Idle or underutilised resources cannot be identified from this data.
- Alarm coverage (whether alarms exist and are in OK/ALARM state) is unknown.

**Cost Explorer** — Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`) (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`). Spend figures are unavailable; any cost-related observations in this view are derived from architecture shape alone, not actual billing data.

**Collector coverage gaps for specific services** — RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by the current collectors (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`). Operational attributes tied to these dimensions (e.g., replica lag, IOPS saturation, DynamoDB throttling mode) are therefore not visible in this view.

| Gap | Operational Risk | Recommended Action |
|---|---|---|
| Security Hub not discovered | No centralised finding aggregation; security events may go undetected | Verify enablement; enable if absent |
| Resource Explorer disabled | Inventory may be incomplete; unknown resources could be unmonitored | Re-run discovery with Resource Explorer enabled |
| Cloud Control disabled | Long-tail resource types absent from inventory | Re-run discovery with Cloud Control enabled |
| CloudWatch metrics not collected | No utilisation baselines; alarm state unknown | Enable CloudWatch collection in next CloudoX run |
| Cost Explorer disabled | Spend anomalies and budget overruns not detectable from this data | Enable cost collection (`CLOUDOX_COST__ENABLED=true`) |
| RDS/DynamoDB/S3/Direct Connect collector gaps | Capacity and performance attributes for these services are blind spots | Await typed-collector updates or use Cloud Control |

> **Confidence note:** All items in this section are rated *Unknown* — they represent the absence of evidence rather than confirmed findings. Operational decisions that depend on these signals should not be made until the gaps are resolved.

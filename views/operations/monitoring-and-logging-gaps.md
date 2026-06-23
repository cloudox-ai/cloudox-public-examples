# Operations View — Monitoring & Logging Gaps

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Monitoring & Logging Gaps

Three collector-level gaps limit the completeness of this discovery run. Operations teams should treat the findings in this view as covering the typed-collector surface only — an unknown quantity of long-tail resource types may not be represented. Security Hub enablement status is also unresolved, which is the most operationally significant gap for alert-routing and compliance posture.

### Logging Coverage

One org-level CloudTrail trail is present in `eu-central-1` (`arn:aws:cloudtrail:eu-central-1:110319895932:trail/cloudox-demo-org-trail-o-aaaapzvebq`) with a dedicated delivery role (`arn:aws:iam::110319895932:role/cloudox-demo-org-trail-logs`). This is the only logging infrastructure captured in this section's evidence.

**Coverage cross-check is incomplete.** Two meta-collectors that would normally validate discovery breadth were not available during this run:

| Meta-collector | Status | Operational impact |
|---|---|---|
| AWS Resource Explorer | Disabled / unavailable | Cannot cross-check whether all resource types in the environment were reached by typed collectors |
| AWS Cloud Control API | Disabled | Long-tail resource types (those without a dedicated typed collector) are not inventoried |

This means the 807 resources captured represent the typed-collector surface. Resources outside that surface — and any logging or monitoring attached to them — are not visible in this run. (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`, `evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`)

### Monitoring Gaps

**Security Hub — no enablement discovered.** The discovery run found no evidence that AWS Security Hub is enabled in any account in scope. This is an Unknown-confidence finding: it may mean Security Hub is genuinely absent, or it may be outside typed-collector coverage. Either way, it represents a gap in centralised security alerting and should be validated directly. (`evidence_gap:security:no-security-hub-enablement-discovered`)

**CloudWatch utilisation metrics — not collected.** CloudoX does not collect CloudWatch utilisation metrics in this version. Idle, underutilised, or right-sizing signals based on actual resource usage are not available. Operational teams relying on this view for capacity or incident triage should supplement with direct CloudWatch access. (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`)

**Cost Explorer and resource-level cost signals — not collected.** Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`), so spend figures are unavailable. Additionally, cost-relevant configuration details for RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by current collectors. These gaps do not affect operational monitoring directly but limit the ability to correlate resource changes with spend anomalies. (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`, `evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`)

**Summary of gaps requiring operational action:**

| Gap | Action |
|---|---|
| Security Hub enablement unknown | Verify directly in each account; enable if absent |
| Resource Explorer meta-collector disabled | Re-run with Resource Explorer enabled to validate discovery breadth |
| Cloud Control meta-collector disabled | Re-run with Cloud Control enabled to surface long-tail resource types |
| CloudWatch metrics not collected | Use CloudWatch console or CLI for utilisation and alarm state |
| Cost Explorer disabled | Enable cost collection in CloudoX or review AWS Cost Explorer directly |

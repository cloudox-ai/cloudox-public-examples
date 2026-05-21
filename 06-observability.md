# Observability – Logging, Monitoring & Operational Visibility

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

## CloudTrail – Audit Logging

A single organization-wide trail, `cloudox-demo-org-trail-o-aaaapzvebq` (account `110319895932`, home region `eu-central-1`), is present with the following properties confirmed directly in the resource data:

| Property | Value |
|---|---|
| `is_organization_trail` | `true` |
| `is_multi_region_trail` | `true` |
| `include_global_service_events` | `true` |
| `log_file_validation_enabled` | `true` |
| `s3_bucket_name` | `cloudox-demo-cloudtrail-122980216815-eu-central-1` |
| `cloud_watch_logs_log_group_arn` | `arn:aws:logs:eu-central-1:110319895932:log-group:/aws/cloudtrail/cloudox-demo-org-trail:*` |

The trail resource was discovered via accounts `122980216815`, `110019496666`, `105769365151`, `122122642149`, and `161388682021`, consistent with an organization trail visible across member accounts. The trail's **activation status is not present** in the resource data; the resource exists but its running state should be validated with the customer.

The destination S3 bucket (`cloudox-demo-cloudtrail-122980216815-eu-central-1`) resides in the `log-archive` account (`122980216815`), which is in the `/Security` OU — consistent with a centralised log archive pattern, though no relationship between the trail and the bucket was detected in the `relationships` block.

**CloudTrail log group** (`/aws/cloudtrail/cloudox-demo-org-trail`, account `110319895932`):

| Field | Value |
|---|---|
| `retention_in_days` | 90 |
| `metric_filter_count` | 0 |
| `stored_bytes` | 0 |

`stored_bytes: 0` means no bytes were observed in the log group at discovery time. This may indicate the trail is not currently delivering to CloudWatch Logs, or the group was recently created. No metric filters are present on this log group, so no CloudWatch alarms are driven from CloudTrail events.

---

## CloudWatch Alarms

Seven alarms are present across the graph (`resource_count_by_type: AWS::CloudWatch::Alarm: 7`). All seven are accounted for in the resource list.

### workload-prod (`122122642149`) – 6 alarms

All six alarms route to SNS topic `arn:aws:sns:eu-central-1:122122642149:cloudox-demo-atlas-prod-ops-alerts` (eu-central-1) or `arn:aws:sns:us-east-1:122122642149:cloudox-demo-atlas-prod-dr-ops-alerts` (us-east-1). All report `state_value: OK` at discovery time.

| Alarm | Metric | Namespace | Threshold | State |
|---|---|---|---|---|
| `cloudox-demo-atlas-prod-dlq-depth` | `ApproximateNumberOfMessagesVisible` | `AWS/SQS` | ≥ 1 | OK |
| `cloudox-demo-atlas-prod-ecs-running-task-low` | `RunningTaskCount` | `AWS/ECS` | < 1 (3 periods) | OK |
| `cloudox-demo-atlas-prod-lambda-errors` | `Errors` | `AWS/Lambda` | ≥ 1 | OK |
| `cloudox-demo-atlas-prod-rds-cpu-high` | `CPUUtilization` | `AWS/RDS` | ≥ 80% (2 periods) | OK |
| `cloudox-demo-atlas-prod-rds-storage-low` | `FreeStorageSpace` | `AWS/RDS` | < 2 GiB | OK |
| `cloudox-demo-atlas-prod-dr-bucket-size` | `BucketSizeBytes` | `AWS/S3` | > 5 GiB (us-east-1) | OK |

The DR bucket-size alarm (`cloudox-demo-atlas-prod-dr-bucket-size`) is in `us-east-1`, suggesting a cross-region DR footprint in `workload-prod`. See `04-networking.md` and `02-architecture-overview.md` for context.

No alarms were detected in `workload-dev` (`105769365151`), `platform-prod` (`150982215529`), `sandbox-ma` (`161388682021`), or the management account (`110319895932`).

### audit (`110019496666`) – 1 alarm

| Alarm | Metric | Namespace | Threshold | State |
|---|---|---|---|---|
| `cloudox-demo-security-findings-rate` | `NumberOfNotificationsFailed` | `AWS/SNS` | ≥ 1 | OK |

This alarm monitors SNS delivery failures rather than a security-finding rate metric directly. The name suggests intent to track security findings, but the metric (`NumberOfNotificationsFailed` on `AWS/SNS`) measures notification delivery failures. Validate whether this is the intended signal or a misconfiguration.

---

## Log Groups

Eleven log groups are present across the graph (`AWS::Logs::LogGroup: 11`). Key details:

| Log Group | Account | Retention | KMS | Metric Filters | Stored Bytes |
|---|---|---|---|---|---|
| `/aws/cloudtrail/cloudox-demo-org-trail` | `110319895932` | 90 days | — | 0 | 0 |
| `/cloudox/cloudox-demo/log-archive/operations` | `122980216815` | 30 days | — | 0 | 0 |
| `/cloudox/cloudox-demo/security-ops` | `110019496666` | 60 days | — | 0 | 0 |
| `/aws/apigateway/cloudox-demo-atlas-prod-api` | `122122642149` | 90 days | `1ac02cc5-a23f-75da-30c9-99c095a072fa` | 0 | 0 |
| `/aws/ecs/cloudox-demo-atlas-prod-api` | `122122642149` | 30 days | `1ac02cc5-a23f-75da-30c9-99c095a072fa` | 0 | 0 |
| `/aws/ecs/containerinsights/cloudox-demo-atlas-prod/performance` | `122122642149` | **1 day** | — | 0 | 0 |
| `/aws/lambda/cloudox-demo-atlas-prod-api` | `122122642149` | 90 days | `1ac02cc5-a23f-75da-30c9-99c095a072fa` | 0 | 0 |
| `/cloudox/cloudox-demo/atlas-prod-dr/operations` | `122122642149` | 90 days | `c7e5f429-9c4e-a068-d93a-52ebe27c0564` | 0 | 0 |
| `/aws/ecs/cloudox-demo-atlas-dev-api` | `105769365151` | 7 days | — | 0 | 0 |
| `/aws/lambda/cloudox-demo-atlas-dev-api` | `105769365151` | 7 days | — | 0 | 0 |
| `/aws/lambda/cloudox-demo-sandbox-scratch` | `161388682021` | **1 day** | — | 0 | 0 |

**Observations:**

- `stored_bytes: 0` on every log group. This may reflect a recently provisioned environment or that services have not yet written logs. It should be validated — it does not confirm services are inactive.
- `metric_filter_count: 0` on all log groups. No CloudWatch alarms are driven from log-based metrics anywhere in the environment.
- `/aws/ecs/containerinsights/cloudox-demo-atlas-prod/performance` has a 1-day retention period. If Container Insights is active, performance data would be discarded after 24 hours.
- `/aws/lambda/cloudox-demo-sandbox-scratch` has a 1-day retention period, consistent with a scratch/ephemeral workload.
- KMS encryption is present on prod log groups in `122122642149` (key `1ac02cc5-...` for eu-central-1, `c7e5f429-...` for us-east-1 DR). Dev and security-ops log groups carry no `kms_key_id` field in the discovery data.

---

## EventBridge Rules

Two EventBridge rules are present (`AWS::Events::Rule: 2`).

| Rule | Account | State | Description |
|---|---|---|---|
| `cloudox-demo-guardduty-to-sns` | `110019496666` | **ENABLED** | Forwards `GuardDuty Finding` events to SNS topic `cloudox-demo-security-findings` |
| `cloudox-demo-atlas-dev-housekeeping` | `105769365151` | **DISABLED** | Daily housekeeping schedule (`rate(1 day)`) |

The `cloudox-demo-atlas-dev-housekeeping` rule is explicitly `state: DISABLED`. The description states "Daily housekeeping rule (DISABLED in dev demo)" — validate whether this is intentional for the demo environment or represents a gap in dev operations.

The GuardDuty-to-SNS rule (`cloudox-demo-guardduty-to-sns`) is enabled and targets `cloudox-demo-security-findings` (SNS, account `110019496666`). A GuardDuty detector resource (`AWS::GuardDuty::Detector: 1`) is present in the graph, but no status field was present in the discovery data for that resource — its activation state is unknown.

---

## SNS Topics

Six SNS topics are present (`AWS::SNS::Topic: 6`). All show `SubscriptionsConfirmed: 0`.

| Topic | Account | Region | KMS |
|---|---|---|---|
| `cloudox-demo-security-findings` | `110019496666` | eu-central-1 | `0658435e-522a-73c7-2bd9-7d7e1131248b` |
| `cloudox-demo-log-archive-notifications` | `122980216815` | eu-central-1 | `d1b59baf-b18a-b78f-1a08-7254c874d325` |
| `cloudox-demo-atlas-prod-ops-alerts` | `122122642149` | eu-central-1 | `1ac02cc5-a23f-75da-30c9-99c095a072fa` |
| `cloudox-demo-atlas-prod-dr-ops-alerts` | `122122642149` | us-east-1 | `c7e5f429-9c4e-a068-d93a-52ebe27c0564` |
| `cloudox-demo-atlas-dev-app-notifications` | `105769365151` | eu-central-1 | — |
| `cloudox-demo-sandbox-orphan` | `161388682021` | eu-central-1 | — |

**`SubscriptionsConfirmed: 0` on every topic.** No confirmed subscribers exist on any SNS topic at discovery time. This means no alarm notifications, GuardDuty findings, or operational alerts would be delivered to any endpoint (email, PagerDuty, Lambda, etc.) at the time of discovery. This applies to the prod ops-alerts topics as well as the security-findings topic.

`cloudox-demo-log-archive-notifications` (account `122980216815`) has a policy granting `sns:Subscribe` and `sns:GetTopicAttributes` to `arn:aws:iam::110019496666:root` (the audit account), but `SubscriptionsPending: 0` — no subscription has been initiated.

`cloudox-demo-sandbox-orphan` carries no KMS key and uses the default SNS policy. Its name suggests it may be an unmanaged or leftover resource; validate with the customer.

---

## Backup / DR

An AWS Backup plan and vault are present in `workload-prod` (`122122642149`):

- **Vault:** `cloudox-demo-atlas-prod-vault` — encrypted with KMS key `1ac02cc5-a23f-75da-30c9-99c095a072fa`, `locked: false`, `number_of_recovery_points: 0`.
- **Plan:** `cloudox-demo-atlas-prod-daily` — rule `daily-7-day-retention`, schedule `cron(0 5 * * ? *)` UTC, 7-day lifecycle delete, 60-minute start window, 180-minute completion window.
- **Selection:** targets `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items` via IAM role `cloudox-demo-atlas-prod-backup`.

`number_of_recovery_points: 0` means no backups have completed at discovery time. This may reflect a newly provisioned plan; validate whether the schedule has run and whether recovery points are expected.

No backup plan or vault was detected in `workload-dev`, `platform-prod`, `sandbox-ma`, or the management account.

---

## Security Monitoring Signals

- **GuardDuty detector** (`AWS::GuardDuty::Detector: 1`) is present in the graph. No status field was available; activation state is unknown.
- **EventBridge rule** `cloudox-demo-guardduty-to-sns` (account `110019496666`, `state: ENABLED`) routes GuardDuty findings to `cloudox-demo-security-findings` SNS topic — but that topic has `SubscriptionsConfirmed: 0`, so findings would not reach any subscriber at discovery time.
- **Access Analyzer** (`AWS::AccessAnalyzer::Analyzer: 1`) is present. No status field was available; activation state is unknown.
- **Log group** `/cloudox/cloudox-demo/security-ops` (account `110019496666`, 60-day retention) exists with `stored_bytes: 0` and `metric_filter_count: 0`. No evidence of what writes to this group.
- No Security Hub hub resource was detected in the provided discovery data.

---

## Unknowns / Gaps

| Area | Gap | Follow-up |
|---|---|---|
| CloudTrail activation | No `status` or `is_logging` field present on `cloudox-demo-org-trail-o-aaaapzvebq` | Confirm trail is actively logging via `aws cloudtrail get-trail-status` |
| CloudTrail log delivery | `stored_bytes: 0` on `/aws/cloudtrail/cloudox-demo-org-trail` | Verify CloudWatch Logs delivery is functioning; check S3 bucket for objects |
| GuardDuty activation | `AWS::GuardDuty::Detector: 1` present but no status field | Confirm detector is enabled and which accounts it covers |
| Access Analyzer activation | `AWS::AccessAnalyzer::Analyzer: 1` present but no status field | Confirm analyzer type (account vs. organization) and active state |
| Security Hub | No hub resource detected | Confirm whether Security Hub is deployed; not detected does not confirm it is absent |
| SNS subscriptions | All 6 topics have `SubscriptionsConfirmed: 0` | Identify intended subscribers for prod ops-alerts and security-findings topics; confirm subscriptions are pending or missing |
| Metric filters | Zero metric filters across all 11 log groups | Confirm whether log-based alerting is intentionally absent or not yet configured |
| Container Insights retention | `/aws/ecs/containerinsights/cloudox-demo-atlas-prod/performance` has 1-day retention | Confirm whether this is intentional given the prod context |
| Backup recovery points | `number_of_recovery_points: 0` on `cloudox-demo-atlas-prod-vault` | Confirm backup plan has executed at least once; verify RDS instance is also covered |
| `cloudox-demo-sandbox-orphan` | No KMS, no subscriptions, name suggests unmanaged | Confirm ownership and whether this topic should be retained or removed |
| Dev housekeeping rule | `cloudox-demo-atlas-dev-housekeeping` is `DISABLED` | Confirm whether this is intentional for the demo or represents a missing operational process |
| `platform-prod` observability | No log
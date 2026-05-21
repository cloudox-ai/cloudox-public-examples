# Assumptions and Unknowns

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

## Assumptions Made in This Report

The following assumptions underpin claims made across the companion documents. Each should be validated with the customer at the kickoff workshop.

| # | Assumption | Basis | Confidence | How to Validate |
|---|-----------|-------|-----------|----------------|
| A1 | `platform-prod` (account `150982215529`) is a production-grade account | Account alias `platform-prod`; OU path `/Platform`; environment classification inferred as `production` | Medium — classification is `explicit: false` | Confirm account purpose and lifecycle with the platform team |
| A2 | The `audit` (account `110019496666`) and `log-archive` (account `122980216815`) accounts serve a centralised security/logging function | Both reside in the `/Security` OU; account aliases match common Landing Zone patterns | Medium — no workload or service-level data was collected from these accounts | Confirm what services are deployed and who owns them |
| A3 | The `cloudox-demo-atlas-prod` workload is internet-facing via `cloudox-demo-atlas-prod-alb` | Finding `exposure:AWS::ElasticLoadBalancingV2::LoadBalancer:cloudox-demo-atlas-prod-alb`; workload anchor in account `122122642149` | High | Confirm ALB DNS name and whether it is publicly resolvable |
| A4 | The `cloudox-demo-atlas-dev` and `cloudox-demo-atlas-prod` workloads represent the same application in different lifecycle stages | Shared naming prefix `cloudox-demo-atlas-*`; dev in account `105769365151` (`/Workloads`), prod in account `122122642149` (`/Workloads`) | Medium — naming convention only; no explicit tag or relationship links the two | Confirm application ownership and whether dev/prod share any data or infrastructure |
| A5 | `cloudox-demo-sandbox-scratch` is an experimental or throwaway workload | Account `161388682021` is in the `/Sandbox` OU; workload confidence is `low`; anchored on a single Lambda function | Low | Confirm whether this workload is active, who owns it, and whether it holds any sensitive data |
| A6 | The management account (`110319895932`, alias `CloudoX`) is not used to host production workloads | No workloads were detected in this account; it is the organisation management account | Low — absence of detected workloads does not prove absence of workloads | Confirm that no application resources are deployed directly in the management account |
| A7 | The 12 CloudTrail trail resources and 1 GuardDuty detector resource represent active, functioning services | Resources of these types are present in the graph (`AWS::CloudTrail::Trail` count 12, `AWS::GuardDuty::Detector` count 1) | Low — resource presence does not confirm enablement or active status; no status fields were available | Retrieve trail `IsLogging` status and GuardDuty detector `Status` field directly |

---

## Unknowns and Gaps

The following gaps affect the confidence of specific sections in this report. They are grouped by the area they impact most.

### Account and Environment Structure (`03-environment-structure.md`)

- **Environment classification for three accounts is `unknown`**: accounts `110319895932` (management), `110019496666` (audit), and `122980216815` (log-archive) all carry `confidence: low` and `explicit: false`. Their roles are inferred from naming and OU placement only.
- **No OU structure below Root is fully mapped**: only four OUs are present in the graph (`AWS::Organizations::OrganizationalUnit` count 4). Whether additional OUs or accounts exist outside the discovered set is not known.
- **Organisation policies**: five `AWS::Organizations::Policy` resources are present, but their content, type (SCP, tag policy, backup policy), and attachment targets are not available in the provided data.

### Architecture and Workloads (`02-architecture-overview.md`)

- **Workload boundaries are inferred, not declared**: all five workloads carry `confidence: medium` or `low`. No explicit tagging or service-catalogue data was available to confirm boundaries or ownership.
- **Cross-account relationships are not detected**: no relationship data was retained for this section (`relationship_count_kept_for_section: 0`). Whether the dev and prod workloads share ECR repositories, RDS instances, secrets, or KMS keys across accounts is unknown.
- **Three ECR repositories** (`AWS::ECR::Repository` count 3) are present but not linked to any specific workload or ECS service in the available data.
- **One RDS instance** (`AWS::RDS::DBInstance` count 1) is present. Its account, region, encryption status, Multi-AZ configuration, and workload association are not available in the filtered context.
- **Four Secrets Manager secrets** (`AWS::SecretsManager::Secret` count 4) are present. Which workloads consume them and whether rotation is enabled is not known.

### Networking (`04-networking.md`)

- **15 VPCs detected** but VPC peering, Transit Gateway attachments, and inter-account connectivity topology are not confirmed from the available data. The 15 `AWS::Route53Resolver::ResolverRule` and 15 `AWS::Route53Resolver::ResolverRuleAssociation` resources suggest DNS forwarding is in use, but the resolver topology and hub account are not confirmed.
- **Five public subnets identified** via route-table findings, but which resources (if any) are deployed directly into those subnets is not determinable from the filtered context.
- **Four VPC endpoints** are present; their service targets and associated policies are not available.

### Security (`05-security-overview.md`)

- **GuardDuty detector status unknown**: one `AWS::GuardDuty::Detector` resource exists, but no `Status` field was available. Whether it is enabled, which accounts it covers, and whether it is centrally managed from the `audit` account is not known.
- **Security Hub status unknown**: no `AWS::SecurityHub::Hub` resource was detected. It is not possible to determine from the available data whether Security Hub is deployed.
- **AWS Config status unknown**: no `AWS::Config::ConfigurationRecorder` or `AWS::Config::DeliveryChannel` resource was detected. Whether Config is recording in any account is not known.
- **Access Analyzer status unknown**: one `AWS::AccessAnalyzer::Analyzer` resource is present, but its `Status`, type (account vs. organisation), and findings are not available.
- **IAM posture is partially visible**: 72 IAM roles, 7 inline policies, and 1 IAM user are present. Privilege boundaries, permission boundaries, and whether the single IAM user (`AWS::IAM::User` count 1) has console access or active access keys are not known.
- **KMS key usage**: 14 KMS keys and 153 aliases are present. Which keys are customer-managed vs. AWS-managed, and which resources they protect, is not determinable from the available data.

### Observability (`06-observability.md`)

- **CloudTrail logging status unknown**: 12 `AWS::CloudTrail::Trail` resources are present across the organisation, but no `IsLogging` status field was available for any trail. Whether logs are being delivered to the `log-archive` account is not confirmed.
- **Log Groups are sparse**: 11 `AWS::Logs::LogGroup` resources are present across an 800-resource environment. Whether application and VPC flow logs are being captured is not known.
- **7 CloudWatch Alarms** are present but their targets, thresholds, and notification endpoints (6 SNS topics are present) are not linked in the available data.
- **No X-Ray, Container Insights, or APM signals** were detected. Whether distributed tracing is in use for the ECS or Lambda workloads is not known.

### Backup and Resilience

- **One Backup Plan and one Backup Vault** are present (`AWS::Backup::BackupPlan` count 1, `AWS::Backup::BackupVault` count 1). Which resources are covered, the retention schedule, and whether cross-account or cross-region copies are configured are not available in the provided data.

---

## Workshop Questions

The following questions are prioritised by their impact on understanding the environment and planning next actions. Aim to answer these before or during the kickoff workshop.

### Ownership and Structure

1. Who owns the `platform-prod` account (`150982215529`) and what services does it host? The account has no detected workloads but carries a production classification.
2. Are the `audit` and `log-archive` accounts managed by a central security team, and do they aggregate findings and logs from all other accounts?
3. Is the management account (`110319895932`) used to deploy any application or operational resources, or is it restricted to organisation management only?

### Architecture

4. Do the `cloudox-demo-atlas-dev` and `cloudox-demo-atlas-prod` workloads share any infrastructure (ECR repositories, databases, secrets, KMS keys) across accounts `105769365151` and `122122642149`?
5. What does the single RDS instance host, which workload does it serve, and is it in the production or development account?
6. What is the purpose of the `cloudox-demo-sandbox-scratch` Lambda in the sandbox account (`161388682021`)? Does it process or store any non-public data?

### Security

7. Is GuardDuty centrally managed from the `audit` account, and does it cover all seven accounts in the organisation?
8. Is AWS Config enabled and recording in all accounts? If so, where are Config snapshots and history delivered?
9. What is the purpose of the single IAM user (`AWS::IAM::User`)? Does it have active access keys or console access, and is it subject to MFA enforcement?
10. Which of the five Organisation policies are SCPs, and what controls do they enforce? Are any accounts exempt?

### Observability and Operations

11. Are VPC Flow Logs enabled for any of the 15 VPCs? If so, where are they delivered?
12. Which resources are covered by the single Backup Plan, and is there a tested recovery procedure?
13. What are the 7 CloudWatch Alarms monitoring, and do the 6 SNS topics route alerts to an on-call or incident management system?

---

## Data Sources That Would Improve Confidence

| Gap | Data Source Needed |
|-----|--------------------|
| GuardDuty and Security Hub enablement and coverage | `GetDetector` API response per account; `DescribeHub` per account |
| CloudTrail logging status | `GetTrail` / `GetTrailStatus` (`IsLogging` field) for each of the 12 trails |
| AWS Config recording status | `DescribeConfigurationRecorders` and `DescribeDeliveryChannels` per account |
| IAM user access keys and MFA | `ListAccessKeys`, `GetLoginProfile`, `ListMFADevices` for the detected IAM user |
| RDS instance details | Full `AWS::RDS::DBInstance` resource properties (encryption, Multi-AZ, account, region) |
| Cross-account relationships | Re-run discovery with relationship collection enabled across all accounts |
| Organisation SCP content | `DescribePolicy` for each of the 5 `AWS::Organizations::Policy` resources |
| Backup coverage | `ListBackupSelections` for the detected Backup Plan |
| VPC Flow Log status | `DescribeFlowLogs` scoped to each of the 15 VPCs |
| KMS key classification | `DescribeKey` for each of the 14 keys to distinguish customer-managed from AWS-managed |
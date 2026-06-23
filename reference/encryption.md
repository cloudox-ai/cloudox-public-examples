> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Encryption Reference

Encryption-at-rest status and key relationships for data stores.

_Displaying all **14** row(s) (no rows omitted)._

_Sorted riskiest first: Not encrypted → Unknown → Encrypted._

| Resource | Resource Type | Encryption Status | Key Type | KMS Key | Confidence | Evidence |
|---|---|---|---|---|---|---|
| cloudox-demo-sandbox-scratch | AWS::DynamoDB::Table | Unknown | — | — | Unknown | `cloudox-demo-sandbox-scratch` |
| cloudox-demo-atlas-dev-items | AWS::DynamoDB::Table | Unknown | — | — | Unknown | `cloudox-demo-atlas-dev-items` |
| cloudox-demo-atlas-prod-items | AWS::DynamoDB::Table | Encrypted | Customer-managed (KMS) | arn:aws:kms:eu-central-1:122122642149:key/1ac02cc5-a23f-75da-30c9-99c095a072fa | Verified | `cloudox-demo-atlas-prod-items` |
| cloudox-demo-atlas-prod-pg | AWS::RDS::DBInstance | Encrypted | AWS-managed | — | Verified | `cloudox-demo-atlas-prod-pg` |
| cloudox-cust-sandbox-a8d4 | AWS::S3::Bucket | Encrypted (AES256) | AWS-managed | — | Verified | `cloudox-cust-sandbox-a8d4` |
| cloudox-public-example | AWS::S3::Bucket | Encrypted (AES256) | AWS-managed | — | Verified | `cloudox-public-example` |
| cloudox-demo-access-logs-122980216815-eu-central-1 | AWS::S3::Bucket | Encrypted (AES256) | AWS-managed | — | Verified | `cloudox-demo-access-logs-122980216815-eu-central-1` |
| cloudox-demo-cloudtrail-122980216815-eu-central-1 | AWS::S3::Bucket | Encrypted (aws:kms) | Customer-managed (KMS) | d1b59baf-b18a-b78f-1a08-7254c874d325 | Verified | `cloudox-demo-cloudtrail-122980216815-eu-central-1` |
| cloudox-demo-config-122980216815-eu-central-1 | AWS::S3::Bucket | Encrypted (aws:kms) | Customer-managed (KMS) | d1b59baf-b18a-b78f-1a08-7254c874d325 | Verified | `cloudox-demo-config-122980216815-eu-central-1` |
| cloudox-demo-sandbox-161388682021-eu-central-1 | AWS::S3::Bucket | Encrypted (AES256) | AWS-managed | — | Verified | `cloudox-demo-sandbox-161388682021-eu-central-1` |
| cloudox-demo-sandbox-abandoned-161388682021-eu-central-1 | AWS::S3::Bucket | Encrypted (AES256) | AWS-managed | — | Verified | `cloudox-demo-sandbox-abandoned-161388682021-eu-central-1` |
| cloudox-demo-atlas-dev-app-105769365151-eu-central-1 | AWS::S3::Bucket | Encrypted (AES256) | AWS-managed | — | Verified | `cloudox-demo-atlas-dev-app-105769365151-eu-central-1` |
| cloudox-demo-atlas-prod-app-122122642149-eu-central-1 | AWS::S3::Bucket | Encrypted (aws:kms) | Customer-managed (KMS) | 1ac02cc5-a23f-75da-30c9-99c095a072fa | Verified | `cloudox-demo-atlas-prod-app-122122642149-eu-central-1` |
| cloudox-demo-atlas-prod-dr-snapshots-122122642149-us-east-1 | AWS::S3::Bucket | Encrypted (aws:kms) | Customer-managed (KMS) | c7e5f429-9c4e-a068-d93a-52ebe27c0564 | Verified | `cloudox-demo-atlas-prod-dr-snapshots-122122642149-us-east-1` |

> Encryption field not collected for this resource.

**Coverage notes**

- Rows marked Unknown lack a collected encryption field; absence is not evidence of being unencrypted.

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Tagging Reference

Ownership, environment, and cost-allocation tag coverage.

_Displaying all **20** row(s) (no rows omitted)._

_Sorted most missing important tags first, then resource type._

| Resource Type | Resource | Environment Tag | Owner Tag | Cost Center Tag | Missing Important Tags | Confidence | Evidence |
|---|---|---|---|---|---|---|---|
| AWS::ECS::Service | cloudox-demo-atlas-dev-api | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-demo-atlas-dev-api` |
| AWS::ECS::Service | cloudox-demo-atlas-prod-api | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-demo-atlas-prod-api` |
| AWS::ElasticLoadBalancingV2::LoadBalancer | cloudox-demo-atlas-prod-alb | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-demo-atlas-prod-alb` |
| AWS::Lambda::Function | cloudox-demo-sandbox-scratch | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-demo-sandbox-scratch` |
| AWS::Lambda::Function | cloudox-demo-atlas-dev-api | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-demo-atlas-dev-api` |
| AWS::Lambda::Function | cloudox-demo-atlas-prod-api | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-demo-atlas-prod-api` |
| AWS::S3::Bucket | cloudox-cust-sandbox-a8d4 | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-cust-sandbox-a8d4` |
| AWS::S3::Bucket | cloudox-public-example | — | — | — | Environment, Owner, CostCenter | Verified | `cloudox-public-example` |
| AWS::DynamoDB::Table | cloudox-demo-sandbox-scratch | sandbox-ma | — | CloudoX | Owner | Verified | `cloudox-demo-sandbox-scratch` |
| AWS::DynamoDB::Table | cloudox-demo-atlas-dev-items | dev | — | CloudoX | Owner | Verified | `cloudox-demo-atlas-dev-items` |
| AWS::DynamoDB::Table | cloudox-demo-atlas-prod-items | prod | — | CloudoX | Owner | Verified | `cloudox-demo-atlas-prod-items` |
| AWS::RDS::DBInstance | cloudox-demo-atlas-prod-pg | prod | — | CloudoX | Owner | Verified | `cloudox-demo-atlas-prod-pg` |
| AWS::S3::Bucket | cloudox-demo-access-logs-122980216815-eu-central-1 | logging | — | CloudoX | Owner | Verified | `cloudox-demo-access-logs-122980216815-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-cloudtrail-122980216815-eu-central-1 | logging | — | CloudoX | Owner | Verified | `cloudox-demo-cloudtrail-122980216815-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-config-122980216815-eu-central-1 | logging | — | CloudoX | Owner | Verified | `cloudox-demo-config-122980216815-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-sandbox-161388682021-eu-central-1 | sandbox-ma | — | CloudoX | Owner | Verified | `cloudox-demo-sandbox-161388682021-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-sandbox-abandoned-161388682021-eu-central-1 | sandbox-ma | — | CloudoX | Owner | Verified | `cloudox-demo-sandbox-abandoned-161388682021-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-atlas-dev-app-105769365151-eu-central-1 | dev | — | CloudoX | Owner | Verified | `cloudox-demo-atlas-dev-app-105769365151-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-atlas-prod-app-122122642149-eu-central-1 | prod | — | CloudoX | Owner | Verified | `cloudox-demo-atlas-prod-app-122122642149-eu-central-1` |
| AWS::S3::Bucket | cloudox-demo-atlas-prod-dr-snapshots-122122642149-us-east-1 | prod | — | CloudoX | Owner | Verified | `cloudox-demo-atlas-prod-dr-snapshots-122122642149-us-east-1` |

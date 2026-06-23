> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Tagging Reference

Ownership, environment, and cost-allocation tag coverage. A missing tag is only reported where tag collection is known complete; collector-blind resource types are shown as Unknown (not collected).

_Displaying all **20** row(s) (no rows omitted)._

_Sorted most verified missing tags first, then resource type._

| Resource Type | Resource | Environment Tag | Owner Tag | Cost Center Tag | Missing Important Tags | Confidence | Evidence |
|---|---|---|---|---|---|---|---|
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
| AWS::ECS::Service | cloudox-demo-atlas-dev-api | Unknown | Unknown | Unknown | Unknown (tags not collected) | Unknown | `cloudox-demo-atlas-dev-api` |
| AWS::ECS::Service | cloudox-demo-atlas-prod-api | Unknown | Unknown | Unknown | Unknown (tags not collected) | Unknown | `cloudox-demo-atlas-prod-api` |
| AWS::ElasticLoadBalancingV2::LoadBalancer | cloudox-demo-atlas-prod-alb | Unknown | Unknown | Unknown | Unknown (tags not collected) | Unknown | `cloudox-demo-atlas-prod-alb` |
| AWS::Lambda::Function | cloudox-demo-sandbox-scratch | Unknown | Unknown | Unknown | Unknown (tags not collected) | Unknown | `cloudox-demo-sandbox-scratch` |
| AWS::Lambda::Function | cloudox-demo-atlas-dev-api | Unknown | Unknown | Unknown | Unknown (tags not collected) | Unknown | `cloudox-demo-atlas-dev-api` |
| AWS::Lambda::Function | cloudox-demo-atlas-prod-api | Unknown | Unknown | Unknown | Unknown (tags not collected) | Unknown | `cloudox-demo-atlas-prod-api` |

> Tags are not collected for this resource type, so tag presence/absence is unknown — not a verified gap.

**Coverage notes**

- Tag collection is not available for some resource types (AWS::ECS::Service, AWS::ElasticLoadBalancingV2::LoadBalancer, AWS::Lambda::Function); their rows show Environment/Owner/Cost Center as Unknown (not collected), NOT as verified missing tags. A missing tag is only asserted where tag collection is known to be complete.
- 'Cost Center' aggregates several cost-allocation tag keys (costcenter, cost-center, cost_center, billingcode, project); a value may come from any of them, so the column reflects cost-allocation coverage rather than a literal 'CostCenter' tag.

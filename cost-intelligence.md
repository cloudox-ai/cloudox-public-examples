# Cost Intelligence

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> CloudoX explains cloud cost in **architectural context** — it does not replace AWS Cost Explorer, the Cost & Usage Report, or a dedicated FinOps platform.

## Executive cost summary

AWS Cost Explorer spend data is **unavailable** for this run; the analysis below is derived from the discovered architecture only. See *Limitations* for details.

## Top cost drivers

Architectural patterns that carry charges (from the knowledge graph; counts, not dollar attributions):

| Pattern | Count | Category | Basis |
|---|---|---|---|
| Multi-region resource footprint | 2 | networking | Inferred |
| ECS services | 2 | compute | Observed |
| Lambda functions | 3 | compute | Observed |
| Application/Network Load Balancers | 1 | compute | Observed |
| S3 buckets | 10 | storage | Observed |
| DynamoDB tables | 3 | database | Observed |
| RDS instances | 1 | database | Observed |

## Architectural cost drivers

### Multi-region resource footprint (Inferred)

Count: 2 · Category: networking · Regions: eu-central-1, us-east-1

Resources span multiple regions; inter-region replication and data transfer typically carry per-GB charges. Review whether cross-region traffic is intentional.

### ECS services (Observed)

Count: 2 · Category: compute · Accounts: 2 · Regions: eu-central-1

ECS services on Fargate bill per vCPU/GB-hour per running task.

Examples: `cloudox-demo-atlas-dev-api`, `cloudox-demo-atlas-prod-api`

### Lambda functions (Observed)

Count: 3 · Category: compute · Accounts: 3 · Regions: eu-central-1

Lambda bills per request and per GB-second; cost depends on invocation volume, which CloudoX does not measure.

Examples: `cloudox-demo-atlas-dev-api`, `cloudox-demo-atlas-prod-api`, `cloudox-demo-sandbox-scratch`

### Application/Network Load Balancers (Observed)

Count: 1 · Category: compute · Accounts: 1 · Regions: eu-central-1

ELBv2 load balancers bill per hour plus an LCU/NLCU usage dimension.

Examples: `cloudox-demo-atlas-prod-alb`

### S3 buckets (Observed)

Count: 10 · Category: storage · Accounts: 5 · Regions: eu-central-1, us-east-1

S3 bills per GB-month by storage class plus request and data-transfer charges; stored volume is not captured by CloudoX.

Examples: `cloudox-cust-sandbox-a8d4`, `cloudox-demo-access-logs-122980216815-eu-central-1`, `cloudox-demo-atlas-dev-app-105769365151-eu-central-1`, `cloudox-demo-atlas-prod-app-122122642149-eu-central-1`, `cloudox-demo-atlas-prod-dr-snapshots-122122642149-us-east-1`, `cloudox-demo-cloudtrail-122980216815-eu-central-1`, `cloudox-demo-config-122980216815-eu-central-1`, `cloudox-demo-sandbox-161388682021-eu-central-1`, `cloudox-demo-sandbox-abandoned-161388682021-eu-central-1`, `cloudox-public-example`

### DynamoDB tables (Observed)

Count: 3 · Category: database · Accounts: 3 · Regions: eu-central-1

DynamoDB bills by capacity mode (provisioned or on-demand) plus storage; capacity mode is not captured by CloudoX.

Examples: `cloudox-demo-atlas-dev-items`, `cloudox-demo-atlas-prod-items`, `cloudox-demo-sandbox-scratch`

### RDS instances (Observed)

Count: 1 · Category: database · Accounts: 1 · Regions: eu-central-1

RDS instances bill per instance-hour by class. Top classes: db.t4g.micro x1.

Examples: `cloudox-demo-atlas-prod-pg`

## Optimization opportunities

No deterministic optimization candidates were detected from the current knowledge graph. Utilization-based recommendations require metrics CloudoX does not collect in this version.

## Assumptions

- Spend figures use the AWS Cost Explorer UnblendedCost metric for complete billing periods only; the current (partial) month is excluded.
- Architectural cost drivers describe resource patterns known to carry charges; they are not dollar attributions. Exact cost attribution comes from AWS Cost Explorer / CUR.
- Spend data is unavailable for this run; the analysis below is derived from the discovered architecture only.

## Limitations and missing permissions

- Cost Explorer collection is disabled (CLOUDOX_COST__ENABLED=false / --no-collect-costs); spend figures are unavailable. The analysis below is derived from the discovered architecture only.
- CloudoX does not collect CloudWatch utilization metrics in this version; idle, underutilized, or right-sizing recommendations based on actual usage are not available.
- RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by the current collectors, so cost drivers for them are not detected.

## Suggested next steps

- Grant `ce:GetCostAndUsage` and `ce:GetDimensionValues` and re-run to populate spend figures.
- Correlate the largest spend services with the architectural cost drivers to confirm where cost concentrates.
- Re-run after the next discovery to compare cost intelligence run-over-run as Environment Evolution support matures.

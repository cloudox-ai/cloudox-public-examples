# Architecture Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

Five workloads were detected across three accounts in `eu-central-1`. All are inferred by CloudoX from resource naming and relationships; none carry explicit workload or tier tags (see [08-assumptions-and-unknowns.md](08-assumptions-and-unknowns.md)).

---

## Detected Workloads

| Workload name | Anchor resource | Account | Confidence | Internet-facing |
|---|---|---|---|---|
| cloudox-demo-atlas-prod | `cloudox-demo-atlas-prod` (ECS Cluster) | `122122642149` (workload-prod) | Medium | Yes — ALB `cloudox-demo-atlas-prod-alb` confirmed by finding + digest entry point |
| cloudox-demo-atlas-prod-api | `cloudox-demo-atlas-prod-api` (Lambda) | `122122642149` (workload-prod) | Medium | Yes — API Gateway `xdmn5ldmif` confirmed as digest entry point |
| cloudox-demo-atlas-dev | `cloudox-demo-atlas-dev` (ECS Cluster) | `105769365151` (workload-dev) | Medium | No — no internet-facing entry point detected |
| cloudox-demo-atlas-dev-api | `cloudox-demo-atlas-dev-api` (Lambda) | `105769365151` (workload-dev) | Medium | No — API Gateway `gfwaiva01f` is a digest entry point but no internet-facing finding was raised against it |
| cloudox-demo-sandbox-scratch | `cloudox-demo-sandbox-scratch` (Lambda) | `161388682021` (sandbox-ma) | Low | No |

> **Note on workload overlap:** `cloudox-demo-atlas-prod-api` (Lambda) and the ECS service `cloudox-demo-atlas-prod-api` share a name and both appear in account `122122642149`. The digest treats them as separate workloads. Whether they represent the same logical service on different compute runtimes should be validated with the customer.

---

## Application Layer

**Production (`122122642149`):** The primary request path observed in the relationships data runs:

```
internet
  → cloudox-demo-atlas-prod-alb  (ALB, internet-facing, sg-06f2b4190bf01d261)
    → listener 9b700d6b85e65ced
      → target group cloudox-demo-atlas-prod-tg
        → ECS service cloudox-demo-atlas-prod-api (task def cloudox-demo-atlas-prod-api:2)
          → ECS cluster cloudox-demo-atlas-prod
```

A parallel path exists via API Gateway `xdmn5ldmif` → Lambda `cloudox-demo-atlas-prod-api`. Whether these two paths serve the same or different consumers is not determinable from the discovery data.

**Development (`105769365151`):** API Gateway `gfwaiva01f` routes to ECS service `cloudox-demo-atlas-dev-api` (task def `cloudox-demo-atlas-dev-api:1`) within cluster `cloudox-demo-atlas-dev`. A Lambda function named `cloudox-demo-atlas-dev-api` is also present; its relationship to the ECS service is not established in the relationships block.

**Sandbox (`161388682021`):** A single Lambda function `cloudox-demo-sandbox-scratch` with a DynamoDB table of the same name. The dependency `cloudox-demo-sandbox-scratch → AROAAAAADE03UVM8VCPAN` (IAM role) is present in the digest. No other resources are connected to this workload.

---

## Network Layer

- 15 VPCs and 63 subnets are present across the 7 accounts (graph_stats). No cross-account VPC peering or Transit Gateway was detected in the provided data.
- The production workload ALB (`cloudox-demo-atlas-prod-alb`) is placed in `vpc-0aaa6d4f2f981e945`, subnets `subnet-013a24d318ed6f3d0` (`cloudox-demo-atlas-prod-public-a`) and `subnet-016c22941a019a137` (`cloudox-demo-atlas-prod-public-b`). Both subnets have internet-facing exposure findings (finding IDs `exposure:AWS::EC2::Subnet:subnet-013a24d318ed6f3d0`, `exposure:AWS::EC2::Subnet:subnet-016c22941a019a137`).
- The ECS service `cloudox-demo-atlas-prod-api` is placed in the same two subnets and secured by `sg-0457a284b8b9ead6c`, which has an `allows_from` relationship to `sg-06f2b4190bf01d261` (the ALB security group). This suggests traffic is intended to flow ALB → ECS, though the actual rule content was not inspected.
- The dev ECS service `cloudox-demo-atlas-dev-api` is in subnets `subnet-0f64d71c952a7898a` and `subnet-065f522206524ab12` (`cloudox-demo-atlas-dev-public-a`), secured by `sg-0d6a48061beb72eae`. Subnet `subnet-065f522206524ab12` has an internet-facing exposure finding (`exposure:AWS::EC2::Subnet:subnet-065f522206524ab12`).
- 4 VPC Endpoints are present (`vpce-0f376e51c370a0f95`, `vpce-0e75224f50dddfb8d`, `vpce-02ddbd8d63a6ab447`, `vpce-0232f3af5b06cd3cf`). Route tables `rtb-009215bffcfd9f9a0` and `rtb-0fb49391955972271` route to these endpoints. The accounts and VPCs they belong to are not resolved in the filtered data.

---

## Data Layer

| Resource | Type | Account | Connected workload |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | RDS DB Instance | `122122642149` | Secured by `sg-0831f3eae117601d2`; `sg-0831f3eae117601d2` allows from `sg-0457a284b8b9ead6c` (ECS SG) — suggests prod ECS → RDS path |
| `cloudox-demo-sandbox-scratch` | DynamoDB Table | `161388682021` | Lambda `cloudox-demo-sandbox-scratch` (via `uses_iam_role` dependency) |
| Two additional DynamoDB tables | DynamoDB Table | Not resolved in filtered data | No relationship detected to named workloads |

The RDS instance `cloudox-demo-atlas-prod-pg` depends on parameter group `stackset-cloudox-demo-workload-prod-3fa47485-72dc-52dd-4672-e5243b802465-rdsparametergroup-jegbgv8kifvi`, indicating it was provisioned via a CloudFormation StackSet.

No backup configuration was directly linked to the RDS instance in the relationships block. A `AWS::Backup::BackupPlan` and `AWS::Backup::BackupVault` are present in the graph (graph_stats), but no relationship to `cloudox-demo-atlas-prod-pg` was detected.

---

## Integration / Messaging Layer

6 SNS topics and 1 SQS queue are present (digest). No relationships connecting these to the named workloads were detected in the provided data. Their purpose and ownership are unknown.

---

## DNS

A Route 53 hosted zone `Z09280431A90P7LBIX0FB` named `atlas.internal.demo` is present in account `122122642149` and is listed as a digest entry point. The zone name suggests internal DNS for the Atlas workload. No records or resolver rule associations were resolved in the filtered data.

12 Route 53 Resolver rules and 15 resolver rule associations are present in the graph (graph_stats). Their targets and associated VPCs are not resolved in the filtered data.

---

## Supporting / Unassociated Resources

The following resource classes are present in the graph but were not connected to any named workload in the relationships or digest:

- **ECR repositories** (3): Likely used by ECS task definitions. No `uses_image_from` or equivalent relationship was detected. *(Inferred — basis: ECS workloads typically pull from ECR; not confirmed by a relationship.)*
- **SNS topics / SQS queue** (6 + 1): No workload relationship detected.
- **S3 buckets** (10): No workload relationship detected in the filtered data.
- **Secrets Manager secrets** (4): No workload relationship detected.
- **KMS keys** (14) and aliases (153): No workload relationship detected.

---

## Inferred Architecture — Confidence Table

| Inference | Basis | Confidence | Validate by |
|---|---|---|---|
| Dev and prod are environment-separated by account | Account aliases `workload-dev` / `workload-prod`; OU path `/Workloads` | Medium | Confirm with customer that no cross-account prod traffic flows |
| ECS service `cloudox-demo-atlas-prod-api` receives traffic from ALB via target group | `cloudox-demo-atlas-prod-api` → `cloudox-demo-atlas-prod-tg` (routes_to); `9b700d6b85e65ced` → `cloudox-demo-atlas-prod-tg` (routes_to) | Medium | Confirm target group registration and health check status |
| RDS `cloudox-demo-atlas-prod-pg` is the prod database for the Atlas ECS service | `sg-0831f3eae117601d2` allows from `sg-0457a284b8b9ead6c` (ECS SG) | Medium | Confirm application connection string / Secrets Manager secret binding |
| Lambda `cloudox-demo-atlas-prod-api` and ECS service `cloudox-demo-atlas-prod-api` serve different purposes | Separate workload entries in digest; different compute types | Low | Ask customer whether Lambda is a migration target, a sidecar function, or a legacy path |
| Dev ECS service is not internet-facing | No internet-facing finding raised against `gfwaiva01f`; no ALB in dev workload | Medium | Confirm API Gateway authorisation and whether `gfwaiva01f` is publicly accessible |

---

## Unknowns / Gaps

- 754 of 800 resources carry no `Environment`, `Stage`, or `Tier` tag (digest unknowns). Workload groupings above rely entirely on name-based inference.
- The two unresolved DynamoDB tables have no detected workload relationship. Their owners and purpose are unknown.
- SNS topics, SQS queue, S3 buckets, and Secrets Manager secrets have no detected workload relationships. Whether they are shared platform resources or workload-specific is unknown.
- VPC Endpoint targets (service names) are not resolved in the filtered data. Whether they provide private access to S3, DynamoDB, or other services cannot be determined.
- No relationship between the Route 53 hosted zone `atlas.internal.demo` and specific VPCs was detected.
- The `platform-prod` account (`150982215529`) has no workloads detected. Its role in the architecture is unknown.

---

## Follow-up Questions

1. Do the Lambda functions `cloudox-demo-atlas-prod-api` and `cloudox-demo-atlas-dev-api` serve the same logical API as the ECS services of the same name, or are they separate entry points?
2. What services do the four VPC Endpoints (`vpce-0f376e51c370a0f95`, `vpce-0e75224f50dddfb8d`, `vpce-02ddbd8d63a6ab447`, `vpce-0232f3af5b06cd3cf`) target?
3. Is the `platform-prod` account (`150982215529`) intentionally empty, or does it host shared services not yet visible in the discovery data?
4. Are the SNS topics and SQS queue used for async communication between the Atlas workload and external consumers, or are they infrastructure-level notifications?
5. Which Secrets Manager secrets are bound to `cloudox-demo-atlas-prod-pg`, and is the RDS instance covered by the detected Backup plan?
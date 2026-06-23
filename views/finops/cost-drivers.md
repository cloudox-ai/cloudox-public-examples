# FinOps View — Cost Drivers

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Cost Drivers

> **Confidence: Likely** — AWS Cost Explorer data collection is disabled for this run. All cost driver analysis is derived from discovered architecture. Dollar figures are not available; the patterns below describe resources and design choices known to carry charges. Validate against Cost Explorer or CUR before acting.

The environment spans **807 resources across 7 accounts**, with **4 significant workloads** and 1 platform system identified. The primary cost-bearing activity is concentrated in two workload accounts — **Workload Dev Account** (`105769365151`) and **Workload Prod Account** (`122122642149`) — both running in `eu-central-1`. A **Sandbox Ma Account** (`161388682021`), **Audit Account** (`110019496666`), **Log Archive Account** (`122980216815`), and **Platform Account** (`150982215529`) carry supporting infrastructure that also generates spend.

### Architectural Cost Drivers

Three architectural patterns stand out as the primary structural cost drivers:

**1. Multi-account, multi-environment API workloads**

Two parallel API workloads run in separate accounts — **Cloudox Demo Atlas Dev API** (`cloudox-demo-atlas-dev-api`) in the Workload Dev Account and **Cloudox Demo Atlas Prod API** (`cloudox-demo-atlas-prod-api`) in the Workload Prod Account, both in `eu-central-1`. Each workload has its own API Gateway endpoint (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) exposed to the internet. Running full parallel stacks for dev and prod is an intentional isolation choice, but it means fixed and per-request costs are duplicated across both environments. This is the expected cost of environment parity — worth confirming that the dev environment is not over-provisioned relative to its actual usage.

**2. Internet-facing infrastructure across multiple accounts and regions**

Four Internet Gateways are present across three accounts and two regions:

| Internet Gateway | Account | Region |
|---|---|---|
| `igw-00ed21b9a0e6596a8` | Audit Account (`110019496666`) | us-east-1 |
| `igw-0d14f1dd4e54d5906` | Audit Account (`110019496666`) | eu-central-1 |
| `igw-0567575921f471548` | Workload Dev Account (`105769365151`) | us-east-1 |
| `igw-0cff0d66b4fd90803` | Log Archive Account (`122980216815`) | us-east-1 |

Internet Gateways themselves carry no hourly charge, but their presence indicates VPCs with public subnets, NAT Gateways, or data transfer paths that do. The presence of gateways in `us-east-1` for accounts whose workloads appear to be in `eu-central-1` (Workload Dev, Audit) is worth validating — unused or orphaned VPC infrastructure in a secondary region is a common source of quiet, ongoing spend. **Requires validation.**

**3. DynamoDB tables across three accounts**

Three DynamoDB tables are identified:

| Table | Account | Environment |
|---|---|---|
| `cloudox-demo-atlas-dev-items` (`arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`) | Workload Dev | Dev |
| `cloudox-demo-atlas-prod-items` (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`) | Workload Prod | Prod |
| `cloudox-demo-sandbox-scratch` (`arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`) | Sandbox Ma | Sandbox |

DynamoDB cost depends heavily on capacity mode (on-demand vs. provisioned) and actual read/write throughput — neither is captured in this run. The sandbox table (`cloudox-demo-sandbox-scratch`) in the **Cloudox Demo Sandbox Scratch** workload (`cloudox-demo-sandbox-scratch`) is a candidate for review: sandbox resources left running at provisioned capacity can accumulate unnecessary spend. **Requires validation.**

### Service & Resource Drivers

**API Gateway** is a direct cost driver for both the dev and prod workloads, with charges accruing per API call and per GB of data transfer out to the internet. Both endpoints are publicly accessible (`internet`-facing), meaning all external traffic contributes to data transfer costs.

**DynamoDB** drives cost through storage, read/write capacity units, and (if enabled) features such as point-in-time recovery or global tables. Without capacity mode data, it is not possible to determine whether the tables are optimally configured — this is a gap to close in Cost Explorer or via a targeted collector run.

**VPC / Networking** costs (NAT Gateway hours and data processing, inter-AZ transfer, data transfer out) are structurally implied by the four Internet Gateways and the multi-account, multi-region VPC footprint. These charges are often underestimated and can be material, particularly if the `us-east-1` gateways in the Audit and Log Archive accounts are associated with active NAT Gateways. **Requires validation.**

**Supporting account infrastructure** — the Management Account (`110319895932`), Audit Account (`110019496666`), Log Archive Account (`122980216815`), and Platform Account (`150982215529`) — carries baseline spend for AWS Control Tower / Organizations, CloudTrail, Config, and log storage. This is expected for a well-governed multi-account structure, but the volume of log storage and Config rule evaluations should be reviewed periodically.

> **Key gaps for FinOps action:** Enable Cost Explorer collection (`CLOUDOX_COST__ENABLED=true`) to attach dollar figures to these architectural patterns. Priority validation targets are: (1) the purpose and activity of `us-east-1` VPC infrastructure in the Audit and Log Archive accounts, (2) DynamoDB capacity mode and utilization for all three tables, and (3) whether the dev API workload is sized proportionally to actual load.

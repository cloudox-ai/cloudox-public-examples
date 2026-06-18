# FinOps View — Cost Drivers

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Cost Drivers

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All cost driver analysis is derived from discovered architecture only. Dollar attributions cannot be made here; use AWS Cost Explorer or CUR for exact figures.

The most important thing to know before reading this section: **no billing data was collected**. What follows describes the architectural patterns that are known to carry charges across 807 resources, 7 accounts, and multiple workloads — giving you a map of where to look in Cost Explorer, not a spend report.

### Architectural Cost Drivers

Seven architectural cost drivers were identified from the discovered environment. These are structural patterns — not line-item costs — that typically generate ongoing charges and warrant direct investigation in your billing data.

**API Gateway exposure (internet-facing endpoints)**

Two public API Gateway endpoints are active in `eu-central-1`:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` — associated with the **Cloudox Demo Atlas Dev API** workload (Workload Dev Account `105769365151`)
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` — associated with the **Cloudox Demo Atlas Prod API** workload (Workload Prod Account `122122642149`)

API Gateway charges accrue per API call and per GB of data transferred. Both endpoints are internet-facing, meaning external traffic volumes directly drive cost. Prod and Dev endpoints running in parallel means both are incurring baseline charges regardless of traffic levels.

**DynamoDB tables across three accounts**

Three DynamoDB tables are present across different account tiers:

| Table | Account | Workload |
|---|---|---|
| `cloudox-demo-atlas-dev-items` | Workload Dev (`105769365151`) | Cloudox Demo Atlas Dev |
| `cloudox-demo-atlas-prod-items` | Workload Prod (`122122642149`) | Cloudox Demo Atlas Prod API |
| `cloudox-demo-sandbox-scratch` | Sandbox Ma (`161388682021`) | Cloudox Demo Sandbox Scratch |

DynamoDB cost is driven by capacity mode (on-demand vs. provisioned), read/write request volumes, and storage. **The capacity mode for these tables is not captured by the current collectors** — this is a material unknown for cost estimation. The sandbox table (`cloudox-demo-sandbox-scratch`) is a candidate for validation: scratch/sandbox resources can accumulate cost without active use.

**Internet Gateways across multiple accounts and regions**

Four Internet Gateways are present across two regions and three accounts:

| IGW | Account | Region |
|---|---|---|
| `igw-00ed21b9a0e6596a8` | Audit Account (`110019496666`) | us-east-1 |
| `igw-0d14f1dd4e54d5906` | Audit Account (`110019496666`) | eu-central-1 |
| `igw-0567575921f471548` | Workload Dev (`105769365151`) | us-east-1 |
| `igw-0cff0d66b4fd90803` | Log Archive Account (`122980216815`) | us-east-1 |

Internet Gateways themselves carry no hourly charge, but they enable data transfer paths that do. The presence of IGWs in the Audit Account and Log Archive Account — and a Dev account IGW in `us-east-1` while the workload appears to operate in `eu-central-1` — are worth validating. Cross-region data transfer and unexpected egress from governance/logging accounts can be silent cost contributors.

**Multi-account, multi-region footprint**

The environment spans 7 accounts (Management, Workload Dev, Workload Prod, Sandbox, Audit, Log Archive, Platform) and at least two regions (`eu-central-1` and `us-east-1`). This structure is architecturally sound for governance, but it means:
- Per-account fixed costs (e.g., AWS Config, CloudTrail, GuardDuty) multiply across accounts
- Cross-region data transfer charges apply wherever resources communicate across regions
- Cost visibility requires consolidated billing and account-level tagging discipline

**Tagging gap limits cost attribution**

761 of 807 resources (94%) have no Environment / Stage / Tier tag and rely on inference for classification. This is not a cost driver itself, but it severely limits the ability to attribute spend to workloads, environments, or teams in Cost Explorer. Any showback or chargeback model will have significant blind spots until tagging coverage improves.

### Service & Resource Drivers

No spend data is available to rank services by cost. The following service-level observations are derived from architecture discovery alone.

**Services with known per-request or per-unit cost models present in this environment:**
- **Amazon API Gateway** — two internet-facing endpoints (dev + prod); charges per million API calls and per GB data out
- **Amazon DynamoDB** — three tables across dev, prod, and sandbox; charges depend on capacity mode (unknown) and request volume
- **Data transfer** — four Internet Gateways across two regions create potential egress paths; cross-region transfer between `us-east-1` and `eu-central-1` resources would carry inter-region charges

**Optimization candidates requiring validation:**

| Candidate | Rationale | Action Required |
|---|---|---|
| `cloudox-demo-sandbox-scratch` DynamoDB table (Sandbox `161388682021`) | Sandbox/scratch resource; may be idle or low-use | Validate activity in CloudWatch; confirm if still needed |
| IGW `igw-0567575921f471548` in `us-east-1` (Workload Dev `105769365151`) | Dev workload appears to be `eu-central-1`-primary; `us-east-1` IGW may be unused | Confirm whether any resources route through this IGW; check for associated NAT or data transfer charges |
| IGW `igw-0d14f1dd4e54d5906` and `igw-00ed21b9a0e6596a8` in Audit Account (`110019496666`) | Governance accounts with internet gateways are unusual; egress from audit/log infrastructure can be unexpected | Validate whether these IGWs carry active traffic; review data transfer costs for this account |
| Dev API endpoint running in parallel with Prod | Parallel API Gateway stages incur independent call and data transfer charges | Confirm Dev endpoint traffic levels; assess whether Dev can be throttled or decommissioned between active development cycles |

> **Note:** CloudWatch utilization metrics are not collected in this version. Idle or underutilized resource identification (e.g., right-sizing, unused capacity) requires separate analysis using CloudWatch or AWS Compute Optimizer.

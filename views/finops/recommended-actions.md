# FinOps View — Recommended Actions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Recommended Actions

### Recommended Cost Actions

**Spend data is unavailable for this run.** Cost Explorer collection was disabled at discovery time (`CLOUDOX_COST__ENABLED=false`), so no dollar figures, trends, or per-service breakdowns can be presented. The actions below are derived entirely from the discovered architecture — treat them as *validation candidates*, not confirmed savings.

> **Confidence: Likely** — Architectural patterns are observed; their cost impact requires confirmation in AWS Cost Explorer or CUR once collection is enabled.

---

#### Enable Cost Data Collection — Highest Priority

Before any other action, enable Cost Explorer collection in CloudoX. Without it, none of the 7 detected architectural cost drivers can be dollar-quantified, and no optimization can be sized or prioritised. This is a prerequisite for all downstream FinOps work.

---

#### Validate Internet Gateway Sprawl

Four internet gateways are present across multiple accounts and regions:

| Resource | Account | Region |
|---|---|---|
| `igw-00ed21b9a0e6596a8` | 110019496666 | us-east-1 |
| `igw-0d14f1dd4e54d5906` | 110019496666 | eu-central-1 |
| `igw-0567575921f471548` | 105769365151 | us-east-1 |
| `igw-0cff0d66b4fd90803` | 122980216815 | us-east-1 |

Internet gateways themselves carry no hourly charge, but they are a proxy for data-transfer cost: traffic routed through them incurs AWS data-transfer egress fees. **Validate** whether all four gateways are actively routing production traffic, or whether any serve idle or decommissioned VPCs. Unused gateways attached to live VPCs still expose egress paths that may be generating unintended transfer charges.

---

#### Review DynamoDB Tables Across Environments

Three DynamoDB tables are present across three separate accounts:

| Table ARN | Account | Environment (inferred) |
|---|---|---|
| `…:table/cloudox-demo-atlas-dev-items` | 105769365151 | Dev |
| `…:table/cloudox-demo-atlas-prod-items` | 122122642149 | Prod |
| `…:table/cloudox-demo-sandbox-scratch` | 161388682021 | Sandbox |

DynamoDB cost is driven by capacity mode (on-demand vs. provisioned), read/write throughput, and storage. **Note:** DynamoDB capacity mode is not captured by the current collectors, so it is unknown whether these tables are on-demand or provisioned. **Validate** the following:
- Is the sandbox `scratch` table actively used, or can it be deleted or paused?
- Are dev and prod tables sized appropriately for their actual traffic?
- Confirm capacity mode in the AWS console; on-demand tables with low, spiky traffic may be cheaper on provisioned, and vice versa.

---

#### Confirm API Gateway Endpoints Are Intentionally Public

Two public API Gateway endpoints are exposed to the internet:

- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`

API Gateway charges per API call and per GB of data transfer. **Validate** that both endpoints are actively used and that request volumes are within expected ranges. Unexpected public exposure can lead to unbudgeted call volume (e.g., from scanners or misconfigured clients).

---

#### Address Tagging Gaps Before the Next Billing Review

761 of 807 resources (94%) carry no `Environment`, `Stage`, or `Tier` tag and rely on inference for classification. Without tags, cost allocation reports in Cost Explorer cannot reliably attribute spend to teams, environments, or workloads. **Recommended action:** define a mandatory tagging policy for at minimum `Environment` and `Owner`, and remediate existing untagged resources. This is a prerequisite for meaningful showback or chargeback.

---

#### What Cannot Be Assessed Yet

The following optimization categories require either cost data or utilization metrics that are not currently available:

- **Right-sizing / idle resources** — CloudWatch utilization metrics are not collected in this version; CPU, memory, and network idle analysis is not possible.
- **RDS cost drivers** — Read replicas, provisioned IOPS, and storage class are not captured by current collectors.
- **S3 storage class optimisation** — S3 storage classes are not captured; Intelligent-Tiering or lifecycle policy recommendations cannot be made.
- **Direct Connect** — Not captured; any associated port-hour or data charges are invisible to this analysis.

Once Cost Explorer collection is enabled and a full billing period is collected, re-run this view to surface dollar-quantified drivers and ranked optimization candidates.

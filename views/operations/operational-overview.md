# Operations View — Operational Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Operational Overview

The environment spans **807 resources across 7 accounts**, with active workloads in Development, Production, and Sandbox environments. Four significant workloads are inferred alongside one system-level workload; one additional workload has been demoted to helper/governance status. Internet-facing API Gateway endpoints and DynamoDB tables are present across multiple accounts, making blast-radius awareness and tagging discipline operationally critical.

### What's Running

The following accounts and environments make up the operational footprint:

| Account | Friendly Name | Environment(s) |
|---|---|---|
| `110319895932` | Management Account | — |
| `105769365151` | Workload Dev Account | Development Environment |
| `122122642149` | Workload Prod Account | Production Environment |
| `161388682021` | Sandbox Ma Account | Sandbox Environment |
| `110019496666` | Audit Account | — |
| `150982215529` | Platform Account | Production Environment |
| `122980216815` | Log Archive Account | Unknown (unclassified) |

Two public API Gateway endpoints are observed in `eu-central-1`:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` — hosted in Workload Dev Account (`105769365151`)
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` — also in `eu-central-1`

Both endpoints are internet-accessible (`internet`), meaning they are reachable without VPN or private connectivity. Any misconfiguration in authorisation or throttling on these endpoints is directly customer-impacting.

Three DynamoDB tables are identified as key data stores:
- `cloudox-demo-atlas-dev-items` — Workload Dev Account (`105769365151`), `eu-central-1`
- `cloudox-demo-atlas-prod-items` — Workload Prod Account (`122122642149`), `eu-central-1`
- `cloudox-demo-sandbox-scratch` — Sandbox Ma Account (`161388682021`), `eu-central-1`

Four internet gateways are present across accounts and regions, indicating VPCs with direct internet egress/ingress paths:
- `igw-00ed21b9a0e6596a8` — Audit Account (`110019496666`), `us-east-1`
- `igw-0d14f1dd4e54d5906` — Audit Account (`110019496666`), `eu-central-1`
- `igw-0567575921f471548` — Workload Dev Account (`105769365151`), `us-east-1`
- `igw-0cff0d66b4fd90803` — Log Archive Account (`122980216815`), `us-east-1`

The presence of internet gateways in the Audit Account (`110019496666`) and Log Archive Account (`122980216815`) warrants scrutiny — these accounts typically do not require direct internet paths and their IGWs represent an unexpected attack surface.

### Operational Footprint

**Tagging and classification gap — high operational risk.** 761 of 807 resources (approximately 94%) carry no `Environment`, `Stage`, or `Tier` tag. Classification of these resources relies entirely on inference. This directly impairs:
- Incident scoping (which resources belong to which environment/workload?)
- Change management blast-radius assessment
- Cost attribution and chargeback
- Automated runbook targeting

This is the single highest-priority operational hygiene issue in the environment.

**Collector coverage gaps.** Two meta-collectors were unavailable during discovery:
- **Resource Explorer** was disabled or unavailable — AWS-visible resource breadth could not be cross-checked. There may be resource types or regions not captured in this inventory.
- **Cloud Control** was disabled — long-tail and newer resource types are limited to typed collector coverage only.

Operators should treat the 807-resource count as a lower bound. Resources in uncovered types or regions will not appear in this view.

**Multi-region presence.** Internet gateways are observed in both `eu-central-1` and `us-east-1`. The primary workload DynamoDB tables are in `eu-central-1`. The `us-east-1` IGWs in the Dev and Log Archive accounts suggest either legacy infrastructure, cross-region replication paths, or unmanaged sprawl — the package does not distinguish between these.

**Log Archive Account environment unknown.** The Log Archive Account (`122980216815`) has no confirmed environment classification (`122980216815-unknown`). Given its role in centralised logging, any operational disruption here could silently degrade observability across all other accounts. Its internet gateway (`igw-0cff0d66b4fd90803` in `us-east-1`) should be reviewed for necessity.

> **Coverage note:** Observability tooling (alarms, dashboards, log groups), backup configurations, and detailed networking topology are covered in dedicated sections of this view. The gaps noted above (collector coverage, tagging) affect those sections as well.

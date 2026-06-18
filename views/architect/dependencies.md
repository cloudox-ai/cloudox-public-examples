# Architect View — Dependencies

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Dependencies

The most actionable finding in this section is a verified single-AZ database dependency sitting in the critical path of a production workload — a resilience gap that warrants immediate architectural review. Beyond that, the dependency graph spans multiple workloads across at least three accounts in eu-central-1, with internet-facing access paths observed across several VPCs.

*Overall confidence: Likely — derived from graph evidence; some classification relies on inference where tagging is absent.*

### Workload Dependencies

Five significant workloads are identified across the environment, anchored in eu-central-1:

| Workload | Account | Region | Confidence |
|---|---|---|---|
| Cloudox Demo Atlas Prod API (`cloudox-demo-atlas-prod-api`) | 122122642149 | eu-central-1 | Likely |
| Cloudox (`cloudox`) | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Dev (`cloudox-demo-atlas-dev`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API (`cloudox-demo-atlas-dev-api`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch (`cloudox-demo-sandbox-scratch`) | 161388682021 | eu-central-1 | Assumed |

Two API Gateway endpoints are observed as internet-facing entry points: `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`. These are consistent with the Atlas Dev and Prod API workloads, though the exact mapping between endpoints and workloads is not explicitly confirmed in the available evidence.

Internet gateways are present across multiple accounts and regions (`igw-00ed21b9a0e6596a8` and `igw-0d14f1dd4e54d5906` in account 110019496666 across us-east-1 and eu-central-1; `igw-0567575921f471548` in account 105769365151 us-east-1; `igw-0cff0d66b4fd90803` in account 122980216815 us-east-1), indicating that internet-facing exposure is not confined to a single account or region. The summary records 25 internet-facing access paths across 15 VPCs — a surface area worth mapping against intended public exposure.

### Data Dependencies

Three DynamoDB tables are identified as data-tier dependencies:

| Table | Account | Workload Association |
|---|---|---|
| `cloudox-demo-atlas-dev-items` | 105769365151 | Atlas Dev workload |
| `cloudox-demo-atlas-prod-items` | 122122642149 | Atlas Prod workload |
| `cloudox-demo-sandbox-scratch` | 161388682021 | Sandbox Scratch workload |

The more critical data dependency is the RDS DBInstance **cloudox-demo-atlas-prod-pg**, which is not Multi-AZ. The **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account 122122642149, eu-central-1) has a verified dependency on this instance. A single availability zone failure would take the production data tier offline. This is the highest-priority architectural concern in this section (`dependency_concern:architecture:cloudox-demo-atlas-prod-pg`, priority 3, significance 0.57).

> **Recommended action:** Confirm whether Multi-AZ is enabled and automated backups are configured for `cloudox-demo-atlas-prod-pg`. Review how the Cloudox Demo Atlas Prod API handles a data-tier failure — connection retry logic, circuit-breaking, and graceful degradation should all be validated.

Note: the account ID and region for `cloudox-demo-atlas-prod-pg` itself are not confirmed in the available evidence (confidence: Unknown on the resource entity).

### Coupling Risks

The primary coupling risk is the tight, unguarded dependency between the production API workload and a non-resilient relational datastore:

- **Single-AZ RDS in a production critical path** — `cloudox-demo-atlas-prod-pg` is not Multi-AZ. There is no evidence of a standby, read replica, or failover mechanism in the package. A zone failure is a full data-tier outage for the Cloudox Demo Atlas Prod API.
- **Multi-account, multi-region internet gateway spread** — Internet gateways appear in accounts 110019496666, 105769365151, and 122980216815 across both eu-central-1 and us-east-1. Without explicit evidence of centralised egress or ingress controls, this distribution increases the blast radius of any misconfiguration.
- **Tagging gaps limit dependency tracing** — 761 resources have no Environment/Stage/Tier tag and rely on inference for classification. This means some dependency paths may be misattributed or invisible to the graph. Architects should treat any inferred workload boundary (particularly `cloudox-demo-sandbox-scratch`, confidence: Assumed) as unconfirmed until validated against actual resource tags or IaC definitions.
- **Dev/Prod account separation** — Atlas Dev (account 105769365151) and Atlas Prod (account 122122642149) are in separate accounts, which is a positive isolation pattern. However, the presence of a shared `cloudox` workload (Verified, account 122122642149) alongside the Prod API in the same account warrants a review of whether blast-radius isolation between those two workloads is sufficient.

# Architect View — Dependencies

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Dependencies

The most pressing dependency concern in this environment is a production workload's reliance on a single-AZ database instance — a resilience gap that sits directly in the critical path. Architects should treat this as the primary structural risk to address before any scaling or modernisation work.

### Workload Dependencies

Five workloads are identified across three accounts and one region (eu-central-1), with varying confidence levels:

| Workload | Account | Region | Confidence |
|---|---|---|---|
| Cloudox Demo Atlas Prod API (`cloudox-demo-atlas-prod-api`) | 122122642149 | eu-central-1 | Likely |
| Cloudox (`cloudox`) | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Dev (`cloudox-demo-atlas-dev`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API (`cloudox-demo-atlas-dev-api`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch (`cloudox-demo-sandbox-scratch`) | 161388682021 | eu-central-1 | Assumed |

The environment summary indicates 9 internet-facing access paths, 15 VPCs, 63 subnets, and 1 load balancer. Internet-facing paths are evidenced across multiple accounts and regions (including `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`, and internet gateways in eu-central-1 and us-east-1). The distribution of internet gateways across accounts (`igw-00ed21b9a0e6596a8`, `igw-0d14f1dd4e54d5906`, `igw-0567575921f471548`, `igw-0cff0d66b4fd90803`) suggests multiple independently internet-connected VPCs rather than a centralised egress model, though cross-account network topology details are not fully resolved in this section.

> **Note:** 761 resources have no Environment / Stage / Tier tag and rely on inference for classification. Workload boundaries and dependency mapping for those resources carry additional uncertainty.

### Data Dependencies

Three DynamoDB tables are observed as data-tier dependencies across the workloads:

| Table | Account | Workload Context |
|---|---|---|
| `cloudox-demo-atlas-dev-items` | 105769365151 | Atlas Dev |
| `cloudox-demo-atlas-prod-items` | 122122642149 | Atlas Prod |
| `cloudox-demo-sandbox-scratch` | 161388682021 | Sandbox |

The critical data dependency is the RDS DB instance **cloudox-demo-atlas-prod-pg** (`cloudox-demo-atlas-prod-pg`), on which the **Cloudox Demo Atlas Prod API** (`cloudox-demo-atlas-prod-api`, account 122122642149, eu-central-1) depends. This instance is not Multi-AZ. A single availability zone failure would take the production API's entire data tier offline. This concern is rated **Verified** confidence and priority 3.

> **Recommended action:** Confirm whether Multi-AZ is enabled and whether automated backups are in place for `cloudox-demo-atlas-prod-pg`. Review how Cloudox Demo Atlas Prod API handles a data-tier failure — specifically whether it degrades gracefully or fails hard.

### Coupling Risks

The primary coupling risk is the tight, single-path dependency between **Cloudox Demo Atlas Prod API** and the non-Multi-AZ `cloudox-demo-atlas-prod-pg` instance. Key architectural questions this raises:

- **No observed redundancy:** There is no evidence in this section of a read replica, standby instance, or failover target for `cloudox-demo-atlas-prod-pg`. The account ID for the DB instance itself is not confirmed (`Unknown`), which limits full traceability.
- **Blast radius:** A zone failure affecting `cloudox-demo-atlas-prod-pg` would directly impact the production API workload. Whether upstream consumers (e.g. the Cloudox workload or internet-facing API Gateway endpoints) would also be affected depends on call chains not fully resolved here.
- **Dev/Prod isolation:** Dev (`cloudox-demo-atlas-dev`, account 105769365151) and Prod (`cloudox-demo-atlas-prod-api`, account 122122642149) workloads are in separate accounts, which is a positive isolation pattern. The sandbox (`cloudox-demo-sandbox-scratch`, account 161388682021) is similarly separated.
- **Tagging gap:** With 761 untagged resources relying on inference, dependency chains involving those resources carry elevated uncertainty. This limits the ability to fully map coupling across tiers.

**Modernisation opportunity:** Enabling Multi-AZ on `cloudox-demo-atlas-prod-pg` is the highest-priority structural improvement surfaced in this section. Beyond that, formalising tier and environment tagging would materially improve dependency visibility and reduce inference-based classification across the environment.

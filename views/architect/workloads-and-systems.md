# Architect View — Workloads & Systems

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Workloads & Systems

The environment resolves into one cross-account system spanning two lifecycle stages (dev and prod), with a separate sandbox workload and a core platform workload. The most architecturally significant signal is that **761 resources carry no Environment / Stage / Tier tag**, meaning workload boundaries and tier assignments are inferred rather than declared — a classification debt that directly affects blast-radius analysis, cost attribution, and promotion-gate enforcement.

### Systems

One logical system is identifiable from the evidence: **Cloudox Demo Atlas Dev** (`Cloudox Demo Atlas Dev`). It is classified at the system level with *Assumed* confidence, meaning CloudoX inferred its boundary from resource naming and relationships rather than an explicit system-of-record tag or manifest. No additional system-level groupings are evidenced in this section's package.

### Workloads

Five workloads are identified across three AWS accounts, all anchored in `eu-central-1`:

| Friendly Name | Workload ID | Account | Confidence |
|---|---|---|---|
| Cloudox | `cloudox` | 122122642149 | Verified |
| Cloudox Demo Atlas Prod API | `cloudox-demo-atlas-prod-api` | 122122642149 | Likely |
| Cloudox Demo Atlas Dev | `cloudox-demo-atlas-dev` | 105769365151 | Likely |
| Cloudox Demo Atlas Dev API | `cloudox-demo-atlas-dev-api` | 105769365151 | Likely |
| Cloudox Demo Sandbox Scratch | `cloudox-demo-sandbox-scratch` | 161388682021 | Assumed |

**Cloudox** (`cloudox`, account `122122642149`) is the only Verified workload — it has the strongest evidence grounding and sits in the same account as the prod API, suggesting it is the platform or control-plane workload.

**Atlas Dev** and **Atlas Dev API** (`cloudox-demo-atlas-dev`, `cloudox-demo-atlas-dev-api`) share account `105769365151` and are backed by the DynamoDB table `arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`. The dev API is internet-reachable via the API Gateway endpoint `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`.

**Atlas Prod API** (`cloudox-demo-atlas-prod-api`, account `122122642149`) is backed by `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items` and is internet-reachable via `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`.

**Cloudox Demo Sandbox Scratch** (`cloudox-demo-sandbox-scratch`, account `161388682021`) is classified with *Assumed* confidence. It holds a DynamoDB table (`arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`) and appears to be an unstructured experimentation space. Its relationship to the Atlas system is not evidenced.

> **Confidence note:** Four of the five workloads are *Likely* or *Assumed*. Architects should treat workload boundaries as working hypotheses until tagging is remediated.

### Components & Tiers

From the evidence available, the Atlas workloads follow a two-tier pattern visible in both dev and prod:

- **API tier** — Amazon API Gateway (internet-facing, `execute-api.eu-central-1.amazonaws.com` endpoints) acting as the public entry point. Both dev (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`) and prod (`https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) endpoints are reachable from `internet`.
- **Data tier** — Amazon DynamoDB tables serving as the persistence layer for each environment (`cloudox-demo-atlas-dev-items`, `cloudox-demo-atlas-prod-items`, `cloudox-demo-sandbox-scratch`).

No compute tier (Lambda, ECS, EC2) is evidenced within this section's package; those components, if present, are covered in other sections of this view.

Internet gateways are present across multiple accounts and regions (`igw-00ed21b9a0e6596a8` and `igw-0d14f1dd4e54d5906` in account `110019496666` across `us-east-1` and `eu-central-1`; `igw-0567575921f471548` in account `105769365151` `us-east-1`; `igw-0cff0d66b4fd90803` in account `122980216815` `us-east-1`). The presence of IGWs in `us-east-1` alongside workloads declared in `eu-central-1` is a design question worth resolving — it may indicate multi-region expansion, legacy VPC scaffolding, or account-level defaults that have not been cleaned up.

**Design issues to address:**
1. **Tag coverage gap** — 761 resources lack Environment/Stage/Tier tags, making automated workload classification unreliable and complicating any future IaC drift or cost-allocation work.
2. **Dev API is internet-exposed** — The dev API Gateway endpoint is publicly reachable. If this is not intentional, it represents an unnecessary attack surface.
3. **Cross-region IGW presence** — Internet gateways in `us-east-1` for accounts whose declared workloads are in `eu-central-1` warrant review for residual or unmanaged infrastructure.
4. **Sandbox isolation** — `cloudox-demo-sandbox-scratch` sits in a dedicated account (`161388682021`), which is a sound isolation pattern, but its *Assumed* confidence means its scope and ownership are not fully declared.

![Workload architecture](./diagrams/architect-workload-architecture.png)

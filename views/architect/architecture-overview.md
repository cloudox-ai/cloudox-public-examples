# Architect View — Architecture Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Architecture Overview

The environment is a multi-account AWS estate centred on **eu-central-1**, running a small number of clearly separated workloads across distinct lifecycle stages. The dominant pattern is serverless API + managed NoSQL storage, with internet-facing endpoints exposed through API Gateway. Four internet gateways across multiple accounts indicate that VPC-based compute also exists alongside the serverless tier, though the relationship between those two layers is not fully resolved from available evidence.

### Architectural Shape

The estate spans **7 accounts** hosting **807 resources**, with **12 significant workloads** inferred from graph evidence (3 further workloads were demoted as helper or governance constructs, and 3 systems were also identified). The primary workload pattern visible in the evidence is:

- **API Gateway → DynamoDB** — two public HTTPS endpoints (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) front workloads that persist to DynamoDB tables in eu-central-1. Both endpoints are reachable from `internet`, making them externally accessible surfaces.
- **VPC / IGW presence** — four internet gateways are present across accounts `110019496666` (both eu-central-1 and us-east-1), `105769365151` (us-east-1), and `122980216815` (us-east-1). This signals VPC-based workloads in at least two regions, though no specific compute or service detail is available in this section's evidence.
- **us-east-1 footprint** — three of the four internet gateways are in us-east-1, suggesting a secondary regional presence that is not reflected in the primary eu-central-1 workload definitions. The relationship between the us-east-1 infrastructure and the named workloads is not established from available evidence.

> **Confidence: Likely.** Shape is derived from graph inference. 761 of 807 resources carry no Environment/Stage/Tier tag, so workload boundaries and environment assignments rely on inference rather than explicit tagging.

### Environments & Accounts

Three lifecycle environments are distinguishable from the key entities and evidence:

| Environment | Account | Primary Region | Key Evidence |
|---|---|---|---|
| **Production** | `122122642149` | eu-central-1 | Workloads: *Cloudox Demo Atlas Prod API* (`cloudox-demo-atlas-prod-api`), *Cloudox* (`cloudox`); DynamoDB table `cloudox-demo-atlas-prod-items` |
| **Development** | `105769365151` | eu-central-1 | Workloads: *Cloudox Demo Atlas Dev* (`cloudox-demo-atlas-dev`), *Cloudox Demo Atlas Dev API* (`cloudox-demo-atlas-dev-api`); DynamoDB table `cloudox-demo-atlas-dev-items`; API Gateway endpoint `xdmn5ldmif`; IGW `igw-0567575921f471548` (us-east-1) |
| **Sandbox** | `161388682021` | eu-central-1 | Workload: *Cloudox Demo Sandbox Scratch* (`cloudox-demo-sandbox-scratch`); DynamoDB table `cloudox-demo-sandbox-scratch` |

Account `110019496666` hosts two internet gateways (eu-central-1 and us-east-1) but does not map to a named workload in this section's evidence — its role is unknown. Account `122980216815` similarly has an IGW in us-east-1 with no workload attribution available.

The *Cloudox* workload (`cloudox`, account `122122642149`) is the only **Verified**-confidence entity; all others are **Likely** or **Assumed**.

### Key Patterns

**Serverless-first, internet-exposed API tier.** The Atlas workloads (dev and prod) follow a consistent pattern: API Gateway endpoint → (implied compute/integration layer) → DynamoDB. Both the dev API endpoint (`xdmn5ldmif`) and at least one prod-side endpoint (`gfwaiva01f`) are directly reachable from the internet. There is no evidence in this section of a WAF, CloudFront distribution, or private API configuration in front of these endpoints — that is a gap worth validating.

**Account-per-environment isolation.** Production, development, and sandbox each occupy a dedicated AWS account, which is a sound isolation boundary. The sandbox account (`161388682021`) has its own DynamoDB table (`cloudox-demo-sandbox-scratch`), consistent with an experimental or scratch-pad use.

**Tagging debt is a structural risk.** 761 of 807 resources (≈94%) have no Environment/Stage/Tier tag. This undermines cost allocation, automated governance, and any tooling that relies on tag-based targeting. Until tagging is remediated, workload boundaries and environment assignments across the estate remain inferred rather than authoritative.

**Unresolved us-east-1 footprint.** Three IGWs in us-east-1 across accounts `110019496666`, `105769365151`, and `122980216815` indicate active VPC infrastructure in a region where no named workload is defined in available evidence. Whether this represents active workloads, legacy infrastructure, or DR capacity is not known from this section's evidence and warrants direct investigation.

![AWS Organizations account structure](./diagrams/architect-org-account-structure.png)

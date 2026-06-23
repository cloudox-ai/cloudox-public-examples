# Generic View — Environment Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Environment Overview

This is a multi-account AWS organisation built around a clear separation of concerns: dedicated accounts for workload delivery, platform services, audit, log archiving, sandbox experimentation, and central management. Across these seven accounts, **807 resources** have been discovered, spanning at least two AWS regions. Four significant workloads are running, supported by one platform system, with one additional workload classified as a helper or governance construct.

> **Confidence: Likely** — Account structure and workload classification are derived from graph evidence. Some relationships and environment assignments rely on inference where tagging is absent.

### Scope & Accounts

Seven accounts make up the observed scope:

| Friendly Name | Account ID | Confidence | Role |
|---|---|---|---|
| Management Account | 110319895932 | Verified | Org management |
| Workload Dev Account | 105769365151 | Verified | Development workloads |
| Workload Prod Account | 122122642149 | Verified | Production workloads |
| Sandbox Ma Account | 161388682021 | Verified | Sandbox / experimentation |
| Audit Account | 110019496666 | Likely | Audit trail |
| Log Archive Account | 122980216815 | Likely | Centralised log storage |
| Platform Account | 150982215529 | Likely | Shared platform services |

Three environments are confirmed with high confidence: **Development Environment** (105769365151-development), **Production Environment** in the Workload Prod Account (122122642149-production), and **Sandbox Environment** (161388682021-sandbox). Two further production-tier environments are inferred with lower confidence: one in the Platform Account (150982215529-production) and one in the Log Archive Account whose environment classification is currently unknown (122980216815-unknown).

The Audit Account (110019496666) and Log Archive Account (122980216815) follow a pattern consistent with AWS Control Tower or a similar landing zone, though this is not explicitly confirmed in this section's evidence.

### Regions & Footprint

Evidence points to activity in at least two AWS regions:

- **eu-central-1 (Frankfurt)** — API Gateway endpoints, DynamoDB tables, and internet gateways are all observed here. This appears to be the primary region for workload resources.
- **us-east-1 (N. Virginia)** — Internet gateways are present across multiple accounts (110019496666, 105769365151, 122980216815), indicating VPC infrastructure with internet connectivity in this region as well.

Specific internet-facing API Gateway endpoints observed:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` (Workload Dev Account, 105769365151)
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` (Workload Dev Account, 105769365151)

DynamoDB tables with confirmed presence:
- `cloudox-demo-atlas-dev-items` — eu-central-1, Workload Dev Account (105769365151)
- `cloudox-demo-atlas-prod-items` — eu-central-1, Workload Prod Account (122122642149)
- `cloudox-demo-sandbox-scratch` — eu-central-1, Sandbox Ma Account (161388682021)

The naming pattern (`cloudox-demo-atlas-*`) suggests a workload named **Atlas** with a dev/prod promotion path, and a separate scratch table in the sandbox account.

> **Note:** The Resource Explorer and Cloud Control meta-collectors were unavailable during discovery. The regional footprint described here is based on typed collector evidence only; additional regions or resource types may exist that are not reflected in this section.

### What This Environment Is

This is the **cloudox-demo** workspace — a multi-account AWS environment structured around a workload called **Atlas**, which has distinct development and production deployments, each with its own account, API Gateway surface, and DynamoDB persistence layer. A sandbox account provides an isolated space for experimentation, separate from the promotion path.

The presence of dedicated Audit, Log Archive, and Management accounts points to a governance-aware landing zone design. The Platform Account adds a layer of shared services, though its specific contents are not detailed in this section's evidence.

Internet connectivity is established in both eu-central-1 and us-east-1 via internet gateways across multiple accounts, and the Atlas workload exposes public API endpoints in eu-central-1.

**Key unknowns to be aware of:**
- 761 of the 807 resources carry no Environment, Stage, or Tier tag; their classification depends on inference and may not be fully accurate.
- Resource Explorer and Cloud Control meta-collectors were not available, so the full breadth of resource types and regions cannot be independently verified from this section.
- The environment classification for the Log Archive Account (122980216815) remains unresolved.

![AWS Organizations account structure](./diagrams/generic-org-account-structure.png)

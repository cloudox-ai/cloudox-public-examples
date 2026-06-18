# Generic View — Key Workloads

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Key Workloads

The environment hosts **12 significant workloads** (out of 15 inferred) spread across **7 accounts** and **807 resources**, with the majority anchored in **eu-central-1**. The production tier carries a notable resilience concern that warrants attention before any availability incident occurs.

> **Confidence: Likely** — workload boundaries and classifications are graph-inferred; 761 resources carry no Environment/Stage/Tier tag and rely on inference rather than explicit tagging.

### Significant Workloads

Five workloads are identified as key entities across the workspace:

| Friendly Name | Account | Region | Confidence |
|---|---|---|---|
| Cloudox | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | 122122642149 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | 161388682021 | eu-central-1 | Assumed |

**Cloudox** (ref: `cloudox`, account `122122642149`) is the only Verified workload entity, indicating the strongest evidence for its boundary and composition. **Cloudox Demo Sandbox Scratch** (ref: `cloudox-demo-sandbox-scratch`, account `161388682021`) carries only Assumed confidence — its classification should be treated as provisional.

Three additional workloads were demoted as helper/governance workloads and are not listed here.

### Composition & Tiers

Evidence points to at least two tiers of deployment — **production** and **development** — mirrored across separate AWS accounts:

- **Production** workloads run in account `122122642149`, with an API surface exposed via `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` and a DynamoDB table at `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`.
- **Development** workloads run in account `105769365151`, with a corresponding API at `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` and a DynamoDB table at `arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`.
- **Sandbox** activity is isolated to account `161388682021`, backed by `arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`.

Internet gateways are present across multiple accounts and regions (eu-central-1 and us-east-1), indicating that at least some workloads have internet-facing exposure:

- `arn:aws:ec2:eu-central-1:110019496666:internet-gateway/igw-0d14f1dd4e54d5906`
- `arn:aws:ec2:us-east-1:110019496666:internet-gateway/igw-00ed21b9a0e6596a8`
- `arn:aws:ec2:us-east-1:105769365151:internet-gateway/igw-0567575921f471548`
- `arn:aws:ec2:us-east-1:122980216815:internet-gateway/igw-0cff0d66b4fd90803`

The presence of us-east-1 gateways is noted; no workloads in the key entities list are attributed to us-east-1, so the relationship between those gateways and the named workloads is not established from this package.

### Key Dependencies

**⚠️ Priority concern: Single-AZ database in the production critical path**

The **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account `122122642149`, eu-central-1) depends on a DBInstance named **`cloudox-demo-atlas-prod-pg`** that is not configured for Multi-AZ. A single availability zone failure would take the production data tier offline for this workload.

| Detail | Value |
|---|---|
| Intelligence item | `dependency_concern:architecture:cloudox-demo-atlas-prod-pg` |
| Affected datastore | `cloudox-demo-atlas-prod-pg` |
| Affected workload | Cloudox Demo Atlas Prod API |
| Impact | Single-AZ datastore in a workload's critical path |
| Confidence | Verified |

**Recommended action:** Confirm whether Multi-AZ is enabled and whether automated backups are in place for `cloudox-demo-atlas-prod-pg`. Additionally, review how the Cloudox Demo Atlas Prod API handles a data-tier failure — for example, whether it degrades gracefully or fails hard. No decision is flagged as immediately required, but this is a priority 3 finding with verified evidence.

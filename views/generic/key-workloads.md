# Generic View — Key Workloads

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Key Workloads

The environment centres on a small set of identified workloads spread across three AWS accounts, all anchored in **eu-central-1**. The production API workload carries a confirmed resilience concern at its data tier that warrants attention.

### Significant Workloads

Five workloads have been identified across three accounts:

| Friendly Name | Workload ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox | `cloudox` | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | `cloudox-demo-atlas-prod-api` | 122122642149 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | `cloudox-demo-atlas-dev` | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | `cloudox-demo-atlas-dev-api` | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | `cloudox-demo-sandbox-scratch` | 161388682021 | eu-central-1 | Assumed |

The pattern suggests a **prod / dev / sandbox** lifecycle split across separate AWS accounts. **Cloudox** (account `122122642149`) is the only Verified workload; the remaining four are Likely or Assumed based on graph inference.

> **Confidence note:** 761 resources carry no Environment / Stage / Tier tag, so workload boundaries and classifications rely on inference rather than explicit tagging. Treat Likely and Assumed workload identities with appropriate caution.

### Composition & Tiers

Evidence points to API-tier components backed by managed datastores:

- **API endpoints** are present for both the dev workload (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`) and the prod workload (`https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`), both served via Amazon API Gateway in eu-central-1 and reachable from the internet.
- **DynamoDB tables** back each environment:
  - Dev: `cloudox-demo-atlas-dev-items` (account `105769365151`, eu-central-1)
  - Prod: `cloudox-demo-atlas-prod-items` (account `122122642149`, eu-central-1)
  - Sandbox: `cloudox-demo-sandbox-scratch` (account `161388682021`, eu-central-1)
- **Internet gateways** are present across multiple accounts and regions (eu-central-1 and us-east-1), indicating that at least some VPC-hosted resources have internet egress or ingress paths. The us-east-1 gateways (`igw-00ed21b9a0e6596a8` in account `110019496666`, `igw-0567575921f471548` in account `105769365151`, `igw-0cff0d66b4fd90803` in account `122980216815`) and the eu-central-1 gateway (`igw-0d14f1dd4e54d5906` in account `110019496666`) are in scope; their exact attachment and routing are covered in network-focused sections.

### Key Dependencies

One dependency concern has been identified and is **Verified**:

> **Critical dependency — `cloudox-demo-atlas-prod-pg`** (`dependency_concern:architecture:cloudox-demo-atlas-prod-pg`)
>
> The **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account `122122642149`) depends on the DBInstance **`cloudox-demo-atlas-prod-pg`**. This datastore is not configured for Multi-AZ. A single availability-zone failure would take the production data tier offline, directly impacting the dependent workload.
>
> **Recommended action:** Confirm whether Multi-AZ is enabled and whether automated backups are in place. Review how the Cloudox Demo Atlas Prod API handles a data-tier failure — for example, whether it degrades gracefully or fails hard.

The account and region of `cloudox-demo-atlas-prod-pg` itself could not be confirmed from available evidence (confidence: Unknown for that resource). The dependency relationship and its impact are nonetheless Verified through graph analysis.

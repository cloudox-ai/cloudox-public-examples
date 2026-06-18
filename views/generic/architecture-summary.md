# Generic View — Architecture Summary

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Architecture Summary

This environment spans **807 resources across 7 AWS accounts**, with the primary workloads concentrated in **eu-central-1** and at least one account extending into **us-east-1**. The estate is organised around a clear dev/prod/sandbox pattern, with internet-facing API surfaces and DynamoDB-backed storage visible across the main workload accounts.

> **Confidence: Likely** — Architecture is derived from graph evidence. Some relationships and workload boundaries rely on inference rather than explicit tagging (see Unknowns below).

### Systems & Workloads

CloudoX identified **12 significant workloads** out of 15 inferred; 3 were demoted as helper or governance workloads and are not detailed here. The significant workloads resolve into **3 systems**.

| Workload | Account | Region | Confidence |
|---|---|---|---|
| Cloudox | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | 122122642149 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | 161388682021 | eu-central-1 | Assumed |

Two workloads — **Cloudox** (`cloudox`) and the **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`) — carry Verified confidence, meaning their identity and boundaries are directly evidenced. The **Cloudox Demo Sandbox Scratch** workload (`cloudox-demo-sandbox-scratch`) is Assumed; treat its scope and membership with caution until confirmed.

Three VPCs are present with Unknown confidence and no friendly names resolved — `vpc-0fd81e106f24b4e85` (account 161388682021, us-east-1), `vpc-02298727f33db8908` (account 122980216815, eu-central-1), and `vpc-0038de9730553c915` (account 110019496666, eu-central-1). Their workload membership and purpose are not established from available evidence.

### How It Fits Together

**Internet exposure** is confirmed across multiple accounts. Two API Gateway endpoints are publicly reachable:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`

Four internet gateways are evidenced across accounts 110019496666 (eu-central-1 and us-east-1), 105769365151 (us-east-1), and 122980216815 (us-east-1), confirming that several VPCs have direct internet paths.

**Persistent storage** is provided by DynamoDB tables in each of the three primary workload accounts:
- `cloudox-demo-atlas-dev-items` (account 105769365151, eu-central-1)
- `cloudox-demo-atlas-prod-items` (account 122122642149, eu-central-1)
- `cloudox-demo-sandbox-scratch` (account 161388682021, eu-central-1)

The naming pattern suggests a consistent data model replicated across dev, prod, and sandbox stages, with each stage isolated in its own AWS account — a common account-per-environment separation strategy. This is an inference from naming and account structure; no explicit cross-account relationship evidence is in scope for this section.

**VPC topology** follows the same account boundary: a dedicated VPC per environment — **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`), **Cloudox Demo Atlas Prod VPC** (`vpc-0aaa6d4f2f981e945`), and **Cloudox Demo Sandbox VPC** (`vpc-0c97d53850c027a14`) — all in eu-central-1. The three unnamed VPCs noted above fall outside this pattern and remain uncharacterised.

> **Unknown:** 761 of 807 resources carry no Environment / Stage / Tier tag. Workload and system boundaries for those resources are inferred rather than declared, which means the workload count and membership should be treated as approximate until tagging is improved.

![Workload architecture](./diagrams/generic-workload-architecture.png)

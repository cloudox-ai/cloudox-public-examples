# Generic View — Architecture Summary

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Architecture Summary

This environment spans multiple AWS accounts and is centred on a serverless, API-driven architecture deployed primarily in **eu-central-1**, with at least one account carrying resources in **us-east-1**. The clearest signal is a pair of internet-facing API Gateway endpoints backed by DynamoDB tables, representing distinct development and production workloads under the **Cloudox Demo Atlas** umbrella. A sandbox account adds a third, more loosely defined workload. Several VPCs exist across the accounts, though the purpose of some remains unclear.

> **Confidence: Likely** — derived from graph evidence; workload boundaries and some VPC roles rely on inference rather than explicit tagging.

### Systems & Workloads

Four workloads and one broader system are identified across three AWS accounts:

| Friendly Name | Account | Region | Confidence |
|---|---|---|---|
| Cloudox Demo Atlas Prod API | 122122642149 | eu-central-1 | Likely |
| Cloudox | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Dev | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | 161388682021 | eu-central-1 | Assumed |

A system-level entity **Cloudox Demo Atlas Dev** (ref: `Cloudox Demo Atlas Dev`) is also present without an account assignment, suggesting it represents a logical grouping that spans or precedes the account-level workload decomposition.

**Production (account 122122642149):** The **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`) is the production-facing API surface. It is backed by the DynamoDB table `cloudox-demo-atlas-prod-items` (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`) and sits within the **Cloudox Demo Atlas Prod VPC** (`vpc-0aaa6d4f2f981e945`). The **Cloudox** workload (`cloudox`) is also resident in this account and region.

**Development (account 105769365151):** The **Cloudox Demo Atlas Dev API** workload (`cloudox-demo-atlas-dev-api`) mirrors the production pattern, backed by `cloudox-demo-atlas-dev-items` (`arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`) and housed in the **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`). The API Gateway endpoint `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` is associated with this workload.

**Sandbox (account 161388682021):** The **Cloudox Demo Sandbox Scratch** workload (`cloudox-demo-sandbox-scratch`) is an assumed classification backed by the DynamoDB table `cloudox-demo-sandbox-scratch` (`arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`) and the **Cloudox Demo Sandbox VPC** (`vpc-0c97d53850c027a14`). This workload's boundaries and purpose are less well-defined than the Atlas workloads.

### How It Fits Together

**Internet ingress** is the primary entry point for the Atlas API workloads. Two API Gateway endpoints are in evidence:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com` (dev account, 105769365151)
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com` (prod account, 122122642149)

Both are reachable from `internet`, indicating public-facing APIs. Traffic flows from the internet through these endpoints to the respective DynamoDB tables within each account's VPC boundary.

**Internet Gateways** are attached across multiple accounts and regions, confirming outbound or inbound internet connectivity beyond just the API Gateway endpoints:

| IGW | Account | Region |
|---|---|---|
| `igw-00ed21b9a0e6596a8` | 110019496666 | us-east-1 |
| `igw-0d14f1dd4e54d5906` | 110019496666 | eu-central-1 |
| `igw-0567575921f471548` | 105769365151 | us-east-1 |
| `igw-0cff0d66b4fd90803` | 122980216815 | us-east-1 |

Three VPCs carry no friendly name and their roles are not established from available evidence:
- `vpc-0fd81e106f24b4e85` — account 161388682021, us-east-1 (Unknown)
- `vpc-02298727f33db8908` — account 122980216815, eu-central-1 (Unknown)
- `vpc-0038de9730553c915` — account 110019496666, eu-central-1 (Unknown)

Accounts **110019496666** and **122980216815** appear in the internet gateway and VPC evidence but are not associated with any named workload in this section. Their role in the broader environment is not established here.

> **Gap:** 761 resources carry no Environment / Stage / Tier tag, meaning workload and environment classifications for a significant portion of the estate rely on inference rather than explicit metadata. Tag coverage should be reviewed to improve classification accuracy.

![Workload architecture](./diagrams/generic-workload-architecture.png)

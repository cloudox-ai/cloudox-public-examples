# Operations View — Operational Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Operational Overview

The environment spans **807 resources across 7 accounts**, with 12 significant workloads active across Development, Production, and Sandbox environments. Three additional workloads have been classified as helper/governance and are not primary operational concerns. This breadth, combined with known observability gaps, means the operational blast radius of any account-level incident is wider than what typed collectors alone can confirm.

### What's Running

The following accounts and environments make up the operational footprint:

| Account | Friendly Name | Environment(s) | Confidence |
|---|---|---|---|
| 110319895932 | Management Account | — | Verified |
| 105769365151 | Workload Dev Account | Development Environment | Verified |
| 122122642149 | Workload Prod Account | Production Environment (122122642149) | Verified |
| 161388682021 | Sandbox Ma Account | Sandbox Environment | Verified |
| 122980216815 | Log Archive Account | 122980216815 Unknown | Likely / Assumed |
| 110019496666 | Audit Account | — | Likely |
| 150982215529 | Platform Account | Production Environment (150982215529) | Likely |

Workload-bearing environments confirmed by evidence:

- **Development Environment** (Workload Dev Account `105769365151`): Hosts the `cloudox-demo-atlas-dev-items` DynamoDB table (`arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`) and an API Gateway endpoint at `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`, both in `eu-central-1`. An internet gateway (`igw-0567575921f471548`) is present in `us-east-1` for this account.
- **Production Environment** (Workload Prod Account `122122642149`): Hosts the `cloudox-demo-atlas-prod-items` DynamoDB table (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`) in `eu-central-1`. An internet gateway (`igw-0cff0d66b4fd90803`) is present in `us-east-1`.
- **Sandbox Environment** (Sandbox Ma Account `161388682021`): Hosts the `cloudox-demo-sandbox-scratch` DynamoDB table (`arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`) in `eu-central-1`.
- **Audit Account** (`110019496666`): Internet gateways present in both `eu-central-1` (`igw-0d14f1dd4e54d5906`) and `us-east-1` (`igw-00ed21b9a0e6596a8`), indicating active networking in two regions.

Two API Gateway endpoints are internet-facing (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`), both in `eu-central-1`. These represent externally reachable ingress points that require monitoring and access control attention.

### Operational Footprint

**Coverage gaps — act on these:**

- **761 of 807 resources carry no Environment / Stage / Tier tag.** Classification for these resources relies entirely on inference. Any automation, alerting, or cost allocation that depends on tags will have significant blind spots. Tag remediation is a prerequisite for reliable operational segmentation.
- **Resource Explorer meta-collector was disabled or unavailable.** The full AWS-visible resource breadth could not be cross-checked. There may be resource types or regions not captured by typed collectors that are operationally active but invisible to this view.
- **Cloud Control meta-collector was disabled.** Long-tail and newer resource types are limited to what typed collectors explicitly cover. Untracked resources cannot be assessed for backup, observability, or recovery posture.

**Multi-region exposure:** Internet gateways are confirmed in both `eu-central-1` and `us-east-1` across multiple accounts (Workload Dev, Workload Prod, Audit). This means operational runbooks, incident response, and monitoring must account for at least two AWS regions. The presence of `us-east-1` internet gateways alongside `eu-central-1` workload resources warrants verification that these gateways are intentional and not residual.

**Account confidence note:** The Log Archive Account (`122980216815`) environment classification is Assumed; the Audit Account (`110019496666`) and Platform Account (`150982215529`) are Likely. Operational procedures that depend on these accounts should be validated against authoritative account inventory before relying on this classification.

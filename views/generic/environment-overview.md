# Generic View — Environment Overview

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Environment Overview

This is a multi-account AWS organisation structured around a clear separation of concerns: dedicated accounts for management, audit, log archiving, platform services, workload delivery (dev and prod), and sandboxing. Across these seven accounts, **807 resources** have been discovered, hosting **12 significant workloads** (out of 15 inferred; 3 classified as helper/governance).

> **Confidence: Likely** — Evidence is derived from graph analysis. Some classification relies on inference rather than explicit tagging, and two meta-collectors were unavailable during discovery, meaning the full resource breadth may be wider than what is represented here.

### Scope & Accounts

Seven accounts make up this environment:

| Friendly Name | Account ID | Confidence | Role |
|---|---|---|---|
| Management Account | 110319895932 | Verified | Org root / management |
| Workload Dev Account | 105769365151 | Verified | Development workloads |
| Workload Prod Account | 122122642149 | Verified | Production workloads |
| Sandbox Ma Account | 161388682021 | Verified | Sandbox / experimentation |
| Audit Account | 110019496666 | Likely | Audit trail |
| Log Archive Account | 122980216815 | Likely | Centralised log storage |
| Platform Account | 150982215529 | Likely | Shared platform services |

Three accounts — Audit, Log Archive, and Platform — are classified at **Likely** confidence, meaning their roles are inferred from evidence rather than confirmed by explicit tagging or naming conventions. The Log Archive account's environment stage is currently **Unknown** (`122980216815-unknown`), indicating no stage tag or strong signal was found to classify it further.

Three workloads were demoted from the significant workload list as helper or governance constructs rather than primary application workloads.

### Regions & Footprint

Active resource evidence spans at least two AWS regions:

- **eu-central-1 (Frankfurt)** — API Gateway endpoints (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`), DynamoDB tables in the Dev account (`arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items`), Prod account (`arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items`), and Sandbox account (`arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch`), plus internet gateways in the Audit account (`arn:aws:ec2:eu-central-1:110019496666:internet-gateway/igw-0d14f1dd4e54d5906`).
- **us-east-1 (N. Virginia)** — Internet gateways present in the Audit account (`arn:aws:ec2:us-east-1:110019496666:internet-gateway/igw-00ed21b9a0e6596a8`), Workload Dev account (`arn:aws:ec2:us-east-1:105769365151:internet-gateway/igw-0567575921f471548`), and Log Archive account (`arn:aws:ec2:us-east-1:122980216815:internet-gateway/igw-0cff0d66b4fd90803`).

The presence of internet gateways in multiple accounts — including Audit and Log Archive — indicates that VPCs with public routing exist beyond just the workload accounts. The practical significance of those gateways (whether actively used or residual) is not determinable from available evidence.

> **Note:** The discovery summary reports **0 regions** in the structured scope metadata, which conflicts with the regional evidence above. This is likely a metadata gap rather than a true zero-region footprint. The Resource Explorer meta-collector was disabled or unavailable, which means AWS-visible regional breadth could not be independently cross-checked.

### What This Environment Is

This is a **workload-hosting AWS organisation** built around the `cloudox-demo` workspace. The naming conventions (`cloudox-demo-atlas-dev-items`, `cloudox-demo-atlas-prod-items`, `cloudox-demo-sandbox-scratch`) point to an application called **Atlas** with at least two promoted stages — development and production — and an active sandbox for experimentation.

The environment follows a recognisable landing-zone pattern: separate accounts for management, audit, and log archiving sit alongside workload and platform accounts. Internet-facing components are confirmed in eu-central-1 (public API Gateway endpoints) and internet gateways exist across multiple accounts and regions, indicating intentional public connectivity in at least the workload tier.

A material tagging gap exists: **761 of 807 resources carry no Environment, Stage, or Tier tag**, meaning workload classification throughout this view relies on inference. Readers should treat workload boundaries and environment assignments as indicative rather than authoritative until tagging is improved. Additionally, the absence of the Cloud Control meta-collector means long-tail or newer AWS resource types may be under-represented in the 807-resource count.

![AWS Organizations account structure](./diagrams/generic-org-account-structure.png)

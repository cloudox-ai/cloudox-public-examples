# Operations View — Connectivity & Routing

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Connectivity & Routing

The environment spans **15 VPCs** and **63 subnets**, with **9 internet-facing access paths** observed across accounts. Three named VPCs are confirmed in `eu-central-1`, each with a dedicated Internet Gateway and public subnets — meaning internet exposure is distributed across multiple accounts and requires per-account security posture management. A single DynamoDB Interface Endpoint is present in the production VPC, indicating at least partial private-path AWS service access in that account.

### Network Connectivity

Three VPCs are confirmed as key entities across three AWS accounts, all in `eu-central-1`:

| Friendly Name | VPC ID | Account | Region |
|---|---|---|---|
| Cloudox Demo Atlas Dev VPC | `vpc-0a4d44a07f48d7ca0` | `105769365151` | eu-central-1 |
| Cloudox Demo Atlas Prod VPC | `vpc-0aaa6d4f2f981e945` | `122122642149` | eu-central-1 |
| Cloudox Demo Sandbox VPC | `vpc-0c97d53850c027a14` | `161388682021` | eu-central-1 |

Five public subnets are confirmed across these three accounts (all `eu-central-1`):

| Subnet ID | Account |
|---|---|
| `subnet-016c22941a019a137` | `122122642149` (Prod) |
| `subnet-013a24d318ed6f3d0` | `122122642149` (Prod) |
| `subnet-0f64d71c952a7898a` | `105769365151` (Dev) |
| `subnet-065f522206524ab12` | `105769365151` (Dev) |
| `subnet-029e2cceb3d0beff7` | `161388682021` (Sandbox) |

All five confirmed subnets are **public-facing**. No private subnets are listed as key entities in this section's package — coverage of private subnet topology is not represented here.

Two AWS-managed prefix lists are referenced in the environment:
- `pl-66a5400f` (`arn:aws:ec2:eu-central-1:aws:prefix-list/pl-66a5400f`)
- `pl-6ea54007` (`arn:aws:ec2:eu-central-1:aws:prefix-list/pl-6ea54007`)

These are AWS-owned prefix lists (e.g. S3 or CloudFront ranges) used in routing or security group rules. Their specific usage context — which route tables or security groups reference them — is not detailed in this package.

### Routing & Gateways

Each of the three VPCs has a dedicated Internet Gateway:

| Friendly Name | IGW ID | VPC / Account |
|---|---|---|
| Cloudox Demo Atlas Dev Igw Internet Gateway | `igw-0155958a0a5e9c500` | Dev VPC / `105769365151` |
| Cloudox Demo Atlas Prod Igw Internet Gateway | `igw-01dd5625ec1ea7a54` | Prod VPC / `122122642149` |
| Sandbox Internet Gateway | `igw-08017528fa36ccb4d` | Sandbox VPC / `161388682021` |

All three accounts have independent internet breakout — there is no evidence of a shared/centralised egress model (e.g. Transit Gateway with centralised inspection) in this package. Operational note: security group and NACL hygiene must be maintained independently per account.

One VPC endpoint is confirmed:

| Friendly Name | Endpoint ID | Type | Account |
|---|---|---|---|
| DynamoDB Interface Endpoint (eu-central-1) | `vpce-02ddbd8d63a6ab447` | Interface | `122122642149` (Prod) |

This endpoint provides private connectivity to DynamoDB from the Prod VPC. No equivalent endpoints are listed for the Dev or Sandbox VPCs in this package — DynamoDB or other AWS service traffic from those VPCs may traverse the public internet.

### Egress Paths

Nine internet-facing access paths are observed across the environment. All three VPCs have direct internet egress via their respective Internet Gateways. One load balancer is present in the environment (details covered in the relevant inventory section).

Operational risks to note:
- **Distributed internet exposure**: Three separate IGWs across three accounts means no single choke point for egress monitoring or filtering. Each account's route tables and security groups are the primary control plane.
- **Prefix list dependency**: Security rules referencing `pl-66a5400f` and `pl-6ea54007` are tied to AWS-managed prefix list updates. If these lists change, effective security group rules change automatically — operators should be aware of which rules depend on them.
- **Endpoint coverage gap**: The DynamoDB Interface Endpoint (`vpce-02ddbd8d63a6ab447`) is only confirmed in the Prod account. Whether Dev or Sandbox workloads access AWS services privately is not evidenced in this package.
- **Public subnet concentration**: All five confirmed key-entity subnets are public. Workload placement in public subnets increases blast radius if security group or NACL rules are misconfigured.

![Workload VPC network topology](./diagrams/operations-network-topology.png)

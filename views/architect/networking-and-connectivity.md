# Architect View — Networking & Connectivity

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Verified_

## Networking & Connectivity

The network topology spans three isolated VPCs across two AWS accounts, all anchored in **eu-central-1**. Each VPC carries its own Internet Gateway, meaning internet egress is distributed rather than centralised — a design choice with direct implications for traffic inspection, cost visibility, and blast-radius control. One VPC-level private connectivity endpoint exists, but coverage is partial.

### VPCs & Subnets

Three VPCs are in scope across three accounts:

| VPC | Friendly Name | Account | Region |
|-----|--------------|---------|--------|
| `vpc-0a4d44a07f48d7ca0` | Cloudox Demo Atlas Dev VPC | 105769365151 | eu-central-1 |
| `vpc-0aaa6d4f2f981e945` | Cloudox Demo Atlas Prod VPC | 122122642149 | eu-central-1 |
| `vpc-0c97d53850c027a14` | Cloudox Demo Sandbox VPC | 161388682021 | eu-central-1 |

Five subnets are identified, all classified as **Public Subnet** across the three accounts:

| Subnet ID | Account | Region |
|-----------|---------|--------|
| `subnet-016c22941a019a137` | 122122642149 | eu-central-1 |
| `subnet-013a24d318ed6f3d0` | 122122642149 | eu-central-1 |
| `subnet-0f64d71c952a7898a` | 105769365151 | eu-central-1 |
| `subnet-065f522206524ab12` | 105769365151 | eu-central-1 |
| `subnet-029e2cceb3d0beff7` | 161388682021 | eu-central-1 |

**Design issue — public-only subnet posture:** All five discovered subnets are public. No private subnets are present in this section's evidence. Workloads placed in these subnets are directly reachable from the internet (subject to security group and NACL rules), which increases the attack surface for compute and data-tier resources. A private-subnet tier with NAT-based egress is the conventional mitigation for production and dev workloads that do not require inbound internet exposure.

### Routing & Egress

Each VPC has a dedicated Internet Gateway:

| IGW | Friendly Name | VPC Account |
|-----|--------------|-------------|
| `igw-0155958a0a5e9c500` | Cloudox Demo Atlas Dev Igw Internet Gateway | 105769365151 |
| `igw-01dd5625ec1ea7a54` | Cloudox Demo Atlas Prod Igw Internet Gateway | 122122642149 |
| `igw-08017528fa36ccb4d` | Sandbox Internet Gateway | 161388682021 |

The 1:1 IGW-to-VPC pattern is standard AWS architecture and confirms that internet egress is decentralised — each environment breaks out independently. This is consistent with an account-per-environment isolation model, but it means there is no single choke-point for egress filtering or flow logging aggregation. Architects evaluating a hub-and-spoke or centralised egress model (e.g., via AWS Network Firewall or a Transit Gateway inspection VPC) should note this as a modernisation opportunity.

Two AWS-managed prefix lists are referenced in the environment (`pl-66a5400f`, `pl-6ea54007`), both in eu-central-1. These correspond to AWS service IP ranges used in route tables or security group rules. Their presence indicates some routing or filtering rules reference AWS-managed service CIDRs, but the specific route tables and rules that consume them are not detailed in this section's evidence.

### Private Connectivity

One VPC endpoint is present:

| Endpoint | Type | Account | Region |
|----------|------|---------|--------|
| `vpce-02ddbd8d63a6ab447` | DynamoDB Interface Endpoint | 122122642149 | eu-central-1 |

This endpoint is in the **Atlas Prod** account (`122122642149`), providing private routing to DynamoDB without traversing the public internet. This is a positive pattern for the production environment.

**Design issue — incomplete private connectivity coverage:** No equivalent VPC endpoints are evidenced for the **Atlas Dev** (`105769365151`) or **Sandbox** (`161388682021`) VPCs. If those environments also use DynamoDB or other AWS services (S3, SSM, Secrets Manager, etc.), traffic to those services currently routes via the IGW over the public internet. This is both a security gap and a potential cost consideration (data transfer charges). Extending gateway or interface endpoints to dev and sandbox VPCs, and auditing which AWS services are accessed without endpoints in all three VPCs, is a concrete modernisation action.

No VPC peering connections, Transit Gateway attachments, or PrivateLink service endpoints are present in this section's evidence. Cross-VPC and cross-account connectivity patterns are not established from this evidence set.

![Workload VPC network topology](./diagrams/architect-network-topology.png)

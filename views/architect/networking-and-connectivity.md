# Architect View — Networking & Connectivity

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Verified_

## Networking & Connectivity

The environment spans **15 VPCs and 63 subnets** across eu-central-1, with **25 internet-facing access paths** observed — a surface area that warrants careful review for any workloads carrying sensitive data. Three named VPCs anchor the topology: **Cloudox Demo Atlas Prod VPC** (`vpc-0aaa6d4f2f981e945`, account `122122642149`), **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`, account `105769365151`), and **Cloudox Demo Sandbox VPC** (`vpc-0c97d53850c027a14`, account `161388682021`), all in eu-central-1. One load balancer is present in the environment.

### VPCs & Subnets

All three named VPCs are single-region (eu-central-1) and each carries at least one **Public Subnet** in the key-entity set:

| VPC | Account | Public Subnets (observed) |
|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | `122122642149` | `subnet-016c22941a019a137`, `subnet-013a24d318ed6f3d0` |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | `105769365151` | `subnet-0f64d71c952a7898a`, `subnet-065f522206524ab12` |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | `161388682021` | `subnet-029e2cceb3d0beff7` |

The package surfaces only public subnets in the key-entity set. Whether private or isolated subnets exist within these VPCs — and how workloads are distributed across them — is not evidenced here, though the total subnet count of 63 across 15 VPCs implies additional subnet tiers exist beyond what is named.

**Design note:** The presence of public subnets in both Prod and Dev VPCs, combined with 25 internet-facing access paths, suggests workloads or endpoints are directly reachable from the internet. Architects should confirm whether all public-subnet placements are intentional and whether security group / NACL controls are appropriately scoped.

### Routing & Egress

Each of the three named VPCs has a dedicated **Internet Gateway** attached:

| VPC | Internet Gateway |
|---|---|
| Cloudox Demo Atlas Dev VPC | Cloudox Demo Atlas Dev Igw (`igw-0155958a0a5e9c500`) |
| Cloudox Demo Atlas Prod VPC | Cloudox Demo Atlas Prod Igw (`igw-01dd5625ec1ea7a54`) |
| Cloudox Demo Sandbox VPC | Sandbox Internet Gateway (`igw-08017528fa36ccb4d`) |

All three environments — including Prod — have direct internet egress/ingress capability. No NAT Gateway, Transit Gateway, or VPN/Direct Connect entity is evidenced in this package. Whether those constructs exist in the remaining 12 VPCs or serve as the egress path for private subnets is unknown from this context.

Two AWS-managed prefix lists are referenced in the environment: `pl-66a5400f` and `pl-6ea54007` (both `arn:aws:ec2:eu-central-1:aws:prefix-list/…`). These are standard AWS-managed prefix lists for eu-central-1 (typically S3 and DynamoDB service CIDRs used in route tables or security groups), indicating some routing or security-group rules reference AWS service ranges — consistent with the VPC endpoint observed below.

**Design consideration:** With internet gateways on all three named VPCs and 25 internet-facing paths, the blast radius of a misconfigured security group or NACL is elevated. Architects should validate whether a hub-and-spoke or shared-egress model (e.g., centralised NAT or inspection VPC) would reduce exposure, particularly for the Prod environment.

### Private Connectivity

One **DynamoDB Interface Endpoint** (`vpce-02ddbd8d63a6ab447`) is present in the Prod VPC account (`122122642149`, eu-central-1). This routes DynamoDB traffic from that VPC over the AWS private network rather than the public internet, which aligns with the AWS-managed prefix list references noted above.

No evidence is found in this package for:
- VPC peering connections between the Prod, Dev, or Sandbox VPCs
- Transit Gateway attachments
- PrivateLink endpoints beyond the single DynamoDB interface endpoint
- VPN or Direct Connect for on-premises connectivity

**Modernization opportunity:** Only one VPC endpoint is evidenced. If workloads in Dev or Sandbox also access DynamoDB or other AWS services (S3, SSM, Secrets Manager, etc.), adding equivalent endpoints would eliminate unnecessary internet-path exposure and reduce data-transfer costs. The absence of observed inter-VPC private connectivity (peering or Transit Gateway) means Dev-to-Prod or Sandbox-to-Prod traffic, if it exists, likely traverses the public internet — a dependency worth validating and potentially replacing with private routing.

![Workload VPC network topology](./diagrams/architect-network-topology.png)

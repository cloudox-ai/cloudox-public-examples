# Networking Architecture

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

## VPC Inventory

15 VPCs are present across 7 accounts and 2 regions (`eu-central-1`, `us-east-1`), per `digest.totals.vpcs`. Three VPCs carry explicit environment tags and named identifiers; the remainder use default AWS-generated names and have no environment tag in the data.

| VPC ID | Account | Region | CIDR | Env Tag | Name |
|---|---|---|---|---|---|
| `vpc-0aaa6d4f2f981e945` | `122122642149` (workload-prod) | eu-central-1 | 10.40.0.0/16 | prod | cloudox-demo-atlas-prod-vpc |
| `vpc-0a4d44a07f48d7ca0` | `105769365151` (workload-dev) | eu-central-1 | 10.30.0.0/16 | dev | cloudox-demo-atlas-dev-vpc |
| `vpc-0c97d53850c027a14` | `161388682021` (sandbox-ma) | eu-central-1 | 10.50.0.0/16 | sandbox | cloudox-demo-sandbox-vpc |
| `vpc-082aa2987b732c239` | `110319895932` (CloudoX mgmt) | eu-central-1 | 172.31.0.0/16 | — | vpc-082aa2987b732c239 |
| `vpc-03887c51c92a0061d` | `110319895932` (CloudoX mgmt) | us-east-1 | 172.31.0.0/16 | — | vpc-03887c51c92a0061d |
| `vpc-02298727f33db8908` | `122980216815` (log-archive) | eu-central-1 | 172.31.0.0/16 | — | vpc-02298727f33db8908 |
| `vpc-0819c77622c7a1e86` | `122980216815` (log-archive) | us-east-1 | 172.31.0.0/16 | — | vpc-0819c77622c7a1e86 |
| `vpc-0038de9730553c915` | `110019496666` (audit) | eu-central-1 | 172.31.0.0/16 | — | vpc-0038de9730553c915 |
| `vpc-0563bd6a8386b6eb8` | `110019496666` (audit) | us-east-1 | 172.31.0.0/16 | — | vpc-0563bd6a8386b6eb8 |
| `vpc-06bc92d9927a2b9a6` | `161388682021` (sandbox-ma) | eu-central-1 | 172.31.0.0/16 | — | vpc-06bc92d9927a2b9a6 |
| `vpc-0fd81e106f24b4e85` | `161388682021` (sandbox-ma) | us-east-1 | 172.31.0.0/16 | — | vpc-0fd81e106f24b4e85 |
| `vpc-0fdf08606e046467e` | `105769365151` (workload-dev) | eu-central-1 | 172.31.0.0/16 | — | vpc-0fdf08606e046467e |
| `vpc-0b807fb10d159b43d` | `105769365151` (workload-dev) | us-east-1 | 172.31.0.0/16 | — | vpc-0b807fb10d159b43d |
| `vpc-0e0655b83d41e71e6` | `122122642149` (workload-prod) | eu-central-1 | 172.31.0.0/16 | — | vpc-0e0655b83d41e71e6 |
| `vpc-08bb10d6c7cf4425c` | `122122642149` (workload-prod) | us-east-1 | 172.31.0.0/16 | — | vpc-08bb10d6c7cf4425c |

The 172.31.0.0/16 CIDR is the AWS default VPC range. Twelve of the fifteen VPCs use this range, which suggests they may be default VPCs rather than purpose-built ones — this should be validated with the customer.

---

## Internet Connectivity

All 15 VPCs have an Internet Gateway attached (`has_igw: true` on every VPC in the digest). No NAT Gateways are present anywhere (`digest.totals.nat_gateways: 0`). No Transit Gateways, VPC peering connections, or VPN connections are detected (`digest.totals.transit_gateways: 0`, `transit_gateway_attachments: 0`, `vpc_peering_connections: 0`, `vpn_connections: 0`).

---

## Public Subnets

Five subnets are classified as public based on route table entries that forward traffic to an Internet Gateway (per `digest.public_subnets` and individual exposure findings):

| Subnet ID | Name | AZ | VPC | Account |
|---|---|---|---|---|
| `subnet-013a24d318ed6f3d0` | cloudox-demo-atlas-prod-public-a | eu-central-1a | `vpc-0aaa6d4f2f981e945` | `122122642149` |
| `subnet-016c22941a019a137` | cloudox-demo-atlas-prod-public-b | eu-central-1b | `vpc-0aaa6d4f2f981e945` | `122122642149` |
| `subnet-065f522206524ab12` | cloudox-demo-atlas-dev-public-a | eu-central-1a | `vpc-0a4d44a07f48d7ca0` | `105769365151` |
| `subnet-0f64d71c952a7898a` | cloudox-demo-atlas-dev-public-b | eu-central-1b | `vpc-0a4d44a07f48d7ca0` | `105769365151` |
| `subnet-029e2cceb3d0beff7` | cloudox-demo-sandbox-public | eu-central-1a | `vpc-0c97d53850c027a14` | `161388682021` |

Route table evidence:
- `rtb-01181117bc9ca1e3e` → attached to `subnet-013a24d318ed6f3d0` and `subnet-016c22941a019a137` → routes to `igw-01dd5625ec1ea7a54`
- `rtb-04732cf316d6c4917` → attached to `subnet-0f64d71c952a7898a` and `subnet-065f522206524ab12` → routes to `igw-0155958a0a5e9c500`
- `rtb-0ef271019f865470e` → attached to `subnet-029e2cceb3d0beff7` → routes to `igw-08017528fa36ccb4d`

The remaining subnets in the named workload VPCs (`subnet-015bacbc19e17a4e6`, `subnet-0cc6baabafce38fa1` in `vpc-0a4d44a07f48d7ca0`; `subnet-0369fbadec4ff1ccc`, `subnet-0b0384769d442046a` in `vpc-0aaa6d4f2f981e945`) are associated with route tables that route to VPC endpoints rather than an IGW, suggesting they are private — but this should be confirmed against the full route table configuration.

---

## Internet-Facing Load Balancer

One Application Load Balancer is present and confirmed internet-facing:

| Resource | Account | Region | Scheme | VPC | Subnets |
|---|---|---|---|---|---|
| `cloudox-demo-atlas-prod-alb` | `122122642149` | eu-central-1 | internet-facing | `vpc-0aaa6d4f2f981e945` | `subnet-016c22941a019a137`, `subnet-013a24d318ed6f3d0` |

The ALB is secured by security group `sg-06f2b4190bf01d261` (`cloudox-demo-atlas-prod-sg-alb`), which has an ingress rule allowing `0.0.0.0/0` on port 80 (per `digest.security_group_exposure`). A listener (`9b700d6b85e65ced`) routes to target group `cloudox-demo-atlas-prod-tg`, which is in `vpc-0aaa6d4f2f981e945`.

Two Elastic IPs are present (`digest.totals.elastic_ips: 2`): `eipalloc-046fb5322a51361f6` attached to `eni-03dbd4a8c443d2c1c` (in `subnet-016c22941a019a137`, secured by `sg-06f2b4190bf01d261`) and `eipalloc-05bf6bad2ba044175` attached to `eni-0f36adeff7612b121` (in `subnet-013a24d318ed6f3d0`, secured by `sg-06f2b4190bf01d261`). These appear to be ALB ENIs based on the subnet and security group alignment, but the relationship between the EIPs and the ALB is not directly stated in the data.

---

## Security Group Exposure

Three security groups have ingress rules permitting traffic from `0.0.0.0/0` or `::/0`, per `digest.security_group_exposure`:

| Security Group ID | Name | VPC | World-Open Ports | Sensitive Ports |
|---|---|---|---|---|
| `sg-054446d655de1ee7f` | cloudox-demo-sandbox-sg-permissive | `vpc-0c97d53850c027a14` | 22, 80, 5432 | 22, 5432 |
| `sg-06f2b4190bf01d261` | cloudox-demo-atlas-prod-sg-alb | `vpc-0aaa6d4f2f981e945` | 80 | — |
| `sg-0459201826f8de5b3` | cloudox-demo-atlas-prod-sg-edge | `vpc-0aaa6d4f2f981e945` | 443 | — |

`sg-054446d655de1ee7f` in the sandbox account exposes port 22 (SSH) and port 5432 (PostgreSQL) to the world. This is the most notable exposure in the data. The sandbox VPC (`vpc-0c97d53850c027a14`) has an IGW attached and a public subnet (`subnet-029e2cceb3d0beff7`), so resources in that subnet using this security group would be directly reachable from the internet on those ports.

`sg-0459201826f8de5b3` (`cloudox-demo-atlas-prod-sg-edge`) allows inbound 443 from the world. A security group-to-security group `allows_from` relationship exists: `sg-016daa1c9dc89e0de` allows from `sg-0459201826f8de5b3`, and `sg-0457a284b8b9ead6c` allows from `sg-06f2b4190bf01d261`, and `sg-0831f3eae117601d2` allows from `sg-0457a284b8b9ead6c`. This chain suggests a layered ingress path from the ALB/edge into the workload, but the full chain should be validated.

---

## VPC Endpoints

Four Gateway-type VPC endpoints are present, covering S3 and DynamoDB in the two workload accounts:

| Endpoint ID | Account | Service | VPC |
|---|---|---|---|
| `vpce-0e75224f50dddfb8d` | `105769365151` (workload-dev) | com.amazonaws.eu-central-1.s3 | `vpc-0a4d44a07f48d7ca0` |
| `vpce-0f376e51c370a0f95` | `105769365151` (workload-dev) | com.amazonaws.eu-central-1.dynamodb | `vpc-0a4d44a07f48d7ca0` |
| `vpce-02ddbd8d63a6ab447` | `122122642149` (workload-prod) | com.amazonaws.eu-central-1.dynamodb | `vpc-0aaa6d4f2f981e945` |
| `vpce-0232f3af5b06cd3cf` | `122122642149` (workload-prod) | com.amazonaws.eu-central-1.s3 | `vpc-0aaa6d4f2f981e945` |

Route table `rtb-009215bffcfd9f9a0` (dev, attached to private subnets `subnet-015bacbc19e17a4e6` and `subnet-0cc6baabafce38fa1`) routes to both dev endpoints. Route table `rtb-0fb49391955972271` (prod, attached to `subnet-0369fbadec4ff1ccc` and `subnet-0b0384769d442046a`) routes to both prod endpoints. No Interface-type endpoints or PrivateLink endpoints are detected in the data.

No VPC endpoints are present in the management, audit, log-archive, or sandbox accounts based on the provided data.

---

## Cross-Account and Cross-Region Connectivity

No VPC peering connections, Transit Gateway attachments, or VPN connections are detected in the data (`digest.totals` confirms all at zero). No cross-account or cross-region network relationships are present in the relationships block. Each VPC is isolated at the network layer based on available evidence.

---

## Unknowns / Gaps

- **NAT Gateway absence**: No NAT Gateways exist anywhere in the environment. Private subnets in the workload VPCs have VPC endpoint routes for S3 and DynamoDB, but any other outbound internet traffic from private subnets has no observed egress path. Whether workloads in private subnets require internet egress (e.g., for container image pulls, third-party APIs) is unknown.
- **Default VPCs**: 12 of 15 VPCs use the 172.31.0.0/16 CIDR and have no environment tag or meaningful name. These appear to be default VPCs. Whether they contain active workloads or are unused is not determinable from the data.
- **Route table completeness**: Only route table entries with explicit `routes_to` relationships are visible. Full route table configurations (e.g., local routes, all destination CIDRs) are not present in the data.
- **Network ACLs**: 15 NACLs are present (`digest.totals.network_acls: 15`) but no NACL rules or associations are included in the provided data.
- **platform-prod account (`150982215529`)**: No VPC, subnet, or networking resource is present in the discovery data for this account.
- **EIP-to-ALB relationship**: The two EIPs (`eipalloc-046fb5322a51361f6`, `eipalloc-05bf6bad2ba044175`) are attached to ENIs in the ALB's subnets and security group, but no direct relationship to the ALB resource is present in the data.

---

## Follow-up Questions

1. **NAT Gateway absence**: Are workloads in private subnets expected to reach the internet? If so, how is outbound connectivity currently handled — or is it intentionally absent?
2. **Default VPCs**: Can the 12 unnamed 172.31.0.0/16 VPCs be confirmed as unused default VPCs, or do any contain active resources that were not captured in this discovery?
3. **Sandbox security group `sg-054446d655de1ee7f`**: Ports 22 and 5432 are open to `0.0.0.0/0`. Is this intentional for the sandbox environment, and are any resources currently attached to this security group?
4. **No inter-account networking**: Is the isolation between accounts (no peering, no TGW) intentional, or is shared-services connectivity planned? The platform-prod account in particular has no detected networking.
5. **NACLs**: Are any NACLs configured with non-default rules? This cannot be determined from the current data.
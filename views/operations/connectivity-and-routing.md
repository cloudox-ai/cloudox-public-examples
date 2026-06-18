# Operations View — Connectivity & Routing

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Connectivity & Routing

The environment spans **15 VPCs** and **63 subnets** across accounts, with **25 internet-facing access paths** observed and a single load balancer in scope. All key entities are in `eu-central-1`. Three named VPCs anchor the topology: **Cloudox Demo Atlas Dev VPC** (`vpc-0a4d44a07f48d7ca0`, account `105769365151`), **Cloudox Demo Atlas Prod VPC** (`vpc-0aaa6d4f2f981e945`, account `122122642149`), and **Cloudox Demo Sandbox VPC** (`vpc-0c97d53850c027a14`, account `161388682021`).

### Network Connectivity

Five subnets are explicitly identified as public-facing across the three primary accounts:

| Friendly Name | Subnet ID | Account | Region |
|---|---|---|---|
| Public Subnet (eu-central-1) | `subnet-016c22941a019a137` | 122122642149 (Prod) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-013a24d318ed6f3d0` | 122122642149 (Prod) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-0f64d71c952a7898a` | 105769365151 (Dev) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-065f522206524ab12` | 105769365151 (Dev) | eu-central-1 |
| Public Subnet (eu-central-1) | `subnet-029e2cceb3d0beff7` | 161388682021 (Sandbox) | eu-central-1 |

All five are in the same region. The presence of public subnets in **all three environments** — including Sandbox — means internet exposure is not limited to production. The 25 internet-facing access paths observed across the environment warrant review to confirm each is intentional.

A **DynamoDB Interface Endpoint** (`vpce-02ddbd8d63a6ab447`) is present in the Prod account (`122122642149`), routing DynamoDB traffic privately within the VPC. No equivalent endpoint is evidenced for the Dev or Sandbox VPCs — traffic to DynamoDB from those environments may traverse the public internet. No other VPC endpoints are evidenced in this package.

Two AWS-managed prefix lists are referenced in the environment:
- `pl-66a5400f` (`arn:aws:ec2:eu-central-1:aws:prefix-list/pl-66a5400f`)
- `pl-6ea54007` (`arn:aws:ec2:eu-central-1:aws:prefix-list/pl-6ea54007`)

These are AWS-managed prefix lists for `eu-central-1` (commonly used for S3 and DynamoDB gateway routing). Their presence in security group or route table rules is the likely context, but specific rule associations are not evidenced in this package.

### Routing & Gateways

Each of the three primary VPCs has a dedicated Internet Gateway:

| Gateway | ID | VPC / Account |
|---|---|---|
| Cloudox Demo Atlas Dev Igw | `igw-0155958a0a5e9c500` | Dev VPC / 105769365151 |
| Cloudox Demo Atlas Prod Igw | `igw-01dd5625ec1ea7a54` | Prod VPC / 122122642149 |
| Sandbox Internet Gateway | `igw-08017528fa36ccb4d` | Sandbox VPC / 161388682021 |

All three VPCs have direct internet egress capability via their respective IGWs. No NAT Gateways, Transit Gateways, or VPC peering connections are evidenced in this package. The absence of evidence for inter-VPC routing means lateral connectivity between Dev, Prod, and Sandbox is not confirmed — treat as unknown.

One load balancer is present in the environment. Its type, placement (public vs. internal), and target configuration are not detailed in this package.

### Egress Paths

With IGWs attached to all three VPCs and public subnets in each, all three environments have direct outbound internet paths. The **25 internet-facing access paths** observed across the environment represent a meaningful operational surface:

- **Operational risk:** Any misconfigured security group or route table on a public subnet in any account can expose workloads directly. This is equally true for Dev and Sandbox as for Prod.
- **No evidence of centralised egress control** (e.g., a shared egress VPC, firewall appliance, or AWS Network Firewall) is present in this package. Egress filtering posture is unknown.
- The DynamoDB Interface Endpoint in Prod (`vpce-02ddbd8d63a6ab447`) provides a private egress path for DynamoDB traffic in that account. No equivalent private egress for S3 or other services is evidenced.

The AWS-managed prefix lists (`pl-66a5400f`, `pl-6ea54007`) may be used to scope gateway-route or security-group rules for AWS service traffic, but their exact usage context is not confirmed in this package.

![Workload VPC network topology](./diagrams/operations-network-topology.png)

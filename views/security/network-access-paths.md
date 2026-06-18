# Security View — Network Access Paths

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Verified_

## Network Access Paths

**Confidence: Verified** — Derived from complete graph evidence for this domain.

The most security-relevant finding here is the combination of internet-facing exposure across multiple environments and a production data tier that sits in a single availability zone. With 25 internet-facing access paths observed and 2 security groups open to the internet, the attack surface is non-trivial and warrants structured review. The sections below break this down by ingress and lateral connectivity.

---

### Ingress Paths

Three VPCs — all in `eu-central-1` — carry internet gateways, meaning each has a direct on-ramp from the public internet:

| VPC | Account | Internet Gateway |
|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | 122122642149 | `igw-01dd5625ec1ea7a54` |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | 105769365151 | `igw-0155958a0a5e9c500` |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | 161388682021 | `igw-08017528fa36ccb4d` |

All three environments — production, development, and sandbox — are internet-reachable. For Security & Governance teams, the presence of an internet gateway on the Dev and Sandbox VPCs alongside Production is a governance signal: if these environments share any credentials, data, or peering paths, the lower-assurance environments could become a stepping stone toward production.

Five public subnets are confirmed across these three VPCs (accounts 122122642149, 105769365151, and 161388682021), all in `eu-central-1`. Resources placed in these subnets are directly reachable from the internet subject to security group and NACL rules.

**Security groups open to the internet:** 2 are confirmed. The package does not identify which resources these groups are attached to, so the precise exposure surface cannot be fully characterised from this evidence alone — this is a material gap (see Unknowns).

**25 internet-facing access paths** are observed in total across the estate. The package does not enumerate each path individually, but the combination of three internet gateways, five public subnets, and two permissive security groups is consistent with this count.

One **DynamoDB Interface Endpoint** (`vpce-02ddbd8d63a6ab447`, account 122122642149, `eu-central-1`) is present in the Prod environment. This routes DynamoDB traffic privately within the VPC rather than over the public internet — a positive signal for that specific service path. No other VPC endpoints are evidenced in this package.

AWS-managed prefix lists `pl-66a5400f` and `pl-6ea54007` (eu-central-1) are referenced in the evidence graph. These are standard AWS-managed prefix lists (typically used for S3 and DynamoDB gateway routing); their presence indicates route table or security group rules reference them, but the package does not confirm the specific rules or their scope.

---

### Lateral Connectivity

The package does not contain direct evidence of VPC peering, Transit Gateway attachments, or other explicit cross-VPC routing between the three environments. Absence of evidence here is not confirmation of isolation — this is a material unknown that should be validated (see Unknowns).

The most significant lateral risk evidenced is the dependency of the **Cloudox Demo Atlas Prod API** workload (`cloudox-demo-atlas-prod-api`, account 122122642149) on the datastore **`cloudox-demo-atlas-prod-pg`** (`dependency_concern:architecture:cloudox-demo-atlas-prod-pg`). This datastore is not Multi-AZ. While this is primarily a resilience concern, it has a security-governance dimension: a zone failure or targeted disruption of the data tier would take the production API's data layer offline. The recommended action is to confirm whether Multi-AZ or backup controls are in place and to review how the API handles a data-tier failure.

> **Confidence on the workload dependency:** Verified for the dependency relationship. The datastore entity `cloudox-demo-atlas-prod-pg` carries Unknown confidence on its account/region attributes, meaning its precise placement cannot be confirmed from current evidence.

No evidence of Security Hub enablement was discovered. This means centralised, cross-account security findings aggregation cannot be confirmed, and the 25 internet-facing paths and 2 open security groups may not be under active automated monitoring.

![Workload VPC network topology](./diagrams/security-network-topology.png)

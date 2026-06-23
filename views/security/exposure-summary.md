# Security View — Exposure Summary

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Exposure Summary

**Confidence: Likely** — derived from graph evidence; some VPCs and accounts have incomplete metadata (see Unknowns below).

The most urgent finding is a **critical-severity security group in the Sandbox account** (`161388682021`) that exposes SSH (port 22) and PostgreSQL (port 5432) to the entire internet — this requires an immediate decision. Beyond that, 9 internet-facing access paths are observed across the environment, spanning 3 VPCs with attached Internet Gateways, 4 public subnets, 2 additional permissive security groups, and 1 internet-facing load balancer.

---

### Internet-Reachable Resources

#### Critical — Requires Decision

| Resource | Account | Ports Exposed | Severity |
|---|---|---|---|
| `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) | `161388682021` (Sandbox) | **22 (SSH), 5432 (PostgreSQL)** — `0.0.0.0/0` and `::/0` | **Critical** |

This security group (**Confidence: Verified**) allows unrestricted inbound access on SSH and PostgreSQL from any source on both IPv4 and IPv6. SSH exposure at the network perimeter is a direct remote-access risk; PostgreSQL exposure suggests a database port may be reachable without an application-layer intermediary. A decision is required: confirm whether this exposure is intentional and document the justification, or restrict the ingress rules immediately.

> **Recommended action:** Confirm whether the exposure is required. If not, restrict ingress rules to known CIDR ranges or remove the rules entirely. (`arn:aws:ec2:eu-central-1:161388682021:security-group/sg-054446d655de1ee7f`)

---

#### Medium — Confirm and Document

The following resources are **Verified** as internet-reachable but are flagged as medium severity. No immediate decision is required, but each should be confirmed as intentional and documented.

**Security Groups (account `122122642149` — Atlas Prod, eu-central-1)**

| Resource | Friendly Name | Port(s) Exposed |
|---|---|---|
| `sg-0459201826f8de5b3` | `cloudox-demo-atlas-prod-sg-edge` | **443** — `0.0.0.0/0` / `::/0` |
| `sg-06f2b4190bf01d261` | `cloudox-demo-atlas-prod-sg-alb` | **80** — `0.0.0.0/0` / `::/0` |

Ports 443 and 80 are consistent with an internet-facing web application or load balancer edge. These are likely intentional for a production service, but should be confirmed and tied to the load balancer below.

**Load Balancer**

| Resource | Account | Scheme |
|---|---|---|
| `cloudox-demo-atlas-prod-alb` | Unknown (account not resolved) | **internet-facing** |

**Confidence: Verified** for the scheme; account and region metadata are **Unknown** for this entity. The internet-facing scheme creates a public attack surface. Confirm this is the intended deployment model for the Atlas Prod workload.

**Public Subnets**

Four subnets have route tables forwarding traffic to an Internet Gateway, making them public subnets (**Confidence: Verified** for all four):

| Subnet ID | Friendly Name | Account | Region |
|---|---|---|---|
| `subnet-016c22941a019a137` | `cloudox-demo-atlas-prod-public-b` | `122122642149` (Atlas Prod) | eu-central-1 |
| `subnet-013a24d318ed6f3d0` | `cloudox-demo-atlas-prod-public-a` | `122122642149` (Atlas Prod) | eu-central-1 |
| `subnet-0f64d71c952a7898a` | `cloudox-demo-atlas-dev-public-b` | `105769365151` (Atlas Dev) | eu-central-1 |
| `subnet-065f522206524ab12` | `cloudox-demo-atlas-dev-public-a` | `105769365151` (Atlas Dev) | eu-central-1 |

Public subnets in both the Atlas Prod and Atlas Dev VPCs are present. Any resource placed in these subnets with a public IP assigned will be directly internet-reachable. Governance teams should confirm that subnet-level public IP auto-assignment policies are reviewed and that only intended resources are placed here.

---

### Exposure Paths

Three VPCs have confirmed Internet Gateways attached (**Confidence: Verified** for all three):

| VPC | Friendly Name | Account | Internet Gateway | Region |
|---|---|---|---|---|
| `vpc-0aaa6d4f2f981e945` | Cloudox Demo Atlas Prod VPC | `122122642149` | `igw-01dd5625ec1ea7a54` | eu-central-1 |
| `vpc-0a4d44a07f48d7ca0` | Cloudox Demo Atlas Dev VPC | `105769365151` | `igw-0155958a0a5e9c500` | eu-central-1 |
| `vpc-0c97d53850c027a14` | Cloudox Demo Sandbox VPC | `161388682021` | `igw-08017528fa36ccb4d` | eu-central-1 |

The Sandbox VPC (`vpc-0c97d53850c027a14`) is the highest-risk path: it has an Internet Gateway and contains the critical permissive security group (`sg-054446d655de1ee7f`) exposing SSH and PostgreSQL. The combination of an IGW, a public-routable subnet, and a world-open security group constitutes a complete internet exposure path for any resource in that group.

For Atlas Prod and Atlas Dev, the exposure paths run through the public subnets listed above, the ALB security groups (ports 80/443), and the internet-facing load balancer `cloudox-demo-atlas-prod-alb`. This pattern is consistent with a standard public web application architecture, but the HTTP (port 80) exposure on `sg-06f2b4190bf01d261` should be confirmed — if the ALB redirects HTTP to HTTPS, the port 80 rule may be intentional; if not, it represents an unencrypted ingress path.

**Additional VPCs with incomplete evidence:** Several VPCs appear in the key entities list with **Unknown** confidence — including VPCs in accounts `122980216815`, `110019496666`, and additional VPCs in `161388682021` (including one in `us-east-1`) and `122122642149`. Their routing, subnet configuration, and security group posture are not fully resolved in this section. These represent a governance gap: internet exposure cannot be ruled out for these VPCs.

> ⚠️ **Governance gap:** No Security Hub enablement was discovered. Centralised security findings and compliance posture checks are not confirmed to be active. This limits the ability to detect misconfigurations and threats programmatically across accounts.

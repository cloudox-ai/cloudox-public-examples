# Security View — Exposure Summary

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Likely_

## Exposure Summary

**Confidence: Likely** — derived from graph evidence; some unknowns and limitations remain.

The most urgent finding requiring a decision is a sandbox security group with world-open ingress on SSH (port 22) and PostgreSQL (port 5432) — protocols that should never be exposed to the internet. Beyond that critical item, the production Atlas environment carries intentional but unconfirmed internet-facing exposure across a load balancer, two security groups, and public subnets. Four additional public subnets exist across the dev and prod Atlas accounts. All findings are evidence-grounded; account and region metadata are noted where available.

---

### Internet-Reachable Resources

#### Critical — Requires Decision

| Resource | Account | Ports Exposed | Severity |
|---|---|---|---|
| `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) | 161388682021 (Sandbox) | 22 (SSH), 5432 (PostgreSQL) | **Critical** |

Security group `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) has ingress rules permitting `0.0.0.0/0` and/or `::/0` on port 22 (SSH) and port 5432 (PostgreSQL). **This is the only finding in this section flagged `decision_required`.** Unrestricted SSH and database access from the internet represents a direct exploitation path. Confirm whether this exposure is intentional; if not, restrict ingress immediately.

#### Medium — Confirm Intentional

The following resources are internet-reachable and carry medium severity. No immediate decision flag has been raised, but each should be confirmed as intentionally public.

**Load Balancer**

| Resource | Account | Region | Notes |
|---|---|---|---|
| `cloudox-demo-atlas-prod-alb` | Unknown | Unknown | Scheme is internet-facing |

Account and region metadata for `cloudox-demo-atlas-prod-alb` are not available in the current evidence. **Confidence: Unknown** for the account/region attribution of this resource.

**Security Groups (Production Atlas — account 122122642149, eu-central-1)**

| Resource | Ports Exposed |
|---|---|
| `cloudox-demo-atlas-prod-sg-edge` (`sg-0459201826f8de5b3`) | 443 (HTTPS) |
| `cloudox-demo-atlas-prod-sg-alb` (`sg-06f2b4190bf01d261`) | 80 (HTTP) |

Ports 443 and 80 open to `0.0.0.0/0`/`::/0` are consistent with a public-facing web application, but should be confirmed as intentional and reviewed for whether HTTP (port 80) should redirect rather than accept traffic.

**Public Subnets**

Four subnets have route tables forwarding traffic to an Internet Gateway, making them internet-reachable:

| Subnet | Friendly Name | Account | Region |
|---|---|---|---|
| `subnet-013a24d318ed6f3d0` | cloudox-demo-atlas-prod-public-a | 122122642149 | eu-central-1 |
| `subnet-016c22941a019a137` | cloudox-demo-atlas-prod-public-b | 122122642149 | eu-central-1 |
| `subnet-065f522206524ab12` | cloudox-demo-atlas-dev-public-a | 105769365151 | eu-central-1 |
| `subnet-0f64d71c952a7898a` | cloudox-demo-atlas-dev-public-b | 105769365151 | eu-central-1 |

Public subnets in both prod and dev Atlas accounts are present. Resources placed in these subnets inherit internet reachability; the risk depends on what is deployed into them.

---

### Exposure Paths

Three VPCs with confirmed Internet Gateways are in scope:

| VPC | Account | IGW | Region |
|---|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | 122122642149 | `igw-01dd5625ec1ea7a54` | eu-central-1 |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | 105769365151 | `igw-0155958a0a5e9c500` | eu-central-1 |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | 161388682021 | `igw-08017528fa36ccb4d` | eu-central-1 |

The sandbox VPC (`vpc-0c97d53850c027a14`) is the highest-risk path: its Internet Gateway combined with `cloudox-demo-sandbox-sg-permissive` (`sg-054446d655de1ee7f`) creates a direct internet route to SSH and PostgreSQL.

The Atlas prod and dev VPCs each have Internet Gateways and public subnets. The prod path includes the internet-facing ALB (`cloudox-demo-atlas-prod-alb`) fronted by security groups permitting ports 80 and 443 from any source — consistent with a public web tier, but requiring confirmation.

**Additional VPCs with unknown posture:** Several VPCs were discovered with **Unknown** confidence — including unnamed VPCs in accounts 122980216815, 110019496666, and additional VPCs in accounts 161388682021 and 122122642149 (eu-central-1 and us-east-1). Their exposure posture is not established from current evidence. This is a material governance gap.

> **Governance gap:** No Security Hub enablement was discovered. Without Security Hub, continuous security findings aggregation and standards compliance checks are absent, meaning the exposures above may not be surfaced through any automated detection channel.

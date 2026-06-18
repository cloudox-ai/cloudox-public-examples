# Executive View — What Needs Attention

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## What Needs Attention

One critical, decision-required security exposure stands out and warrants immediate leadership action: a sandbox security group is openly exposing SSH and PostgreSQL to the entire internet. Three medium-severity internet-facing exposures round out the picture and should be confirmed as intentional.

### Highest-Priority Issues

**🔴 Critical — Action Required**

| Resource | Exposure | Ports Open to Internet | Decision Needed? |
|---|---|---|---|
| cloudox-demo-sandbox-sg-permissive | World-open ingress | SSH (22), PostgreSQL (5432) | **Yes** |

The security group **cloudox-demo-sandbox-sg-permissive** (Sandbox account, eu-central-1) allows unrestricted inbound traffic from any IP address on port 22 (SSH) and port 5432 (PostgreSQL). Exposing a database port and an administrative access port directly to the internet is a high-likelihood breach vector. Engineering leadership should confirm whether this exposure is intentional and, if not, restrict ingress rules immediately. *(Confidence: Verified)*

**🟡 Medium — Confirm Intent**

Three additional resources in the production Atlas environment carry world-open ingress rules. No immediate decision is flagged, but leadership should confirm these are deliberately internet-facing:

| Resource | Exposure | Ports |
|---|---|---|
| cloudox-demo-atlas-prod-alb (Load Balancer) | Internet-facing scheme | — |
| cloudox-demo-atlas-prod-sg-edge (Security Group) | World-open ingress | HTTPS (443) |
| cloudox-demo-atlas-prod-sg-alb (Security Group) | World-open ingress | HTTP (80) |

An internet-facing load balancer with open HTTP/HTTPS security groups is a common and often intentional pattern for public-facing services. However, the HTTP (port 80) exposure on **cloudox-demo-atlas-prod-sg-alb** should be reviewed — if traffic is expected to redirect to HTTPS, port 80 ingress may be unnecessary. *(Confidence: Verified)*

**Notable gaps to be aware of:**
- No AWS Security Hub enablement was discovered across the environment, meaning centralised security findings and compliance checks may not be active.
- 761 of 807 resources carry no Environment/Stage/Tier tag, limiting the ability to assess blast radius or classify resources by criticality with confidence.

# Executive View — What Needs Attention

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## What Needs Attention

One critical security exposure demands an immediate decision from leadership. Beyond that, three medium-severity internet-facing configurations should be reviewed and confirmed as intentional.

### Highest-Priority Issues

**🔴 Critical — Action required: Sandbox security group open to SSH and PostgreSQL**

The security group **cloudox-demo-sandbox-sg-permissive** (eu-central-1, sandbox account) allows unrestricted inbound traffic from the entire internet on port 22 (SSH) and port 5432 (PostgreSQL). This is a verified, critical-severity finding and the only item in this section flagged as requiring a leadership decision.

- **Risk:** Any actor on the internet can attempt to connect directly to SSH and a PostgreSQL database endpoint. If those services are reachable, the blast radius includes credential brute-force, data exfiltration, and lateral movement.
- **Decision needed:** Confirm whether this exposure is intentional (e.g., a short-lived test resource). If not, engineering should restrict ingress to known IP ranges or remove the rules immediately.

---

**🟡 Medium — Confirm intent: Internet-facing production load balancer and associated security groups**

Three additional findings relate to the production Atlas environment and are likely intentional for a public-facing service, but should be explicitly confirmed:

| Resource | Exposure | Severity |
|---|---|---|
| **cloudox-demo-atlas-prod-alb** (load balancer) | Internet-facing scheme | Medium |
| **cloudox-demo-atlas-prod-sg-edge** (security group) | World-open ingress on 443 (HTTPS) | Medium |
| **cloudox-demo-atlas-prod-sg-alb** (security group) | World-open ingress on 80 (HTTP) | Medium |

Ports 443 and 80 open to the internet on a production load balancer are a common and expected pattern for a public application. However, the HTTP (port 80) exposure is worth confirming — if it exists only to redirect to HTTPS, that is acceptable; if it serves content directly, it should be reviewed. No decision is flagged as required by the analysis, but leadership should ensure the team has documented these as approved intentional exposures.

---

**Notable gaps to be aware of**

- **No AWS Security Hub enablement was discovered.** This means there is no centralised, continuous security findings service active (or its status could not be confirmed). This limits ongoing detection capability.
- **761 resources carry no Environment/Stage/Tier tag** and rely on inference for classification. This makes it harder to assess blast radius, enforce environment-level controls, or allocate costs accurately. A tagging policy decision would address this at scale.

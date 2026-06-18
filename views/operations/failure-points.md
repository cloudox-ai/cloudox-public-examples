# Operations View — Failure Points

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Failure Points

With a single load balancer serving 25 internet-facing access paths across 15 VPCs and 63 subnets, the architecture carries concentrated failure risk at the ingress layer — that one load balancer is the most operationally significant component to watch.

### Single Points of Failure

The most immediately actionable concern is the **single load balancer** observed across the entire environment. With 25 internet-facing access paths routed through it, any failure, misconfiguration, or capacity event at that load balancer has broad blast radius — potentially taking down all externally reachable services simultaneously.

Two AWS-managed prefix lists are referenced in the network configuration:

| Prefix List | ARN |
|---|---|
| pl-66a5400f | `arn:aws:ec2:eu-central-1:aws:prefix-list/pl-66a5400f` |
| pl-6ea54007 | `arn:aws:ec2:eu-central-1:aws:prefix-list/pl-6ea54007` |

These are AWS-managed (not customer-managed), so their content is controlled by AWS and cannot be directly audited or locked. Security group or route table rules that depend on these prefix lists will silently change behaviour if AWS updates the underlying CIDR ranges — a subtle but real operational risk.

### Resilience Gaps

**Tagging coverage is a material operational gap.** 761 of 807 resources (approximately 94%) carry no Environment, Stage, or Tier tag, meaning their classification relies on inference rather than explicit metadata. This directly affects:

- **Incident scoping**: During an outage, operators cannot quickly filter resources by environment (e.g., production vs. non-production) using tags alone.
- **Blast radius assessment**: Without tier tags, it is not possible to reliably determine which resources are customer-facing vs. internal at query time.
- **Automation and runbook targeting**: Any tag-driven automation (auto-remediation, backup policies, scaling rules) will have unreliable coverage across the 807-resource estate.

The 15 VPCs and 63 subnets spread across 7 accounts suggest a multi-account topology, but no evidence is available in this package about cross-VPC connectivity redundancy, Transit Gateway presence, or subnet distribution across Availability Zones. Those are coverage gaps that should be investigated separately to complete the resilience picture.

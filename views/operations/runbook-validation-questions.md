# Operations View — Runbook / Validation Questions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Runbook / Validation Questions

One open validation question requires owner confirmation before it can be closed. The item below is flagged **Assumed** confidence — meaning the concern is inferred from observed configuration rather than confirmed policy — so a human decision is needed to resolve it.

### Operational Checks

The following environmental context is relevant when working through the validation items:

- **Network footprint:** 15 VPCs, 63 subnets, 1 load balancer, and 25 internet-facing access paths are present in scope. Any role with administrative privileges operating in this environment has a correspondingly broad blast radius.
- **Coverage gaps to be aware of:** Two material observability gaps affect the completeness of this runbook:
  - Resource Explorer meta-collector was disabled or unavailable; AWS-visible resource breadth could not be cross-checked against typed collector findings.
  - Cloud Control meta-collector was disabled; long-tail resource types are limited to typed collector coverage only.
  
  These gaps mean additional roles or resources with elevated privileges may exist but are not visible in the current dataset. Treat the validation question below as a minimum, not an exhaustive list.

- **AWS-managed prefix lists in scope** (`pl-66a5400f`, `pl-6ea54007`) are referenced in the network graph. Confirm that security group or route table rules consuming these prefix lists are intentional and reviewed as part of any privilege audit.

### Questions to Validate

The single open item requiring owner response is:

| # | Item ID | Affected Resource | Question |
|---|---------|-------------------|----------|
| 1 | `validation_question:security:cloudox-demo-sandbox-unused-admin` | `cloudox-demo-sandbox-unused-admin` (IAM Role, account `161388682021`) | Does IAM role `cloudox-demo-sandbox-unused-admin` require its current privilege level, and who owns it? |

**Detail — Confirm the scope of IAM role `cloudox-demo-sandbox-unused-admin`**

- **Entity:** IAM role `cloudox-demo-sandbox-unused-admin` (role ID `AROAAAAADPCL3BVEXUDTH`, account `161388682021`)
- **Concern:** The role name contains `unused-admin`, which raises the question of whether administrative privileges are intentional and actively required, or whether this is a dormant role that has not been cleaned up.
- **Confidence:** Assumed — the concern is inferred from observed configuration; no confirmed policy or usage data was available to resolve it automatically.
- **Action required:** Identify the team or individual who owns this role. Confirm whether the current privilege level is necessary. If the role is genuinely unused, assess whether it should be scoped down or removed to reduce standing privilege exposure.
- **No automated recommended action is set** — this requires a human judgement call from the role owner.

> **Note for operators:** Because Resource Explorer and Cloud Control collectors were both unavailable during discovery, it is not possible to confirm whether this is the only over-privileged or unused role in the account. A manual or CLI-assisted review of IAM roles in account `161388682021` is advisable to supplement this finding.

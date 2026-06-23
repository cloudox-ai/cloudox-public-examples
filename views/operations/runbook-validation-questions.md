# Operations View — Runbook / Validation Questions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Verified_

## Runbook / Validation Questions

One open item requires owner confirmation before it can be closed. The check below is flagged **Assumed** — meaning the analysis identified a pattern that warrants human verification rather than a deterministic finding. No immediate automated remediation is recommended; the action is to confirm intent with the role owner.

### Operational Checks

The following prefix lists are present in `eu-central-1` and are referenced in the environment graph. They are AWS-managed and do not require operator action, but are recorded here for traceability:

| Prefix List | ARN |
|---|---|
| pl-66a5400f | `arn:aws:ec2:eu-central-1:aws:prefix-list/pl-66a5400f` |
| pl-6ea54007 | `arn:aws:ec2:eu-central-1:aws:prefix-list/pl-6ea54007` |

> **Coverage gap:** Resource Explorer meta-collector was disabled or unavailable, so AWS-visible resource breadth could not be cross-checked. Cloud Control meta-collector was also disabled, limiting coverage of long-tail resource types to typed collector output. Operational checks in this section reflect only what typed collectors surfaced.

### Questions to Validate

The single open validation question is in the **security** domain. It carries a significance score of 0.36 and does not require an immediate decision, but should be routed to the role owner for confirmation.

---

**Confirm the scope of IAM role `cloudox-demo-sandbox-unused-admin`**
`validation_question:security:cloudox-demo-sandbox-unused-admin`

- **Affected resource:** IAM role `cloudox-demo-sandbox-unused-admin` (`AROAAAAADPCL3BVEXUDTH`) in account `161388682021` (global, no region)
- **Question to resolve with owner:** Does IAM role `cloudox-demo-sandbox-unused-admin` require its current privilege level, and who owns it?
- **Why this matters operationally:** The role name includes the token `unused-admin`, which may indicate it was created for a temporary purpose and never decommissioned, or that it is a standing break-glass role. Either way, an unowned administrative role is a recovery risk — if credentials are ever needed urgently and ownership is unclear, access paths may be blocked or, conversely, the role may be exercised without proper change control.
- **Confidence:** Assumed — the analysis inferred this warrants review based on naming and privilege signals; the role's actual usage history and intent are not confirmed in this dataset.

**Suggested action:** Contact the team responsible for account `161388682021` and confirm (a) whether the role is actively used, (b) who owns it, and (c) whether its administrative privileges are still required. If the role is genuinely unused, consider disabling or deleting it to reduce the standing attack surface.

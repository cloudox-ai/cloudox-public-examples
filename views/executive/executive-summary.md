# Executive View — Executive Summary

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Executive Summary

The environment is operational and multi-account, but carries meaningful visibility and security gaps that warrant leadership attention before they become incidents.

### The Environment in Brief

The estate spans **7 AWS accounts** — including a Management Account, a Workload Dev Account, a Workload Prod Account, a Sandbox Ma Account, and a Log Archive Account — with **807 resources** discovered across them. CloudoX inferred **15 workloads**, of which **12 are significant** (3 were demoted as helper or governance constructs). Two public API Gateway endpoints are present and internet-reachable (`gfwaiva01f` and `xdmn5ldmif`, both in eu-central-1), and DynamoDB tables back workloads in both the Development and Production environments, as well as the Sandbox.

The account structure follows a recognisable landing-zone pattern — separate accounts for production workloads, development, sandbox, and log archiving — which is a positive structural signal.

### Overall Posture

Three issues stand out at the leadership level:

| Area | Finding | Decision needed? |
|---|---|---|
| **Security visibility** | No Security Hub enablement was discovered across any account. Centralised threat and compliance findings are absent. | Yes — enable or confirm an alternative |
| **Tagging discipline** | 761 of 807 resources (94%) carry no Environment, Stage, or Tier tag. Ownership, cost allocation, and blast-radius analysis all rely on inference rather than declared metadata. | Yes — enforce a tagging policy |
| **Discovery coverage** | Resource Explorer and Cloud Control meta-collectors were unavailable during this run. Long-tail resource types and the full AWS-visible breadth could not be cross-checked, meaning the 807-resource count is a lower bound. | Yes — re-enable collectors for a complete picture |

The two internet-facing API endpoints and the production DynamoDB table (`cloudox-demo-atlas-prod-items`) represent the highest-consequence assets in scope. Without Security Hub and with incomplete tagging, the ability to detect, attribute, and respond to an incident against these assets is reduced.

> **Confidence: Likely.** Findings are derived from graph evidence. The absence of Security Hub discovery and the meta-collector gaps mean some risks may be understated. Treat this summary as directionally reliable, not exhaustive.

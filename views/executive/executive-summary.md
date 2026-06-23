# Executive View — Executive Summary

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Executive Summary

The environment is a multi-account AWS organisation with four verified workload accounts, a management account, a sandbox, and a likely log archive account — seven accounts in total hosting 807 discovered resources. Four significant workloads are active in production and development tiers; one additional system is present, and one workload has been classified as a helper or governance function rather than a customer-facing service.

### The Environment in Brief

The organisation follows a recognisable account-separation pattern: a **Management Account**, a **Workload Dev Account**, a **Workload Prod Account**, a **Sandbox Ma Account**, and a **Log Archive Account** (confidence: Likely). Two internet-facing API Gateway endpoints are present in eu-central-1 — one associated with the development environment and one with production — indicating at least one externally reachable service tier. Persistent state is held in DynamoDB tables across dev (`cloudox-demo-atlas-dev-items`), prod (`cloudox-demo-atlas-prod-items`), and sandbox (`cloudox-demo-sandbox-scratch`) accounts.

A material tagging gap exists: 761 of 807 resources carry no Environment, Stage, or Tier tag, meaning workload and environment classification relies on inference rather than authoritative metadata. This limits confidence in any resource-to-workload attribution and should be treated as a governance debt item.

### Overall Posture

**Confidence: Likely** — findings are graph-derived; some gaps remain.

Three issues warrant leadership attention:

1. **Security Hub not detected.** No Security Hub enablement was discovered across the organisation. Without a centralised findings aggregator, there is no unified view of security posture, compliance drift, or cross-account threat signals. This is the highest-priority gap for a production environment with internet-facing endpoints.

2. **Observability blind spots.** The AWS Resource Explorer and Cloud Control meta-collectors were unavailable during discovery. This means the 807-resource count may understate actual breadth — long-tail or newer resource types are not fully covered. Engineering leadership should not treat the current inventory as complete.

3. **Tagging discipline.** With 94% of resources untagged for environment or tier, cost allocation, blast-radius analysis, and automated policy enforcement are all impaired. A tagging policy enforced at account or organisation level is a prerequisite for reliable governance at scale.

**Decision needed:** Prioritise Security Hub enablement (ideally organisation-wide via the Management Account) and establish a tagging enforcement policy before the workload footprint grows further.

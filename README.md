# AWS Discovery Report — Account `118535197059`

> **Public Example Report — Sanitized**
>
> This discovery report is a sanitized public demonstration produced by [CloudoX](https://cloudox.io).
>
> **What was changed:**
> - AWS account IDs replaced with synthetic 12-digit numbers.
> - All resource IDs (VPC, subnet, security group, IAM role, internet gateway, route table,
>   VPC endpoint, ENI, EIP, ACL, etc.) replaced with deterministic synthetic equivalents
>   that preserve the correct prefix, length, and hex/alphanumeric format.
> - Organisation IDs and CloudFormation bucket suffixes replaced.
> - Company and employer names replaced with a generic placeholder.
> - All replacements are internally consistent — a resource ID replaced in one file
>   matches its replacement everywhere else in the report.
>
> **What was kept:**
> - AWS service names, resource types, and regions.
> - Architecture patterns, relationships between resources, networking topology.
> - Security findings, exposure analysis, and observability gaps.
> - Workload structure and discovery-engine output quality.
>
> The report demonstrates what a real CloudoX discovery engagement looks like
> without exposing any real infrastructure or customer data.

---

**Snapshot scope:** 1 account · 1 environment (sandbox, high-confidence explicit classification) · 277 total resources · 7 findings · 2 workloads detected

---

## Report Index

| File | Purpose |
|------|---------|
| [01-executive-summary.md](01-executive-summary.md) | Give leadership and consultants a quick understanding of the AWS environment. |
| [02-architecture-overview.md](02-architecture-overview.md) | Explain the architecture patterns visible from the discovery data. |
| [03-environment-structure.md](03-environment-structure.md) | Explain account, region, environment, and ownership structure. |
| [04-networking.md](04-networking.md) | Explain the discovered network architecture. |
| [05-security-overview.md](05-security-overview.md) | Summarize visible security posture from discovered configuration. |
| [06-observability.md](06-observability.md) | Summarize logging, monitoring, and operational visibility signals. |
| [07-findings-and-risks.md](07-findings-and-risks.md) | List practical findings, risks, and follow-up items. |
| [08-assumptions-and-unknowns.md](08-assumptions-and-unknowns.md) | Make uncertainty explicit. |

---

## How to Read This Report

**Observed facts** are drawn directly from discovery data — resource properties, tags, relationships, configuration fields, or findings. Every material claim cites a resource identifier, field name, or finding ID.

**Inferred observations** are labelled with cautious language: *"appears to"*, *"suggests"*, or *"based on available evidence"*. They require validation with the customer.

**Assumptions** are stated explicitly with a basis and a validation step. They are not presented as facts.

**"No evidence found for …"** means the discovery data did not contain the relevant signal. It does not mean a control is absent or unnecessary — only that it was not observed in the snapshot.

Resource presence (e.g. a GuardDuty detector, a CloudTrail trail) is noted as the resource existing. Whether the underlying service is active or correctly configured is stated only when a status field in the data confirms it.

---

## Disclaimer

This report is generated solely from AWS discovery data captured at a single point in time. It reflects the state of account `118535197059` at the moment of discovery. It does not represent a continuous audit, a penetration test, or a compliance assessment. Configuration, resources, and risk posture may have changed since the snapshot was taken.
# Architect View — Assumptions & Unknowns

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Assumptions & Unknowns

The architecture picture presented in this view is grounded in graph evidence, but two categories of limitation materially affect how confidently you should act on it: gaps in collector coverage that leave parts of the environment unverified, and open questions about specific resources that require human confirmation. Architects should treat findings in affected areas as **Likely** rather than definitive until the gaps below are closed.

### Assumptions Made

No explicit assumptions were recorded in the discovery package for this section. However, one inference-dependent condition is worth flagging:

- **Environment/stage classification by inference.** 761 resources carry no `Environment`, `Stage`, or `Tier` tag. CloudoX has inferred their classification from naming conventions and graph context rather than authoritative metadata. Any architecture diagram, environment boundary, or workload grouping that relies on these classifications should be treated as approximate until a tagging remediation pass is completed.

### Open Questions

The following gaps and validation questions are unresolved. They are grouped by domain.

#### Coverage Gaps

Two collector-level gaps limit the completeness of the resource inventory:

| Gap | Effect on this view |
|---|---|
| Cloud Control meta-collector was disabled | Long-tail resource types (those without a dedicated typed collector) may be absent from the graph. The full breadth of deployed resources is not confirmed. |
| Resource Explorer meta-collector was disabled or unavailable | AWS-visible resource breadth could not be cross-checked against the typed-collector inventory. Shadow or orphaned resources may not appear. |

Architects relying on this view for dependency mapping or blast-radius analysis should note that the resource graph may be incomplete for less-common service types. (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`, `evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`)

#### Security Gaps

- **No Security Hub enablement discovered.** It is unknown whether AWS Security Hub is active in any account in scope. This means no centralised findings baseline, compliance standard mapping, or cross-account security posture aggregation has been confirmed. Architects designing security architecture or reviewing control coverage should treat this as an open question requiring direct verification. (`evidence_gap:security:no-security-hub-enablement-discovered`)

- **IAM role `cloudox-demo-sandbox-unused-admin` (AROAAAAADPCL3BVEXUDTH) — privilege scope unconfirmed.** This role in account `161388682021` (sandbox) carries what appears to be administrative privileges. It is not confirmed whether this is intentional, who owns it, or whether it is actively used. The validation question is: *Does this role require its current privilege level, and who owns it?* This is flagged as **Assumed** confidence — it warrants explicit owner confirmation before the sandbox environment is treated as controlled. (`validation_question:security:cloudox-demo-sandbox-unused-admin`)

#### Cost Evidence Gaps

Three cost-related gaps affect any spend or sizing analysis derived from this view. They do not directly alter the architecture picture, but they limit the ability to validate right-sizing or identify idle resources:

- **Cost Explorer collection is disabled** (`CLOUDOX_COST__ENABLED=false`). No actual spend figures are available; cost-related observations in this view are architecture-derived only. (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`)
- **CloudWatch utilisation metrics are not collected** in this version. Idle, underutilised, or right-sizing recommendations based on actual usage cannot be made. (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`)
- **RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes** are not captured by current collectors. Cost drivers for these services are not detected and should be assessed separately. (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`)

> **Recommended actions to close these gaps:** Enable Cost Explorer collection, enable the Cloud Control and Resource Explorer meta-collectors on the next discovery run, enforce a tagging policy for Environment/Stage/Tier, verify Security Hub enablement across accounts, and confirm ownership and intent for `cloudox-demo-sandbox-unused-admin`.

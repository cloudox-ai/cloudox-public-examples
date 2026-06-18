# Architect View — Assumptions & Unknowns

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Assumptions & Unknowns

Two collector gaps and one unresolved IAM question materially limit the confidence of this view. Architects should treat any workload classification, cost estimate, or security posture statement as **Likely** rather than Verified until the gaps below are closed.

### Assumptions Made

No explicit assumptions were recorded in the discovery run for this section. However, the following inference-driven conditions shape the entire view and should be treated as working assumptions:

- **Environment classification by inference.** 761 of the 807 discovered resources carry no `Environment`, `Stage`, or `Tier` tag. CloudoX has inferred environment membership (dev / prod / sandbox) from naming conventions and account context rather than authoritative tags. Workload groupings and environment-boundary analysis downstream depend on this inference being correct.
- **Architecture-only cost analysis.** Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`). All cost-related observations in this view are derived from the discovered architecture shape alone — no actual spend figures are available (`evidence_gap:cost:cost-explorer-collection-is-disabled-cloudox-cost-enabled-false-no-collect-costs-spend-figures-are-unavailable-the-analysis-below-is-derived-from-the-discovered-architecture-only`). Treat any cost commentary as directional.
- **Collector coverage is typed, not exhaustive.** The Cloud Control meta-collector was disabled, meaning long-tail or less-common AWS resource types are only visible if a dedicated typed collector exists for them (`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`). Resources outside typed collector scope are silently absent from the graph.

### Open Questions

The following gaps and validation questions remain unresolved and are material to architectural decisions:

#### Discovery Coverage

| Gap | Impact on this view |
|---|---|
| Resource Explorer meta-collector disabled or unavailable | AWS-visible resource breadth could not be cross-checked; undiscovered resources may exist (`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`) |
| Cloud Control meta-collector disabled | Long-tail resource types (e.g. less-common managed services) may be absent from the dependency graph |

#### Security Posture

- **Security Hub not discovered.** No Security Hub enablement was found across any account in scope (`evidence_gap:security:no-security-hub-enablement-discovered`). It is unknown whether Security Hub is absent, disabled, or simply outside collector reach. This means no centralised findings aggregation or compliance standard status can be confirmed from this data.
- **IAM role privilege validation required.** The role `cloudox-demo-sandbox-unused-admin` (id: `AROAAAAADPCL3BVEXUDTH`, account `161388682021`) carries what appears to be administrative privileges. It is unconfirmed whether this scope is intentional and who owns the role (`validation_question:security:cloudox-demo-sandbox-unused-admin`). This is flagged as **Assumed** — the concern is inferred from the role name and privilege level, not from a policy document audit.

#### Cost Drivers

Several cost-relevant dimensions are outside current collector coverage and cannot be assessed (`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`):

- RDS read replicas and provisioned IOPS
- DynamoDB capacity mode (on-demand vs. provisioned)
- Direct Connect
- S3 storage classes

Right-sizing and idle-resource recommendations are also unavailable because CloudWatch utilisation metrics are not collected in this version (`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-idle-underutilized-or-right-sizing-recommendations-based-on-actual-usage-are-not-available`). Any compute or database sizing decisions should be validated against actual CloudWatch data before acting.

#### Recommended Next Steps to Close Gaps

1. Enable Resource Explorer and re-run discovery to cross-check resource breadth.
2. Enable the Cloud Control meta-collector to surface long-tail resource types.
3. Enable Cost Explorer collection (`CLOUDOX_COST__ENABLED=true`) to replace architecture-inferred cost commentary with actuals.
4. Confirm Security Hub enablement status across all accounts — or enable it if absent.
5. Resolve ownership and intended privilege scope of `cloudox-demo-sandbox-unused-admin` with the sandbox account owner.
6. Apply consistent `Environment` / `Stage` / `Tier` tags to the 761 untagged resources to replace inference-based classification.

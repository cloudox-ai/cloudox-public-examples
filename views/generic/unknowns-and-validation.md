# Generic View — Unknowns & Validation Questions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Unknowns & Validation Questions

Several evidence gaps and one open security question shape how much confidence to place in the rest of this view. The gaps are structural — they reflect collector configuration choices, not missing infrastructure — so they are resolvable. The validation question requires a human answer.

> **Confidence: Likely** — The architecture picture is derived from typed collectors; two meta-collectors were disabled, meaning some resource types and the overall inventory breadth could not be independently verified.

### What We Couldn't Confirm

The following gaps are known limitations of this discovery run. None of them indicate a problem in the environment; they indicate places where the analysis is incomplete.

#### Cost data is unavailable

Cost Explorer collection was disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`), so no spend figures exist for this run. All cost-related observations in this view are derived from the discovered architecture — resource patterns known to carry charges — not from actual billing data. [`evidence_gap:cost:cost-explorer-collection-is-disabled-...`]

Additionally, CloudoX does not collect CloudWatch utilization metrics in this version. Idle, underutilized, or right-sizing recommendations based on actual usage are therefore not available. [`evidence_gap:cost:cloudox-does-not-collect-cloudwatch-utilization-metrics-in-this-version-...`]

Several cost-relevant resource attributes are also outside current collector scope: RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured, so cost drivers for those dimensions are not detected. [`evidence_gap:cost:rds-read-replicas-rds-provisioned-iops-dynamodb-capacity-mode-direct-connect-and-s3-storage-classes-are-not-captured-by-the-current-collectors-so-cost-drivers-for-them-are-not-detected`]

**To resolve:** Enable Cost Explorer collection and re-run discovery. For utilization-based recommendations, CloudWatch metric collection will need to be supported in a future version.

#### Inventory breadth could not be cross-checked

Two meta-collectors that would have provided an independent view of all resources in the accounts were not available for this run:

| Meta-collector | Status | Effect |
|---|---|---|
| AWS Cloud Control | Disabled | Long-tail resource types are limited to typed collector coverage only |
| AWS Resource Explorer | Disabled / unavailable | AWS-visible resource breadth could not be cross-checked against what CloudoX discovered |

This means there may be resource types not covered by typed collectors that are absent from this view. The discovered resources are accurate; completeness is uncertain. [`evidence_gap:coverage:cloud-control-meta-collector-was-disabled-long-tail-resource-types-are-limited-to-typed-collector-coverage`] [`evidence_gap:coverage:resource-explorer-meta-collector-was-disabled-or-unavailable-aws-visible-breadth-could-not-be-cross-checked`]

**To resolve:** Enable Cloud Control and/or Resource Explorer meta-collectors on the next run.

#### Tagging gaps affect environment classification

761 resources have no Environment / Stage / Tier tag and rely on inference for classification. Environment assignments for those resources should be treated as approximate.

#### Security Hub enablement

No Security Hub enablement was discovered in this run. Coverage of Security Hub findings is therefore not available in this view.

### Questions to Validate

#### IAM role `cloudox-demo-sandbox-unused-admin` — confirm privilege scope

**Account:** `161388682021` (sandbox) 
**Role ID:** `AROAAAAADPCL3BVEXUDTH`

This role carries a name suggesting administrative privileges and appears to be unused. The open question is whether that privilege level is intentional and whether the role has a current owner.

> **Validation question:** Does IAM role `cloudox-demo-sandbox-unused-admin` require its current privilege level, and who owns it?

This is flagged as **Assumed** confidence — the concern is inferred from the role name and apparent inactivity, not from a policy evaluation. A human with access to the sandbox account should confirm whether the role is still needed, whether its permissions are appropriately scoped, and whether it should be removed or restricted. [`validation_question:security:cloudox-demo-sandbox-unused-admin`]

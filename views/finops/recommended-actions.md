# FinOps View — Recommended Actions

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Recommended Actions

_Cost actions worth taking next._

_Covers: Recommended Cost Actions_

**Assumptions**

- Spend figures use the AWS Cost Explorer UnblendedCost metric for complete billing periods only; the current (partial) month is excluded.
- Architectural cost drivers describe resource patterns known to carry charges; they are not dollar attributions. Exact cost attribution comes from AWS Cost Explorer / CUR.
- Spend data is unavailable for this run; the analysis below is derived from the discovered architecture only.

**Unknowns**

- Cost Explorer collection is disabled (CLOUDOX_COST__ENABLED=false / --no-collect-costs); spend figures are unavailable. The analysis below is derived from the discovered architecture only.
- CloudoX does not collect CloudWatch utilization metrics in this version; idle, underutilized, or right-sizing recommendations based on actual usage are not available.
- RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured by the current collectors, so cost drivers for them are not detected.
- 761 resource(s) have no Environment / Stage / Tier tag and rely on inference for classification.

_This section is shown with CloudoX's deterministic rendering (AI narration unavailable or could not be grounded)._

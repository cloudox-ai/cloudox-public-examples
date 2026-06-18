# FinOps View — Optimization Opportunities

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Likely_

## Optimization Opportunities

> **Confidence: Likely** — Spend data is unavailable for this run (Cost Explorer collection is disabled). All analysis is derived from discovered architecture only; no dollar savings figures can be stated.

The one identified candidate in this section is an **architectural risk item, not a cost-saving opportunity**. There are currently **no optimization candidates with a direct cost-reduction angle** surfaced from the discovered architecture. The absence of spend data and utilization metrics (see Unknowns below) materially limits what can be validated at this time.

### Optimization Candidates

No cost-reduction optimization candidates were identified from the discovered architecture in this run. The summary confirms **0 optimization candidates** across 807 resources in 7 accounts.

One architectural finding was surfaced that carries an indirect cost relevance — unplanned downtime from a single-AZ database can generate incident costs and recovery effort:

| Resource | Finding | Recommended Action | Requires Validation |
|---|---|---|---|
| `cloudox-demo-atlas-prod-pg` | Single-AZ DBInstance — a zone failure could cause downtime or data loss | Evaluate enabling Multi-AZ for resilience | Yes |

*Source: `modernization_opportunity:architecture:cloudox-demo-atlas-prod-pg`*

Enabling Multi-AZ for `cloudox-demo-atlas-prod-pg` would **increase** the RDS line item (standby instance charges apply), but reduces the risk of unplanned recovery costs. This is a resilience trade-off, not a savings action — finance and engineering should evaluate it together against the workload's recovery requirements.

### Estimated Impact

No estimated savings impact can be stated. Spend data is unavailable, and CloudoX does not collect CloudWatch utilization metrics in this version, so idle-resource or right-sizing recommendations based on actual usage are not available.

The following gaps prevent a complete optimization picture and should be resolved before drawing conclusions:

- **Cost Explorer is disabled** (`CLOUDOX_COST__ENABLED=false`). Enabling it would unlock spend-attributed driver analysis and dollar-level opportunity sizing.
- **No utilization metrics** are collected. Idle EC2, underutilized RDS, and right-sizing candidates cannot be detected without CloudWatch data.
- **Untagged resources**: 761 of 807 resources carry no Environment / Stage / Tier tag, making workload-level cost attribution unreliable. Tag coverage is a prerequisite for meaningful FinOps segmentation.
- **Collector gaps**: RDS read replicas, provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes are not captured — cost drivers for these services are not detected in this run.

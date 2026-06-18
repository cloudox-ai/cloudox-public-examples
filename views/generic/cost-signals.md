# Generic View — Cost Signals

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Likely_

## Cost Signals

> **Confidence: Likely** — Spend data is unavailable for this run. All signals below are derived from the discovered architecture; no dollar figures or exact cost attributions are available.

Seven architectural patterns have been identified across this environment that are known to carry AWS charges. No optimization candidates were surfaced from the current discovery pass. Because Cost Explorer collection is disabled, the analysis cannot be tied to actual billing amounts — treat the drivers below as structural indicators of where spend is likely concentrated, not as measured cost breakdowns.

### Cost Drivers

The following accounts are in scope and each contributes to the overall cost surface of the environment:

| Account | ID | Confidence |
|---|---|---|
| Management Account | 110319895932 | Verified |
| Sandbox Ma Account | 161388682021 | Verified |
| Workload Dev Account | 105769365151 | Verified |
| Workload Prod Account | 122122642149 | Verified |
| Log Archive Account | 122980216815 | Likely |
| Audit Account | 110019496666 | Likely |
| Platform Account | 150982215529 | Likely |

Seven architectural cost drivers have been identified from the discovered resources across these accounts. Exact service-level attribution requires AWS Cost Explorer or a Cost and Usage Report (CUR) query; CloudoX has not collected that data in this run.

Note that several common cost driver categories are outside the current collector scope and therefore not detected: RDS read replicas, RDS provisioned IOPS, DynamoDB capacity mode, Direct Connect, and S3 storage classes.

### Optimization Signals

No optimization candidates were identified in this run.

Two capability gaps limit optimization coverage:

- **No spend data** — Cost Explorer collection is disabled (`CLOUDOX_COST__ENABLED=false` / `--no-collect-costs`). Enabling it would allow CloudoX to surface spend-ranked drivers and savings estimates.
- **No utilization metrics** — CloudWatch utilization data is not collected in this version, so idle-resource and right-sizing recommendations based on actual usage are not available.

Enabling both capabilities in a future run would materially improve the signal quality here.

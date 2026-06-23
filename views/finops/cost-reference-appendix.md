# FinOps View — Cost Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Unknown_

## Cost Reference Appendix

This appendix catalogues the accounts and workloads that form the cost boundary for this FinOps view. Use it to cross-reference entities mentioned in other sections against their account IDs and confidence levels. No provider-native billing evidence was available to CloudoX at the time this view was generated.

### Cost Entity Reference

The following accounts and workloads were identified within the `cloudox-demo` workspace. Confidence reflects how firmly each entity was established during discovery.

**Accounts**

| Friendly Name | Account ID | Confidence |
|---|---|---|
| Management Account | 110319895932 | Verified |
| Workload Prod Account | 122122642149 | Verified |
| Workload Dev Account | 105769365151 | Verified |
| Sandbox Ma Account | 161388682021 | Verified |
| Log Archive Account | 122980216815 | Likely |
| Platform Account | 150982215529 | Likely |
| Audit Account | 110019496666 | Likely |

**Workloads**

| Friendly Name | Workload ID | Account | Region | Confidence |
|---|---|---|---|---|
| Cloudox | cloudox | Workload Prod Account (122122642149) | eu-central-1 | Verified |
| Cloudox Demo Atlas Prod API | cloudox-demo-atlas-prod-api | Workload Prod Account (122122642149) | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev | cloudox-demo-atlas-dev | Workload Dev Account (105769365151) | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev API | cloudox-demo-atlas-dev-api | Workload Dev Account (105769365151) | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch | cloudox-demo-sandbox-scratch | Sandbox Ma Account (161388682021) | eu-central-1 | Assumed |

The majority of workload activity is concentrated in `eu-central-1`. Production workloads sit in the Workload Prod Account; development workloads in the Workload Dev Account. The Sandbox Ma Account hosts a scratch workload identified with lower confidence (Assumed) — cost attribution for that workload should be treated cautiously until confirmed.

### Evidence

No provider-native cost evidence (e.g. billing exports, Cost Explorer data, or invoice line items) was available to CloudoX for this section. Cost figures, spend breakdowns, and savings estimates cited elsewhere in this view are derived from resource-level analysis rather than direct billing data. Finance teams should validate any cost totals against authoritative billing sources before acting on them.

# FinOps View — Cost Reference Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [FinOps View](./README.md) · Audience: FinOps / Finance · Confidence: Unknown_

## Cost Reference Appendix

This appendix catalogues the accounts and workloads that form the cost boundary for this FinOps view. Use it to cross-reference entities mentioned elsewhere in the view against their account IDs and confidence levels. No provider-native cost evidence (billing exports, Cost Explorer data, or equivalent) was available to CloudoX at the time this view was generated — see the Evidence sub-section below.

### Cost Entity Reference

The following accounts and workloads were identified within the `cloudox-demo` workspace. Confidence reflects how firmly each entity's identity and boundary have been established.

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

> **Confidence note:** Entities marked *Likely* have been inferred with reasonable confidence but have not been fully verified. The *Assumed* entry (Cloudox Demo Sandbox Scratch) should be treated as provisional — validate its account association and cost attribution before acting on any figures tied to it.

### Evidence

No provider-native cost evidence was available to CloudoX for this section. No billing records, cost allocation tags, Cost Explorer exports, or equivalent data sources were present in the package. As a result:

- Spend figures, cost breakdowns, and savings estimates **cannot be derived** from this appendix alone.
- The entity reference above is suitable for use as a lookup table when reconciling account IDs with workload names, but it does not carry monetary values.

**Recommended action (requires validation):** Confirm that Cost and Usage Report (CUR) or equivalent billing data is being exported and is accessible to CloudoX. Once cost data is ingested, this appendix will be populated with per-account and per-workload spend context to support the cost driver and optimization sections of this view.

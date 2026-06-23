> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Identity and Trust Reference

IAM role usage, assumption, and policy attachment relationships. Cross-account rows are INFERRED from endpoint account ids (labelled Assumed), not read from trust policies.

_Displaying all **8** row(s) (no rows omitted)._

_Sorted cross-account first, then principal (cross-account is review-worthy)._

| Account | Principal | Trust Type | Role / Policy | Access Scope | Confidence | Evidence |
|---|---|---|---|---|---|---|
| 105769365151 | cloudox-demo-atlas-dev-api | uses IAM role | cloudox-demo-atlas-dev-lambda (AROAAAAACJ2F32IXZJ88A) | same-account | Verified | `cloudox-demo-atlas-dev-api`, `AROAAAAACJ2F32IXZJ88A` |
| 105769365151 | cloudox-demo-atlas-dev-api:1 | uses IAM role | cloudox-demo-atlas-dev-ecs-task (AROAAAAADLX1PNSXHC8KC) | same-account | Verified | `cloudox-demo-atlas-dev-api:1`, `AROAAAAADLX1PNSXHC8KC` |
| 105769365151 | cloudox-demo-atlas-dev-api:1 | uses IAM role | cloudox-demo-atlas-dev-ecs-exec (AROAAAAADLCZISR8G4FBU) | same-account | Verified | `cloudox-demo-atlas-dev-api:1`, `AROAAAAADLCZISR8G4FBU` |
| 122122642149 | cloudox-demo-atlas-prod-api | uses IAM role | cloudox-demo-atlas-prod-lambda (AROAAAAADD422V3IH190S) | same-account | Verified | `cloudox-demo-atlas-prod-api`, `AROAAAAADD422V3IH190S` |
| 122122642149 | cloudox-demo-atlas-prod-api:2 | uses IAM role | cloudox-demo-atlas-prod-ecs-task (AROAAAAAA12PWSX5SK0XF) | same-account | Verified | `cloudox-demo-atlas-prod-api:2`, `AROAAAAAA12PWSX5SK0XF` |
| 122122642149 | cloudox-demo-atlas-prod-api:2 | uses IAM role | cloudox-demo-atlas-prod-ecs-exec (AROAAAAABKVS2YU8FPYNK) | same-account | Verified | `cloudox-demo-atlas-prod-api:2`, `AROAAAAABKVS2YU8FPYNK` |
| 161388682021 | cloudox-demo-sandbox-scratch | uses IAM role | cloudox-demo-sandbox-scratch-lambda (AROAAAAADE03UVM8VCPAN) | same-account | Verified | `cloudox-demo-sandbox-scratch`, `AROAAAAADE03UVM8VCPAN` |
| 161388682021 | cloudox-demo-sandbox-unused-admin | assumes role | cloudox-demo-sandbox-unused-admin (AROAAAAADPCL3BVEXUDTH) | same-account | Verified | `cloudox-demo-sandbox-unused-admin`, `AROAAAAADPCL3BVEXUDTH` |

**Coverage notes**

- Role trust policies (AssumeRolePolicyDocument) and Identity Center / SSO assignments are not collected. Cross-account trust is therefore INFERRED from the accounts of the linked resources (low confidence: Assumed) — it is not confirmed IAM trust and may be incomplete or over-stated. Verify against the actual role trust policy.

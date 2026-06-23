> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Identity and Trust Reference

IAM role usage, assumption, and policy-attachment relationships. Confirmed trust-policy analysis is UNAVAILABLE (role trust policies are not collected). Cross-account trust is shown only for assume-role edges and is INFERRED from endpoint account ids (labelled Assumed) — policy attachments and role usage are never classified as cross-account trust.

_Displaying all **8** row(s) (no rows omitted)._

_Sorted inferred cross-account assume-role first, then principal (cross-account assumption is review-worthy)._

| Account | Principal | Relationship | Role / Policy | Access Scope | Confidence | Evidence |
|---|---|---|---|---|---|---|
| 105769365151 | cloudox-demo-atlas-dev-api | uses IAM role | cloudox-demo-atlas-dev-lambda (AROAAAAACJ2F32IXZJ88A) | attachment (not a trust relationship) | Verified | `cloudox-demo-atlas-dev-api`, `AROAAAAACJ2F32IXZJ88A` |
| 105769365151 | cloudox-demo-atlas-dev-api:1 | uses IAM role | cloudox-demo-atlas-dev-ecs-task (AROAAAAADLX1PNSXHC8KC) | attachment (not a trust relationship) | Verified | `cloudox-demo-atlas-dev-api:1`, `AROAAAAADLX1PNSXHC8KC` |
| 105769365151 | cloudox-demo-atlas-dev-api:1 | uses IAM role | cloudox-demo-atlas-dev-ecs-exec (AROAAAAADLCZISR8G4FBU) | attachment (not a trust relationship) | Verified | `cloudox-demo-atlas-dev-api:1`, `AROAAAAADLCZISR8G4FBU` |
| 122122642149 | cloudox-demo-atlas-prod-api | uses IAM role | cloudox-demo-atlas-prod-lambda (AROAAAAADD422V3IH190S) | attachment (not a trust relationship) | Verified | `cloudox-demo-atlas-prod-api`, `AROAAAAADD422V3IH190S` |
| 122122642149 | cloudox-demo-atlas-prod-api:2 | uses IAM role | cloudox-demo-atlas-prod-ecs-task (AROAAAAAA12PWSX5SK0XF) | attachment (not a trust relationship) | Verified | `cloudox-demo-atlas-prod-api:2`, `AROAAAAAA12PWSX5SK0XF` |
| 122122642149 | cloudox-demo-atlas-prod-api:2 | uses IAM role | cloudox-demo-atlas-prod-ecs-exec (AROAAAAABKVS2YU8FPYNK) | attachment (not a trust relationship) | Verified | `cloudox-demo-atlas-prod-api:2`, `AROAAAAABKVS2YU8FPYNK` |
| 161388682021 | cloudox-demo-sandbox-scratch | uses IAM role | cloudox-demo-sandbox-scratch-lambda (AROAAAAADE03UVM8VCPAN) | attachment (not a trust relationship) | Verified | `cloudox-demo-sandbox-scratch`, `AROAAAAADE03UVM8VCPAN` |
| 161388682021 | cloudox-demo-sandbox-unused-admin | assumes role | cloudox-demo-sandbox-unused-admin (AROAAAAADPCL3BVEXUDTH) | same-account | Verified | `cloudox-demo-sandbox-unused-admin`, `AROAAAAADPCL3BVEXUDTH` |

**Coverage notes**

- Confirmed trust-policy analysis is unavailable: role trust policies (AssumeRolePolicyDocument) and Identity Center / SSO assignments are not collected. Only assume-role edges already in the graph are treated as (inferred) cross-account trust; verify each against the actual role trust policy.
- Policy attachments (`has policy`) and role usage (`uses IAM role`) are not trust relationships and are never classified cross-account — AWS-managed policy resources carry an arbitrary discovery-account context that would otherwise produce false cross-account rows.

# Executive View — Evidence Appendix

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Unknown_

## Evidence Appendix

This appendix identifies the accounts and workloads that underpin the analysis in this view. No raw evidence items were surfaced for this section.

### Key Entities

The following cloud accounts and workloads were identified as in-scope for this view.

| Friendly Name | Type | Region | Confidence |
|---|---|---|---|
| Management Account | Account | — | Verified |
| Sandbox Ma Account | Account | — | Verified |
| Workload Dev Account | Account | — | Verified |
| Workload Prod Account | Account | — | Verified |
| Log Archive Account | Account | — | Likely |
| Cloudox Demo Atlas Prod API | Workload | eu-central-1 | Likely |

Two entities carry a **Likely** confidence rating — Log Archive Account and the Cloudox Demo Atlas Prod API workload — meaning their classification or scope boundary has not been fully verified. Decisions that depend on the completeness of log archiving or the production API workload boundary should account for this uncertainty.

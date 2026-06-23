# Knowledge Quality Intelligence

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

_Deterministic evaluation of whether each Knowledge View delivers useful cloud understanding to its audience. The scores below are the authoritative quality scores._

**Overall quality:** 90.9/100 (grade A)

- **Strongest view:** executive
- **Weakest view:** operations
- **Biggest issue:** FinOps → Recommended Actions declares recommendation but the environment produced none to surface.

## View scorecards

| View | Score | Grade | Persona coverage | Findings |
| --- | ---: | :---: | ---: | ---: |
| Architect | 90.6 | A | 100% | 0 |
| Executive | 98.1 | A | 100% | 0 |
| FinOps | 90.7 | A | 100% | 2 |
| Generic | 91.2 | A | 100% | 0 |
| Operations | 87.0 | B | 100% | 1 |
| Security | 87.8 | B | 100% | 1 |

## Architect View — 90.6/100 (A)

_Audience: Solutions / Cloud Architects_

| Dimension | Score | Detail |
| --- | ---: | --- |
| persona coverage | 100 | 3/3 persona questions answered |
| hollow sections | 100 | 0/9 content-bearing sections are hollow |
| redundancy | 100 | 0 redundant unit(s) of 30 |
| signal to noise | 47 | signal-to-noise ratio 0.47 |
| actionability | 100 | 21/21 surfaced items are actionable |
| prioritization | 90 | 18/20 adjacent pairs correctly ordered |
| intelligence coverage | 100 | 3/3 expected intelligence kinds surfaced |

**Persona questions**

- ✓ What design issues exist?
- ✓ What dependencies matter?
- ✓ What modernization opportunities exist?

## Executive View — 98.1/100 (A)

_Audience: CTO / Engineering leadership_

| Dimension | Score | Detail |
| --- | ---: | --- |
| persona coverage | 100 | 3/3 persona questions answered |
| hollow sections | 100 | 0/8 content-bearing sections are hollow |
| redundancy | 100 | 0 redundant unit(s) of 20 |
| signal to noise | 94 | signal-to-noise ratio 0.94 |
| actionability | 100 | 12/12 surfaced items are actionable |
| prioritization | 91 | 10/11 adjacent pairs correctly ordered |
| intelligence coverage | 100 | 3/3 expected intelligence kinds surfaced |

**Persona questions**

- ✓ What requires leadership attention?
- ✓ What are the highest risks?
- ✓ What decisions are needed?

## FinOps View — 90.7/100 (A)

_Audience: FinOps / Finance_

| Dimension | Score | Detail |
| --- | ---: | --- |
| persona coverage | 100 | 3/3 persona questions answered |
| hollow sections | 75 | 2/8 content-bearing sections are hollow |
| redundancy | 100 | 0 redundant unit(s) of 15 |
| signal to noise | 74 | signal-to-noise ratio 0.74 |
| actionability | 100 | 10/10 surfaced items are actionable |
| prioritization | 89 | 8/9 adjacent pairs correctly ordered |
| intelligence coverage | 100 | 1/1 expected intelligence kinds surfaced |

**Persona questions**

- ✓ What is driving cost?
- ✓ What optimization is worth validating?
- ✓ Where is the waste?

## Generic View — 91.2/100 (A)

_Audience: Any technical reader_

| Dimension | Score | Detail |
| --- | ---: | --- |
| persona coverage | 100 | 3/3 persona questions answered |
| hollow sections | 100 | 0/9 content-bearing sections are hollow |
| redundancy | 100 | 0 redundant unit(s) of 24 |
| signal to noise | 50 | signal-to-noise ratio 0.50 |
| actionability | 100 | 14/14 surfaced items are actionable |
| prioritization | 92 | 12/13 adjacent pairs correctly ordered |
| intelligence coverage | 100 | 3/3 expected intelligence kinds surfaced |

**Persona questions**

- ✓ What is this environment and what runs in it?
- ✓ How is it structured, connected, and secured?
- ✓ What is uncertain or unknown?

## Operations View — 87.0/100 (B)

_Audience: Platform / Operations Engineers_

| Dimension | Score | Detail |
| --- | ---: | --- |
| persona coverage | 100 | 3/3 persona questions answered |
| hollow sections | 89 | 1/9 content-bearing sections are hollow |
| redundancy | 100 | 0 redundant unit(s) of 20 |
| signal to noise | 41 | signal-to-noise ratio 0.41 |
| actionability | 100 | 12/12 surfaced items are actionable |
| prioritization | 82 | 9/11 adjacent pairs correctly ordered |
| intelligence coverage | 100 | 2/2 expected intelligence kinds surfaced |

**Persona questions**

- ✓ What can break?
- ✓ What requires operational attention?
- ✓ What recovery risks exist?

## Security View — 87.8/100 (B)

_Audience: Security & Governance teams_

| Dimension | Score | Detail |
| --- | ---: | --- |
| persona coverage | 100 | 3/3 persona questions answered |
| hollow sections | 88 | 1/8 content-bearing sections are hollow |
| redundancy | 100 | 0 redundant unit(s) of 25 |
| signal to noise | 46 | signal-to-noise ratio 0.46 |
| actionability | 100 | 20/20 surfaced items are actionable |
| prioritization | 84 | 16/19 adjacent pairs correctly ordered |
| intelligence coverage | 100 | 4/4 expected intelligence kinds surfaced |

**Persona questions**

- ✓ What is exposed, and to whom?
- ✓ Where is identity / access over-privileged or unclear?
- ✓ What governance or evidence gaps exist?

## Quality findings

- **[info] Hollow section: Recommended Actions** (finops/recommended-actions) — FinOps → Recommended Actions declares recommendation but the environment produced none to surface.
  - _Recommendation:_ Populate the section, broaden its declared kinds/domains, or drop it from the view so the report has no empty shells.
- **[info] Hollow section: Waste / Utilization Signals** (finops/waste-and-utilization) — FinOps → Waste / Utilization Signals declares operational_concern but the environment produced none to surface.
  - _Recommendation:_ Populate the section, broaden its declared kinds/domains, or drop it from the view so the report has no empty shells.
- **[info] Hollow section: Failure Points** (operations/failure-points) — Operations → Failure Points declares operational_concern but the environment produced none to surface.
  - _Recommendation:_ Populate the section, broaden its declared kinds/domains, or drop it from the view so the report has no empty shells.
- **[info] Hollow section: Security Findings** (security/security-findings) — Security → Security Findings declares finding, operational_concern, recommendation but the environment produced none to surface.
  - _Recommendation:_ Populate the section, broaden its declared kinds/domains, or drop it from the view so the report has no empty shells.

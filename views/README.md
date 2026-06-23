# Knowledge Views

> **Demo Report** — This discovery report was produced from a real AWS environment by
> [CloudoX](https://cloudox.io). AWS account IDs and all resource identifiers (VPC,
> subnet, security group, IAM role IDs, KMS key IDs, and similar) have been replaced
> with deterministic synthetic equivalents so the report can be shared publicly.
> Architecture patterns, workload structure, findings, and service configurations are
> otherwise unmodified — this is not dummy data.

---

This environment is presented as **audience lenses** over one shared understanding. Every view draws on the same Canonical Interpretation, Presentation (friendly names + confidence), and Knowledge View artifacts — different explanations and priorities, **never different truths**.

Pick the view that matches your question:

| View | Audience | Confidence | Sections |
| --- | --- | --- | --- |
| [Architect](./architect/README.md) | Solutions / Cloud Architects | Likely | 9 |
| [Executive](./executive/README.md) | CTO / Engineering leadership | Likely | 8 |
| [FinOps](./finops/README.md) | FinOps / Finance | Likely | 8 |
| [Generic](./generic/README.md) | Any technical reader | Likely | 9 |
| [Operations](./operations/README.md) | Platform / Operations Engineers | Verified | 9 |
| [Security](./security/README.md) | Security & Governance teams | Likely | 8 |

_Navigation for these views is in `navigation.json` (view-aware: each view lists its own section structure)._

## Reference & deterministic layers

Beyond the narrated views, this report ships **deterministic layers** built straight from the knowledge graph (no AI narration) — facts you can audit directly:

| Layer | What it gives you |
| --- | --- |
| [Reference](../reference/README.md) | Inventory & relationship tables straight from the knowledge graph: network routing, connectivity, security flows, public exposure, service dependencies, identity & trust, DNS, encryption, and tagging coverage. |
| [Cost Intelligence](../cost-intelligence.md) | Spend explained in architectural context — graph-grounded cost drivers and optimization candidates (not a billing export). |
| [Knowledge Quality](../knowledge-quality.md) | How well each view answers its audience's questions — deterministic, per-view quality scoring. |

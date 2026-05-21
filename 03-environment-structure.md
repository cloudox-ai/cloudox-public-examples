# 03 — Environment Structure

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

## Account Inventory

Seven accounts are active within a single AWS Organization (`o-aaaapzvebq`). The management account carries the alias `CloudoX` (`110319895932`). All other accounts were created (not invited) and are distributed across four OUs.

| Account ID | Alias | OU Path | Joined |
|---|---|---|---|
| `110319895932` | CloudoX | `/` (Root) | 2026-05-19 |
| `110019496666` | audit | `/Security` | 2026-05-19 |
| `122980216815` | log-archive | `/Security` | 2026-05-19 |
| `122122642149` | workload-prod | `/Workloads` | 2026-05-19 |
| `105769365151` | workload-dev | `/Workloads` | 2026-05-19 |
| `150982215529` | platform-prod | `/Platform` | 2026-05-19 |
| `161388682021` | sandbox-ma | `/Sandbox` | 2026-05-19 |

All seven accounts share the same join date (2026-05-19), which suggests the organization structure was provisioned in a single automated operation. This should be validated with the customer.

The organization has `feature_set: ALL` enabled (source: organization tag on `o-aaaapzvebq`), indicating AWS Organizations service control policies and consolidated billing are available.

---

## Organizational Unit Structure

Four OUs are present beneath the Root:

| OU | Accounts |
|---|---|
| `/Security` | `audit` (`110019496666`), `log-archive` (`122980216815`) |
| `/Workloads` | `workload-prod` (`122122642149`), `workload-dev` (`105769365151`) |
| `/Platform` | `platform-prod` (`150982215529`) |
| `/Sandbox` | `sandbox-ma` (`161388682021`) |

Five Organizations policies are present in the graph (`AWS::Organizations::Policy` count: 5). Their content, targets, and enforcement status are not available in the provided discovery data.

---

## Environment Classification

| Account ID | Alias | Classification | Confidence | Explicit |
|---|---|---|---|---|
| `110319895932` | CloudoX | unknown | low | No |
| `110019496666` | audit | unknown | low | No |
| `122980216815` | log-archive | unknown | low | No |
| `122122642149` | workload-prod | **production** | high | **Yes** |
| `105769365151` | workload-dev | **development** | high | **Yes** |
| `150982215529` | platform-prod | production | medium | No |
| `161388682021` | sandbox-ma | **sandbox** | high | **Yes** |

**Explicit classifications** (`explicit: true`) are present for three accounts: `workload-prod` (production), `workload-dev` (development), and `sandbox-ma` (sandbox). These classifications were directly asserted in the discovery data, not inferred.

`platform-prod` (`150982215529`) is classified as production at medium confidence without an explicit flag — this appears to be inferred, likely from the account alias. It should be confirmed with the customer.

The management account (`110319895932`) and both Security OU accounts (`audit`, `log-archive`) carry `classification: unknown`. Their functional roles are strongly suggested by their aliases and OU placement, but no explicit environment tag or classification signal was detected for these three accounts.

---

## Region Coverage

`region_count: 0` is recorded in the graph stats, meaning no regions were explicitly enumerated at the account level. However, all five detected workloads are anchored to `eu-central-1` (source: workload records for `cloudox-demo-atlas-dev`, `cloudox-demo-atlas-prod`, `cloudox-demo-atlas-dev-api`, `cloudox-demo-atlas-prod-api`, `cloudox-demo-sandbox-scratch`). This suggests `eu-central-1` is the active deployment region, but this should be validated — other regions may host resources not captured in the workload anchors.

No evidence was found for multi-region deployment in the provided discovery data.

---

## Workload and Ownership Signals

Five workloads were inferred from resource anchors. All are in `eu-central-1`. None are flagged as internet-facing in the workload records (though `cloudox-demo-atlas-prod` includes an ALB resource `cloudox-demo-atlas-prod-alb`, which warrants validation — see `04-networking.md`).

| Workload Name | Account | Confidence | Anchor Resource Type |
|---|---|---|---|
| `cloudox-demo-atlas-prod` | `workload-prod` (`122122642149`) | medium | `AWS::ECS::Cluster` |
| `cloudox-demo-atlas-prod-api` | `workload-prod` (`122122642149`) | medium | `AWS::Lambda::Function` |
| `cloudox-demo-atlas-dev` | `workload-dev` (`105769365151`) | medium | `AWS::ECS::Cluster` |
| `cloudox-demo-atlas-dev-api` | `workload-dev` (`105769365151`) | medium | `AWS::Lambda::Function` |
| `cloudox-demo-sandbox-scratch` | `sandbox-ma` (`161388682021`) | low | `AWS::Lambda::Function` |

The `cloudox-demo-atlas` prefix appears across both the dev and prod workload accounts, suggesting a shared application or product line. This is an inference based on naming convention only and should be validated with the customer. No ownership tags (e.g. `Owner`, `Project`, `CostCenter`, `Team`) were detected on these workloads in the provided discovery data.

No workloads were detected in `platform-prod` (`150982215529`), `audit` (`110019496666`), `log-archive` (`122980216815`), or the management account (`110319895932`).

---

## Tagging Coverage

The only tags observed in the discovery data are OU-level tags (`ou_name`, `ou_path`) on account records. No resource-level tags for `Environment`, `Owner`, `Project`, `CostCenter`, or lifecycle indicators were present in the provided data.

This does not confirm that resource tagging is absent — the discovery context for this section was filtered to account and environment entities. Tagging coverage at the resource level should be assessed from the full resource inventory.

---

## Assumptions

| Assumption | Basis | How to Validate |
|---|---|---|
| The organization was provisioned via automation in a single operation | All 7 accounts share join date 2026-05-19 | Confirm with the customer whether a landing zone tool (e.g. Control Tower, custom IaC) was used |
| `platform-prod` (`150982215529`) is a production account | Account alias and OU path `/Platform`; classification confidence is medium, `explicit: false` | Confirm environment classification and workload purpose with the customer |
| `eu-central-1` is the sole active region | All workload anchors reference `eu-central-1`; `region_count: 0` in graph stats | Ask the customer to confirm whether any workloads or data residency requirements exist in other regions |

---

## Unknowns / Gaps

- **Environment classification for management and Security OU accounts**: `110319895932`, `110019496666`, and `122980216815` all carry `classification: unknown`. Their roles are suggested by aliases and OU placement but are not confirmed.
- **Organizations policy content and targets**: Five `AWS::Organizations::Policy` resources exist; their type (SCP, tag policy, backup policy), targets, and enforcement status are not available in the provided data.
- **Resource-level tagging**: No `Owner`, `Project`, `CostCenter`, or `Environment` tags were observed on any resource in this section's data. Whether a tagging strategy exists and is enforced cannot be determined from the available evidence.
- **Platform account workloads**: No workloads were detected in `platform-prod` (`150982215529`). The account's purpose and hosted resources are unknown from the provided data.
- **Multi-region posture**: Region-level discovery returned zero explicit regions. Whether resources exist outside `eu-central-1` is unknown.

---

## Follow-up Questions

1. Was the organization provisioned using AWS Control Tower or a custom landing zone? If so, what version and customisations are in place?
2. What is the intended purpose of `platform-prod` (`150982215529`)? What services or shared infrastructure does it host?
3. Are the five Organizations policies SCPs, tag policies, or backup policies — and which OUs or accounts are they attached to?
4. Is `eu-central-1` the only active region, or are there workloads, data stores, or DR resources in other regions?
5. Is there a mandatory tagging policy in place (e.g. via SCP or AWS Config rule)? If so, which tags are required and how is compliance tracked?
6. Who owns the `cloudox-demo-atlas` workload — is this a single product team or multiple teams sharing a naming prefix?
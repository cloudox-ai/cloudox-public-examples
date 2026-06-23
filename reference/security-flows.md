> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Security Flow Reference (Ingress Only)

Inbound security-group rules as source → destination flows, riskiest first. v1 covers INGRESS ONLY — egress rules and network ACLs are out of scope, so this is not a complete traffic-control picture.

_Displaying all **24** row(s) (no rows omitted)._

_Sorted riskiest first: world-open (non-web) → world-open (web) → scoped._

| Source | Destination (Security Group) | Protocol | Port Range | Direction | Scope | Risk Indicator | Confidence | Evidence |
|---|---|---|---|---|---|---|---|---|
| 0.0.0.0/0 | cloudox-demo-sandbox-sg-permissive (sg-054446d655de1ee7f) | tcp | 5432 | Ingress | world (IPv4) | world-open (non-web) | Verified | `sg-054446d655de1ee7f` |
| 0.0.0.0/0 | cloudox-demo-sandbox-sg-permissive (sg-054446d655de1ee7f) | tcp | 22 | Ingress | world (IPv4) | world-open (non-web) | Verified | `sg-054446d655de1ee7f` |
| 0.0.0.0/0 | cloudox-demo-atlas-prod-sg-alb (sg-06f2b4190bf01d261) | tcp | 80 | Ingress | world (IPv4) | world-open (web) | Verified | `sg-06f2b4190bf01d261` |
| 0.0.0.0/0 | cloudox-demo-atlas-prod-sg-edge (sg-0459201826f8de5b3) | tcp | 443 | Ingress | world (IPv4) | world-open (web) | Verified | `sg-0459201826f8de5b3` |
| 0.0.0.0/0 | cloudox-demo-sandbox-sg-permissive (sg-054446d655de1ee7f) | tcp | 80 | Ingress | world (IPv4) | world-open (web) | Verified | `sg-054446d655de1ee7f` |
| 10.30.0.0/16 | cloudox-demo-atlas-dev-sg-app (sg-0d82037d4b797ae50) | tcp | 443 | Ingress | CIDR | scoped | Verified | `sg-0d82037d4b797ae50` |
| sg-0459201826f8de5b3 | cloudox-demo-atlas-prod-sg-app (sg-016daa1c9dc89e0de) | tcp | 443 | Ingress | security group | scoped | Verified | `sg-016daa1c9dc89e0de` |
| sg-06f2b4190bf01d261 | cloudox-demo-atlas-prod-sg-ecs (sg-0457a284b8b9ead6c) | tcp | 80 | Ingress | security group | scoped | Verified | `sg-0457a284b8b9ead6c` |
| sg-0457a284b8b9ead6c | cloudox-demo-atlas-prod-sg-rds (sg-0831f3eae117601d2) | tcp | 5432 | Ingress | security group | scoped | Verified | `sg-0831f3eae117601d2` |
| sg-04cc29b734fbe1d80 | default (sg-04cc29b734fbe1d80) | all | all | Ingress | security group | scoped | Verified | `sg-04cc29b734fbe1d80` |
| sg-0857d616f209fa91d | default (sg-0857d616f209fa91d) | all | all | Ingress | security group | scoped | Verified | `sg-0857d616f209fa91d` |
| sg-0ab7bc6fb6ff0b0eb | default (sg-0ab7bc6fb6ff0b0eb) | all | all | Ingress | security group | scoped | Verified | `sg-0ab7bc6fb6ff0b0eb` |
| sg-0c30cd6f679657f7a | default (sg-0c30cd6f679657f7a) | all | all | Ingress | security group | scoped | Verified | `sg-0c30cd6f679657f7a` |
| sg-0ac5339a7aa87a30c | default (sg-0ac5339a7aa87a30c) | all | all | Ingress | security group | scoped | Verified | `sg-0ac5339a7aa87a30c` |
| sg-027456f17ebd7a420 | default (sg-027456f17ebd7a420) | all | all | Ingress | security group | scoped | Verified | `sg-027456f17ebd7a420` |
| sg-070d1fc97b5f6e04d | default (sg-070d1fc97b5f6e04d) | all | all | Ingress | security group | scoped | Verified | `sg-070d1fc97b5f6e04d` |
| sg-0df85b141d7cd1fc4 | default (sg-0df85b141d7cd1fc4) | all | all | Ingress | security group | scoped | Verified | `sg-0df85b141d7cd1fc4` |
| sg-0607bb1d84d622274 | default (sg-0607bb1d84d622274) | all | all | Ingress | security group | scoped | Verified | `sg-0607bb1d84d622274` |
| sg-09545ae92c853734f | default (sg-09545ae92c853734f) | all | all | Ingress | security group | scoped | Verified | `sg-09545ae92c853734f` |
| sg-00106a6e6dcbea237 | default (sg-00106a6e6dcbea237) | all | all | Ingress | security group | scoped | Verified | `sg-00106a6e6dcbea237` |
| sg-06ed7a1adcea4d9e1 | default (sg-06ed7a1adcea4d9e1) | all | all | Ingress | security group | scoped | Verified | `sg-06ed7a1adcea4d9e1` |
| sg-03c6d617c13e918db | default (sg-03c6d617c13e918db) | all | all | Ingress | security group | scoped | Verified | `sg-03c6d617c13e918db` |
| sg-0b0640c3b06110b6a | default (sg-0b0640c3b06110b6a) | all | all | Ingress | security group | scoped | Verified | `sg-0b0640c3b06110b6a` |
| sg-09ba82cb0827434e3 | default (sg-09ba82cb0827434e3) | all | all | Ingress | security group | scoped | Verified | `sg-09ba82cb0827434e3` |

**Coverage notes**

- Ingress only: egress (outbound) security-group rules are not analysed in v1, so a flow's absence here says nothing about outbound access.
- Network ACLs are not included; effective access also depends on the subnet NACL, which this table does not evaluate.

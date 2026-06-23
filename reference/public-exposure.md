> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Public Exposure Reference

Resources reachable from the internet, with the exposure path.

_Displaying all **9** row(s) (no rows omitted)._

_Sorted by account, then resource (stable, not by severity)._

| Account | Region | Resource | Resource Type | Exposure Type | Risk Notes | Confidence | Evidence |
|---|---|---|---|---|---|---|---|
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-public-a (subnet-065f522206524ab12) | AWS::EC2::Subnet | Public subnet | Subnet route table forwards traffic to an Internet Gateway (public subnet). | Verified | `internet`, `subnet-065f522206524ab12` |
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-public-b (subnet-0f64d71c952a7898a) | AWS::EC2::Subnet | Public subnet | Subnet route table forwards traffic to an Internet Gateway (public subnet). | Verified | `internet`, `subnet-0f64d71c952a7898a` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-alb | AWS::ElasticLoadBalancingV2::LoadBalancer | Internet-facing load balancer | Load balancer scheme is internet-facing (cloudox-demo-atlas-prod-alb). | Verified | `internet`, `cloudox-demo-atlas-prod-alb` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-public-a (subnet-013a24d318ed6f3d0) | AWS::EC2::Subnet | Public subnet | Subnet route table forwards traffic to an Internet Gateway (public subnet). | Verified | `internet`, `subnet-013a24d318ed6f3d0` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-public-b (subnet-016c22941a019a137) | AWS::EC2::Subnet | Public subnet | Subnet route table forwards traffic to an Internet Gateway (public subnet). | Verified | `internet`, `subnet-016c22941a019a137` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-sg-alb (sg-06f2b4190bf01d261) | AWS::EC2::SecurityGroup | Open security group | Security group cloudox-demo-atlas-prod-sg-alb has ingress rules allowing 0.0.0.0/0 or ::/0. | Verified | `internet`, `sg-06f2b4190bf01d261` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-sg-edge (sg-0459201826f8de5b3) | AWS::EC2::SecurityGroup | Open security group | Security group cloudox-demo-atlas-prod-sg-edge has ingress rules allowing 0.0.0.0/0 or ::/0. | Verified | `internet`, `sg-0459201826f8de5b3` |
| 161388682021 | eu-central-1 | cloudox-demo-sandbox-public (subnet-029e2cceb3d0beff7) | AWS::EC2::Subnet | Public subnet | Subnet route table forwards traffic to an Internet Gateway (public subnet). | Verified | `internet`, `subnet-029e2cceb3d0beff7` |
| 161388682021 | eu-central-1 | cloudox-demo-sandbox-sg-permissive (sg-054446d655de1ee7f) | AWS::EC2::SecurityGroup | Open security group | Security group cloudox-demo-sandbox-sg-permissive has ingress rules allowing 0.0.0.0/0 or ::/0. | Verified | `internet`, `sg-054446d655de1ee7f` |

**Coverage notes**

- Exact entry-point port/protocol is not modelled on exposure edges; see the Security Flow Reference for open ingress ports.

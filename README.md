# CloudoX — Example Cloud Discovery Report

This repository is a **public, sanitized example** of a cloud discovery report produced by [CloudoX](https://cloudox.io) — an intelligent cloud knowledge platform that turns a real cloud environment into evidence-grounded, stakeholder-ready understanding.

> **About this example** — It was generated from a **real AWS environment** and then sanitized for public sharing: every account ID, organization ID, and resource identifier (VPC, subnet, security group, IAM role, KMS key, …) — including the identifiers drawn inside the architecture **diagrams** — has been replaced with deterministic synthetic equivalents. Architecture patterns, workload structure, findings, and service configurations are otherwise unmodified — **this is not dummy data**.

## How to read it

CloudoX presents one environment as **audience lenses** ("Knowledge Views") over a single shared understanding — different explanations and priorities for different readers, **never different truths**.

Start with the **[Knowledge Views index](./views/README.md)**, then open the view that matches your question:

| View | Best for |
| --- | --- |
| [Generic](./views/generic/README.md) | Any technical reader |
| [Executive](./views/executive/README.md) | CTO / Engineering leadership |
| [Architect](./views/architect/README.md) | Solutions / Cloud Architects |
| [Operations](./views/operations/README.md) | Platform / Operations Engineers |
| [Security](./views/security/README.md) | Security & Governance teams |
| [FinOps](./views/finops/README.md) | FinOps / Finance |

Each view is a self-contained folder: its `README.md` is the composed view, every section is also a standalone Markdown file, and `diagrams/` holds the rendered architecture diagrams.

## About CloudoX

CloudoX discovers a cloud environment, builds a typed knowledge graph of what exists and how it connects, interprets it into architecture, networking, security, and cost meaning, and narrates that understanding for each audience — so a team can understand an unfamiliar environment in minutes instead of weeks.

Learn more at **[cloudox.io](https://cloudox.io)**.

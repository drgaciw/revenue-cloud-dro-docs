---
title: "Salesforce RCA/DRO Agent Engineering Docs"
description: "Entry point for humans and coding agents using this 21-file RCA/DRO documentation set."
agent_use: "Start here in a new session; then follow the load order and linked topic documents."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator", "Salesforce Platform"]
related: ["index", "agentic-skills-inventory", "interface-coverage", "wrapper-patterns"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html"]
---

## Purpose

Provide source-grounded, agent-consumable context for development on Revenue Cloud Advanced and Dynamic Revenue Orchestrator. Project-specific entitlement, OCI, Nexus, Azure, and audit patterns are clearly separated from native Salesforce capabilities.

## When to use this doc (agent trigger conditions)

- First request in this repository.
- Unsure which topic file to load.
- Need the recommended traversal order or complete table of contents.

## Key concepts

- **Official fact:** backed by Salesforce documentation.
- **Project policy:** an architectural decision in these docs.
- **Unverified:** exact claim requires target-org, vendor, or repository confirmation.
- **Agent playbook:** executable, guarded workflow.
- **Canonical taxonomy:** skill names remain consistent across the five agentic-skill documents.

## Data model & objects

Start with [DRO Mapping](./dro-mapping.md) for design-time objects and [Fulfillment Orchestration](./fulfillment-orchestration.md) for runtime plans/steps. Use [Interface Coverage](./interface-coverage.md) before selecting an API.

## Flow / sequence

Recommended load order:
1. `README.md` and [index.md](./index.md).
2. [agentic-dro.md](./agentic-dro.md) for operating boundaries.
3. [dro-mapping.md](./dro-mapping.md) and [fulfillment-orchestration.md](./fulfillment-orchestration.md).
4. Topic document for the requested change.
5. [interface-coverage.md](./interface-coverage.md) and [wrapper-patterns.md](./wrapper-patterns.md) before implementation.
6. Verification, audit, observability, and chaos docs before release.

## APIs & extension points

Never choose an interface from memory. Confirm object/action/metadata support in official docs and target-org describe, then use Salesforce CLI for source changes and dry-run deployment.

## Configuration & metadata

Each file has YAML frontmatter for retrieval: `description`, `agent_use`, `salesforce_products`, `related`, `last_reviewed`, and `sources`. Relative links allow agents to expand context one hop at a time.

## Agent playbook

1. Read [index.md](./index.md) and match trigger terms.
2. Load no more than the core file plus two related files initially.
3. Resolve every API/object against official docs and org describe.
4. Treat `Unverified` blocks as stop points.
5. Edit source locally.
6. Run tests and dry-run deploy.
7. Return diffs, commands, and evidence.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not interpret project architecture recommendations as native Salesforce behavior.
- Do not skip linked lifecycle or audit docs when a change has external side effects.

## Verification & tests

1. Validate frontmatter and links.
2. Check exact API names against the target release.
3. Run commands in a sandbox.
4. Execute positive, negative, duplicate, and permission-denied paths.
5. Review all `Unverified` blocks before production design approval.

## References

- [Dynamic Revenue Orchestrator Essentials](https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Salesforce CLI project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)

### Table of contents

| File | Load for |
|---|---|
| [index](./index.md) | Dense machine-oriented topic routing table. |
| [agentic-dro](./agentic-dro.md) | Operating boundary for CLI agents that validate and propose DRO changes. |
| [agentic-skills-gap-analysis](./agentic-skills-gap-analysis.md) | Build-versus-adopt analysis for the canonical DRO skill taxonomy. |
| [agentic-skills-inventory](./agentic-skills-inventory.md) | Canonical skill names, triggers, inputs, outputs, interfaces, and guardrails. |
| [agentic-tooling](./agentic-tooling.md) | Minimal CLI, skills, MCP, and wrapper toolchain. |
| [amendments-renewals](./amendments-renewals.md) | Deferred asset lifecycle scope and future DRO reconciliation requirements. |
| [audit-system](./audit-system.md) | Cross-system evidence ledger and reconciliation design. |
| [azure-middleware](./azure-middleware.md) | Salesforce-to-Azure messaging, idempotency, ordering, and secrets pattern. |
| [chaos-testing](./chaos-testing.md) | Controlled failure-injection playbook. |
| [decomposition-viewer](./decomposition-viewer.md) | Read-only decomposition inspection and headless validation guidance. |
| [dro-mapping](./dro-mapping.md) | Commercial-to-technical mapping, rules, and enrichment. |
| [external-agentic-skills](./external-agentic-skills.md) | Due diligence for official and third-party skill repositories. |
| [fulfillment-orchestration](./fulfillment-orchestration.md) | Plans, steps, dependencies, callouts, fallout, and jeopardy. |
| [interface-coverage](./interface-coverage.md) | Verified interface routing and explicit gaps. |
| [licensing](./licensing.md) | Project entitlement schema and revocation boundary. |
| [nfr-parking-lot](./nfr-parking-lot.md) | Measurable deferred reliability and performance requirements. |
| [observability](./observability.md) | Correlation-driven telemetry and alerts. |
| [rlm-skills-porting](./rlm-skills-porting.md) | Selective semantic porting into SKILL.md packages. |
| [token-auditability](./token-auditability.md) | Safe token lifecycle evidence and negative events. |
| [wrapper-patterns](./wrapper-patterns.md) | Canonical Apex/Flow/MCP facade patterns. |

**Retrieval keywords:** RCA, DRO, documentation index, agent context, load order, RAG, Salesforce Revenue Cloud

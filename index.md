---
title: "RCA/DRO Agent Routing Index"
description: "Dense retrieval map from task intent and keywords to the smallest relevant DRO document set."
agent_use: "Load when routing an agent request to one or more topic files; prefer this over loading the full corpus."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["README", "agentic-skills-inventory", "dro-claude-code-research", "interface-coverage"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm"]
---

## Purpose

Route by intent with minimal context loading. This page is deliberately dense; [README](./README.md) is the human-oriented entry point.

## When to use this doc (agent trigger conditions)

- Any request that mentions DRO, RCA, decomposition, fulfillment, entitlement, audit, middleware, or agent skills.
- Any task where the correct file is uncertain.

## Key concepts

- Load one primary file.
- Add one implementation file (`interface-coverage` or `wrapper-patterns`).
- Add one operations file only when the change crosses systems.

## Data model & objects

For object/API lookup, route directly to [DRO Mapping](./dro-mapping.md), [Fulfillment Orchestration](./fulfillment-orchestration.md), or [Interface Coverage](./interface-coverage.md).

## Flow / sequence

1. Match topic/keywords.
2. Load primary file.
3. Follow `related` frontmatter one hop.
4. Stop when required facts and guardrails are present.
5. Escalate `Unverified` items to org describe or owner confirmation.

## APIs & extension points

Route implementation questions through [Interface Coverage](./interface-coverage.md); route missing-interface work through [Wrapper Patterns](./wrapper-patterns.md).

## Configuration & metadata

All routes are relative links and filenames without `.md` appear in frontmatter `related` arrays.

## Agent playbook

1. Extract nouns and requested verb.
2. Select the narrowest row.
3. Load linked file and its official sources.
4. Do not load all files.
5. Re-route if the task changes phase from design to implementation or operations.

## Guardrails & anti-patterns

- Do not answer exact API questions from this index alone.
- Do not treat keywords as proof of support.
- Do not load parked lifecycle/NFR scope unless the request triggers it.

## Verification & tests

1. Ensure every sibling file appears exactly once below.
2. Validate every relative link.
3. Test representative routing prompts against the expected primary file.

## References

- [Dynamic Revenue Orchestrator Essentials](https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)

### Routing table

| Topic / search terms | File | When to load |
|---|---|---|
| start, entry, overview | [README](./README.md) | Human-oriented entry point and load order. |
| agentic, dro | [agentic-dro](./agentic-dro.md) | Operating boundary for CLI agents that validate and propose DRO changes. |
| agentic, skills, gap | [agentic-skills-gap-analysis](./agentic-skills-gap-analysis.md) | Build-versus-adopt analysis for the canonical DRO skill taxonomy. |
| agentic, skills, inventory | [agentic-skills-inventory](./agentic-skills-inventory.md) | Canonical skill names, triggers, inputs, outputs, interfaces, and guardrails. |
| agentic, tooling | [agentic-tooling](./agentic-tooling.md) | Minimal CLI, skills, MCP, and wrapper toolchain. |
| amendments, renewals | [amendments-renewals](./amendments-renewals.md) | Deferred asset lifecycle scope and future DRO reconciliation requirements. |
| audit, system | [audit-system](./audit-system.md) | Cross-system evidence ledger and reconciliation design. |
| azure, middleware | [azure-middleware](./azure-middleware.md) | Salesforce-to-Azure messaging, idempotency, ordering, and secrets pattern. |
| chaos, testing | [chaos-testing](./chaos-testing.md) | Controlled failure-injection playbook. |
| claude, code, research, plugin, mcp | [dro-claude-code-research](./dro-claude-code-research.md) | Steps, best practices, and community experience driving DRO Product Decomposition from Claude Code. |
| decomposition, viewer | [decomposition-viewer](./decomposition-viewer.md) | Read-only decomposition inspection and headless validation guidance. |
| dro, mapping | [dro-mapping](./dro-mapping.md) | Commercial-to-technical mapping, rules, and enrichment. |
| external, agentic, skills | [external-agentic-skills](./external-agentic-skills.md) | Due diligence for official and third-party skill repositories. |
| fulfillment, orchestration | [fulfillment-orchestration](./fulfillment-orchestration.md) | Plans, steps, dependencies, callouts, fallout, and jeopardy. |
| interface, coverage | [interface-coverage](./interface-coverage.md) | Verified interface routing and explicit gaps. |
| licensing | [licensing](./licensing.md) | Project entitlement schema and revocation boundary. |
| nfr, parking, lot | [nfr-parking-lot](./nfr-parking-lot.md) | Measurable deferred reliability and performance requirements. |
| observability | [observability](./observability.md) | Correlation-driven telemetry and alerts. |
| rlm, skills, porting | [rlm-skills-porting](./rlm-skills-porting.md) | Selective semantic porting into SKILL.md packages. |
| token, auditability | [token-auditability](./token-auditability.md) | Safe token lifecycle evidence and negative events. |
| wrapper, patterns | [wrapper-patterns](./wrapper-patterns.md) | Canonical Apex/Flow/MCP facade patterns. |

**Retrieval keywords:** routing, index, topic map, retrieval, RAG, load file, agent context

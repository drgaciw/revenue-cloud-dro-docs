# Revenue Cloud DRO Design Notes

Index of practical design notes for Salesforce Revenue Cloud Advanced (RCA) and Dynamic Revenue Orchestrator (DRO), focused on delivering on-prem software as containers via Sonatype Nexus Cloud.

Each file is kept under ~200 lines, named by concern (not conversation), and tagged with keywords for agentic routing. An agent should match a query against the keywords and one-line summary before reading file contents.

## Core mapping and fulfillment

- [dro-mapping.md](dro-mapping.md) — Commercial-to-technical decomposition, execution rules, attribute mapping, and line-item design. **Keywords:** decomposition, commercial product, technical product, execution rules, attribute mapping, fulfillment order line items, hybrid line design.
- [fulfillment-orchestration.md](fulfillment-orchestration.md) — Step graph, dependencies, per-node failure handling, shallow-graph discipline, offline callout hold, fulfillment scenarios, and line-item sub-topic. **Keywords:** orchestration, step graph, dependencies, compensation, retry, line items, offline hold, fulfillment scenarios.
- [decomposition-viewer.md](decomposition-viewer.md) — Pre-commit checkpoint UI for inspecting decomposition results before order commit. **Keywords:** decomposition viewer, validation, launch gate, human inspection.
- [interface-coverage.md](interface-coverage.md) — CLI, API, Tooling API, and MCP reachability for RCA/RLM/DRO functions; which operations need browser automation and how to wrap them. **Keywords:** interface coverage, CLI, API, MCP, Tooling API, invocable actions, browser automation, reachability, agentic engineering.
- [wrapper-patterns.md](wrapper-patterns.md) — Thin Apex/Flow/MCP wrappers that close the Tier 4 browser gaps: Tooling API facades, decomposition validation, step-graph builder from JSON, and governance. **Keywords:** wrappers, invocable methods, Tooling API, MCP tools, browser automation, agentic engineering, decomposition rules, fulfillment steps, validation, gap closure.

## Entitlements, licensing, and access

- [licensing.md](licensing.md) — Schema evolution, revocation SLA, deny-list, token TTL, and the JSON schema contract between license system and Nexus. **Keywords:** licensing, schema evolution, revocation, deny-list, entitlement JSON, contract.
- [token-auditability.md](token-auditability.md) — Token-service audit chain: mint events, negative events, correlation IDs, from contract to receipt. **Keywords:** token, audit, correlation ID, negative events, mint, revoke.

## Systems and middleware

- [audit-system.md](audit-system.md) — Audit microservice, event chain, negative events, compensation records, download receipts. **Keywords:** audit, events, receipt, negative events, compensation records.
- [azure-middleware.md](azure-middleware.md) — Event Grid, Service Bus, async callout, idempotency, dead-letter queues, compensation logic, offline coordination, Key Vault. **Keywords:** middleware, Event Grid, Service Bus, idempotency, dead-letter, Key Vault, async callout, offline.
- [observability.md](observability.md) — Grafana dashboards, correlation-driven views, alerting on fulfillment gaps. **Keywords:** observability, Grafana, correlation ID, alerting, dashboard.

## Agentic and future topics

- [agentic-dro.md](agentic-dro.md) — Agentic validation across decomposition, entitlements, OCI labels, and the audit chain. **Keywords:** agentic, Claude, skills, validation, OCI labels, agent harness.
- [agentic-tooling.md](agentic-tooling.md) — Open-source and commercial tooling (Claude Code plugin, MCP servers, skills, LangGraph) to improve agentic engineering efficiency with DRO. **Keywords:** agentic tooling, Claude Code plugin, MCP server, Salesforce skills, LangGraph, CrewAI, headless development.
- [external-agentic-skills.md](external-agentic-skills.md) — Curated public GitHub repos with Claude Code skills for DRO and Revenue Cloud (lzdravkov/rlm-skills is the top match). **Keywords:** external skills, Claude Code, rlm-skills, DRO skill, agentic engineering, fulfillment callouts.
- [agentic-skills-gap-analysis.md](agentic-skills-gap-analysis.md) — Gap analysis of public agentic skills for DRO; efficacy of building custom skills; recommended skill set and build-vs-adopt decision. **Keywords:** gap analysis, custom skills, skill efficacy, decomposition validator, fulfillment graph reviewer, entitlement auditor, OCI label sync, build vs adopt.
- [agentic-skills-inventory.md](agentic-skills-inventory.md) — Full inventory of public GitHub agents/skills for DRO, ranked by relevance, plus the recommended Claude teaming model (router, schema grounding, custom validators/auditors, wrapper author). **Keywords:** skills inventory, Claude teaming, skill routing, build vs adopt, custom skills, public skills.
- [rlm-skills-porting.md](rlm-skills-porting.md) — Assessment of porting RLM skills (bgaldino/rlm-base-dev) to Claude Code SKILL.md format: feasibility, selective port plan, effort, risks. **Keywords:** porting, RLM skills, Claude Code, skill-port, SKILL.md, schema grounding, selective port.
- [chaos-testing.md](chaos-testing.md) — Failure injection to verify compensation, retries, and audit-trail behavior. **Keywords:** chaos, failure injection, resilience, compensation verification.
- [nfr-parking-lot.md](nfr-parking-lot.md) — Performance, smoke, and remaining non-functional topics. **Keywords:** NFR, performance, smoke test, load.
- [amendments-renewals.md](amendments-renewals.md) — Parked: amendments, renewals, cancellations flowing back to technical side. New logos only for now. **Keywords:** amendments, renewals, cancellation, parked.

## Additional parking-lot items (to be expanded)

- DRO-triggered provisioning of software images (vs CI/CD ahead of orders).
- CI/CD-to-Nexus versioning and naming alignment.
- JSON schema contract between license system and Nexus (lives in licensing.md but flagged for deeper treatment).

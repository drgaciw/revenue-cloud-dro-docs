# Interface Coverage: CLI, API, and MCP Reachability for RCA / RLM / DRO

**Keywords:** interface coverage, CLI, API, MCP, Tooling API, invocable actions, browser automation, agentic engineering, reachability, gaps

## Summary
Most DRO design-time configuration is metadata-deployable and reachable via Salesforce CLI and Tooling API. Runtime fulfillment is reachable via invocable actions, Connect REST, and Submit API. However, several design-time and operational functions still require the browser UI or browser automation. This file maps the coverage and recommends how to close the gaps for agentic workflows.

## Coverage tiers

### Tier 1 — Fully reachable (CLI + Tooling API + MCP)
- Decomposition rules, fulfillment step definitions, scenarios, workspaces, integration definitions, value transformations, fallout/jeopardy rules, task assignment rules.
- These are metadata types. `sf project deploy start` / retrieve works. Salesforce Hosted MCP servers (GA) can expose them via custom Apex or Named Queries.
- New in Spring '26: `OrchestrationPlanCtxMapping` Tooling API object for context mapping entries.

### Tier 2 — Runtime reachable (Invocable Actions + Connect REST + Submit API)
- Order submission to DRO (`Submit Sales Transaction` invocable action / Submit API).
- Decomposition and orchestration as separate processes (individual invocable actions).
- Callout step payloads (full hierarchy including bundle children) available for integration definitions.
- Assetization and fulfillment status via standard objects + REST/SOQL.
- Community licenses support "Submit orders to Dynamic Revenue Orchestrator."

### Tier 3 — Partial / data-only (REST/SOQL, limited CLI)
- Runtime fulfillment plans, steps, and status changes (monitor + manual status edits).
- Product2, related products, queues, users — standard sObjects, reachable via Data API / SOQL but not as clean metadata packages for complex graphs.
- Migration of rules between orgs: Data API (or Bulk API for high volume).

### Tier 4 — Browser / automation required
- Visual plan editor in the fulfillment workspace (drag-and-drop step graphs, dependency wiring).
- Decomposition Viewer (read-only inspection UI).
- Some advanced configuration in Salesforce Go (feature toggles, dunning templates, future-dated steps unlock).
- Manual task assignment UI and certain operator dashboards.
- Any function not yet exposed as an invocable action or Tooling object.

## Recommendations for agentic engineering

1. **Prefer metadata-first.** Keep decomposition rules, step definitions, scenarios, and integration definitions in source control. Agents (Claude Code + Salesforce DX MCP) can read, diff, propose, and deploy changes without touching the UI.

2. **Wrap the Tier 4 gaps.** For the visual editor and Decomposition Viewer, build thin Apex `@InvocableMethod` or Flow wrappers that perform the equivalent create/update/validate operations. Expose those as Hosted MCP tools. This turns "browser required" into "agent callable."

3. **Use the community MCP server as a starting point.** `MarijanMiletic/mcp_salesforce_revenue_cloud` (FastMCP) already exposes products, price books, quotes, orders, and arbitrary SOQL. Extend it with DRO-specific tools: list decomposition rules, fetch fulfillment step definitions, query orchestration plan status. This gives Claude Desktop / Cursor immediate read access.

4. **Leverage the official rlm-skills.** `lzdravkov/rlm-skills` includes an `rlm-dynamic-revenue-orchestrator` skill covering fulfillment plans, callout provider types, platform events, and object references. Install it into your Claude Code / Cursor harness and point it at the wrappers from step 2.

5. **Govern the write path.** Agents should propose changes (PR + metadata deploy) rather than mutate production rules directly. Use the same permission sets as humans: Fulfillment Designer for design-time, Submit Transactions for runtime. Never let an agent hold DRO Admin in a headless context.

6. **Track the gap list.** Maintain a living table (below) of Tier 4 functions and the wrapper status. Revisit each release — Salesforce is actively adding Tooling objects and invocable actions (e.g., OrchestrationPlanCtxMapping, separate decompose/orchestrate actions).

## Gap table (living)

| Function | Current interface | Agent path today | Recommended wrapper |
|---|---|---|---|
| Visual step-graph editor | Browser UI | None | Apex/Flow: create step + dependency from JSON spec |
| Decomposition Viewer | Browser UI (read-only) | SOQL on fulfillment objects | Apex: validate decomposition output vs entitlement JSON |
| Salesforce Go feature toggles | Browser | None | Document manual; avoid automation |
| Manual task reassignment | Browser | Limited | Flow: reassign by rule |
| Rule migration between orgs | Data/Bulk API | Scriptable | Keep in DX project; deploy via CLI |

## Practical default for this project
Start with Tier 1 + Tier 2 only. All decomposition rules, step definitions, scenarios, and callout integrations live in metadata and are agent-operable via CLI + Hosted MCP. Runtime submission and status are invocable-action driven. Anything that still needs the browser gets a thin wrapper before we rely on it in an agentic workflow — no silent browser automation in production paths.

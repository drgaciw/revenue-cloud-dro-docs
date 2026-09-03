# Agentic Tooling for DRO Efficiency

Open-source and commercial tools that improve agentic engineering workflows around Dynamic Revenue Orchestrator (DRO). Focused on Claude Code, MCP, and skills that reduce manual catalog and fulfillment work.

## Top recommendations

### 1. Salesforce Development Plugin for Claude Code (official, free)
- Install: `/plugin install salesforce-development@claude-plugins-official`
- Bundles ~40 skills, 3 MCP servers (api-context, metadata-experts, local LSP for Apex/SOQL), specialized agents, and hooks.
- Auto-detects SFDX projects, enforces deploy safety gates, and routes requests skills → CLI → MCP.
- Best for: authoring decomposition rules, custom Apex callouts, and metadata for fulfillment plans without leaving the terminal.
- Source: forcedotcom/sf-skills and Anthropic plugin marketplace.

### 2. Salesforce Hosted MCP Servers (official, GA)
- Enable in Setup → External Client Apps. Exposes SOQL, metadata, Flows, invocable actions, and custom Apex as discoverable tools.
- Lets Claude or any MCP client query fulfillment order line items, read decomposition rules, or invoke your middleware callouts headlessly.
- Best for: live validation of entitlement JSON against org data and runtime inspection of DRO plans.
- Pair with a custom MCP server that wraps your audit microservice or Nexus API for end-to-end traces.

### 3. salesforce-metadata-mcp (community, npm)
- `npx -y salesforce-metadata-mcp` — 200+ tools including Apex create/test, Flow builder, security scans, and a `cpq` toolset.
- Useful when the official plugin is too heavy; supports toolset filtering to keep agent context small.
- Best for: rapid iteration on technical product definitions and attribute mapping expressions.

### 4. arohitu/salesforce-revenue-cloud-skills (open source)
- `npx rcaskills add arohitu/salesforce-revenue-cloud-skills`
- Skills for PCM catalog, pricing diagnostics, configurator APIs, and decision tables.
- Complements the DRO-specific skill from lzdravkov/rlm-skills (already in external-agentic-skills.md).
- Best for: catalog hygiene and ensuring commercial products stay aligned with technical products.

### 5. LangGraph or CrewAI (frameworks, for custom agents)
- LangGraph for stateful, checkpointed workflows that mirror DRO step graphs (compensation, retries, idempotency).
- CrewAI for role-based crews (catalog agent, fulfillment agent, audit agent) when you need multi-agent review before commit.
- Best for: building the agentic validation layer described in agentic-dro.md outside Salesforce, then exposing results via MCP.

## Integration pattern

1. Claude Code + Salesforce plugin for authoring and deploying DRO metadata.
2. Hosted MCP for live org queries during design.
3. Custom MCP server for Nexus + audit + license system so agents can close the loop on entitlements.
4. External skills (rlm-skills, revenue-cloud-skills) for domain reasoning without bloating context.
5. LangGraph only if you need durable, multi-step agent orchestration that Salesforce alone cannot express.

## What to avoid

- Generic agent frameworks (OpenClaw, etc.) without Salesforce connectors — they lack field-level security and governor awareness.
- Loading every MCP tool at once — use toolsets or progressive disclosure to prevent context rot.
- Letting agents write production decomposition rules without the Decomposition Viewer gate.

## Keywords
agentic tooling, Claude Code plugin, MCP server, Salesforce skills, LangGraph, CrewAI, headless development, fulfillment validation

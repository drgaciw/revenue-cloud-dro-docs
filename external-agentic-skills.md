# External Agentic Skills for DRO

Curated public GitHub repositories containing Claude Code / agent skills relevant to Dynamic Revenue Orchestrator (DRO) and Revenue Cloud. These are third-party (non-Agentforce) resources for agentic engineering.

## Top recommendation

- [lzdravkov/rlm-skills](https://github.com/lzdravkov/rlm-skills) — Contains an explicit `rlm-dynamic-revenue-orchestrator` skill covering fulfillment plans, step definitions, three callout provider types (Standard HTTP, Apex Type, External Services), platform events (`FulfillmentSourceChangeEvent`, `SalesTrxnDecompositionEvent`), and object references. Closest match to our DRO process. Install via `./install.sh` into `~/.claude/skills/`.

## Supporting skills

- [arohitu/salesforce-revenue-cloud-skills](https://github.com/arohitu/salesforce-revenue-cloud-skills) — Broader Revenue Cloud skills (PCM, pricing, configurator APIs, decision tables). No dedicated DRO skill, but useful for catalog and decomposition context.
- [PranavNagrecha/AwesomeSalesforceSkills](https://github.com/PranavNagrecha/AwesomeSalesforceSkills) — Large library (1,000+ skills) with an Integration domain that mentions RLM DRO; includes MCP server for live-org metadata.
- [alexkwitko/Salesforce-Agentforce](https://github.com/alexkwitko/Salesforce-Agentforce) — Reference org with `salesforce-rlm` skill and headless RLM patterns; more Agentforce-oriented but reusable Claude Code skills.

## How to use

Clone the top repo, run the install script, then point Claude Code or your harness at the skills directory. Cross-reference with our `agentic-dro.md` for validation patterns specific to the Sonatype entitlement flow.

**Keywords:** external skills, Claude Code, rlm-skills, DRO skill, agentic engineering, fulfillment callouts.

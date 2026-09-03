# Agentic Skills Inventory and Claude Teaming

Inventory of public GitHub agents/skills relevant to DRO/RLM, plus a recommended teaming model for Claude Code. Focus is on what is actually reusable versus what must be custom-built for our container + Sonatype entitlement flow.

## Public inventory (ranked by DRO relevance)

| Repo / skill | Format | DRO depth | Reuse value | Notes |
|---|---|---|---|---|
| bgaldino/rlm-base-dev `.cursor/skills/revenue-cloud-data-model` (+ `domains/dro.md`) | Plain markdown, multi-agent | High schema reference | High as reference, low as procedure | 27 DRO objects, decomposition rules, step definitions, scenarios, fallout rules. Excellent object map; no entitlement/OCI/Nexus logic. |
| bgaldino/rlm-base-dev `.cursor/skills/expression-sets` | Plain markdown + scripts | Medium (pricing/rating procedures) | Medium | Connect/Metadata CRUD, overlays, validators. Useful pattern for step-graph authoring, not DRO-specific. |
| bgaldino/rlm-base-dev `.cursor/skills/skill-authoring` | Plain markdown | Meta | High | Lifecycle, registration, progressive disclosure, non-Cursor consumption. Copy its structure. |
| bgaldino/rlm-base-dev `.cursor/skills/revenue-cloud-docs` | Plain markdown | Low | Medium | Grounding against Salesforce Help. |
| bgaldino/rlm-base-dev `.cursor/skills/rlm-business-apis` | Plain markdown | Low-Medium | Medium | Revenue Cloud REST APIs. |
| lzdravkov/rlm-skills `rlm-dynamic-revenue-orchestrator` | Claude Code skill | Medium (fulfillment plans, callout providers, platform events) | High starter | Closest dedicated DRO skill; still generic, no container/Nexus entitlement model. |
| arohitu/salesforce-revenue-cloud-skills | Claude Code skills | Broad, shallow | Low | No DRO-specific skill. |
| PranavNagrecha/AwesomeSalesforceSkills | Library | Mentions RLM/DRO in one line | Low | Integration domain only. |
| alexkwitko/Salesforce-Agentforce | Agentforce-oriented | Low for DRO | Low | Reusable patterns, not DRO procedures. |
| lu-zhengda/skill-port | CLI | N/A | High tooling | Converts Cursor/Codex/Claude skills; useful for porting. |
| 0xMH/claude-skillify | Claude Code plugin | N/A | Medium | Turns a finished session into a SKILL.md. |

## What is missing from all public skills

- Commercial-to-technical mapping for a hundred-image catalog.
- OCI label/manifest discovery as the relationship layer.
- Entitlement JSON as the contract between RCA, DRO, license system, and Nexus.
- Token-service mint/revoke, deny-list, and negative-event audit chain.
- Async Event Grid callout, idempotency keys, dead-letter, compensation.
- Sonatype Nexus content selectors and path-based isolation.
- Decomposition Viewer as a pre-commit gate.
- Versioning policy: auto-include future versions until contract expiry.

## Recommended Claude teaming model

Treat skills as a layered team, not a flat list. Claude loads metadata first, then full SKILL.md only when triggered, then reference files on demand.

1. **Router / index skill** — `agentic-skills-inventory.md` (this file) plus `index.md`. Claude matches keywords before reading bodies.
2. **Schema grounding** — ported `revenue-cloud-data-model` (DRO domain) for object names, fields, relationships. Read-only reference.
3. **Decomposition validator** (custom) — simulates commercial-to-technical explosion, compares to entitlement JSON and OCI labels. Primary capability-uplift skill.
4. **Fulfillment graph reviewer** (custom) — reviews step graph, dependencies, per-node failure handling, shallow-graph discipline.
5. **Entitlement auditor** (custom) — checks token mint/revoke events, negative events, correlation IDs, deny-list coverage.
6. **OCI label sync** (custom) — reads manifests, detects drift between labels and catalog mapping.
7. **Wrapper author** (custom) — generates Apex/Flow/MCP wrappers for Tier 4 gaps (visual plan editor, Decomposition Viewer, Go toggles).
8. **Skill authoring** — ported `skill-authoring` for lifecycle, registration, evals.

### Teaming rules

- One skill per concern. Do not merge validator + auditor + OCI sync into one kitchen-sink skill.
- Keep each SKILL.md under ~500 lines; push detail to `references/`.
- Descriptions must include trigger phrases a human would actually say ("validate decomposition", "audit entitlement", "sync OCI labels").
- Custom skills own the proprietary logic; public skills own schema and generic procedure.
- Agents propose and validate; they do not silently rewrite live decomposition rules or fire production orders.
- Register every skill in the index with a one-line summary and keywords for routing.

## Build-vs-adopt decision

| Layer | Adopt public | Build custom |
|---|---|---|
| Object/schema reference | Yes (rlm-base-dev data model) | No |
| Generic step-graph CRUD patterns | Yes (expression-sets patterns) | Adapt, don't copy wholesale |
| Skill lifecycle | Yes (skill-authoring) | No |
| DRO decomposition validation | No | Yes — highest value |
| Entitlement/token/OCI audit | No | Yes |
| Nexus-specific access | No | Yes |

Public skills accelerate the foundation. They do not encode our hundred-image, OCI-label, Sonatype entitlement model. Custom skills are the differentiator.

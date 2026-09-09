---
title: "Agentic Skills Inventory and Teaming Model"
description: "Canonical inventory and capability taxonomy for CLI agents working on RCA/DRO."
agent_use: "Load when planning, selecting, evaluating, or implementing agent skills for DRO."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator", "Salesforce Platform"]
related: ["agentic-dro", "agentic-skills-gap-analysis", "agentic-tooling", "external-agentic-skills", "index", "rlm-skills-porting"]
last_reviewed: 2026-09-09
sources: ["https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html", "https://github.com/forcedotcom/sf-skills", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html"]
---

## Purpose

Define the one authoritative skill catalog used by inventory, gap analysis, porting, external sources, and tooling docs.

## When to use this doc (agent trigger conditions)

- “Which skill should the coding agent load?”
- “What skills are missing?”
- “Adopt or build?”
- “Define inputs, outputs, and guardrails.”

## Key concepts

- **Schema foundation:** Salesforce object/API grounding.
- **Domain validator:** deterministic comparison of Salesforce results with project fixtures.
- **Reviewer:** read-only graph/security/resilience checks.
- **Wrapper author:** produces governed interfaces, not business outcomes.
- **Progressive disclosure:** load the smallest skill that matches the trigger.

## Data model & objects

Skills must use exact APIs from the [DRO object reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm). The schema skill should cache describes for `ProductFulfillmentDecompRule`, `FulfillmentStepDefinition`, `ProductFulfillmentScenario`, `FulfillmentPlan`, and `FulfillmentStep` rather than carry guessed fields.

| Skill | Trigger | Inputs | Outputs | Backing interface | Guardrails |
|---|---|---|---|---|---|
| `dro-schema-grounding` | “Which DRO object/field?” | Org alias, API version, task | Cited schema map | Object reference + describe/SOQL | Read-only; no inferred fields |
| `dro-decomposition-validator` | “Validate decomposition” | Test order, expected entitlement fixture | Normalized diff | DRO runtime SObjects / read adapter | No live rule rewrite |
| `dro-fulfillment-graph-reviewer` | “Review step graph” | Scenario/group/steps/dependencies | Cycle/order/failure report | Design-time SObjects | No activation |
| `dro-entitlement-auditor` | “Trace entitlement” | Correlation ID, order/entitlement IDs | Chain-of-custody gaps | Audit API + read-only Salesforce records | Never expose token material |
| `dro-oci-label-sync` | “Check OCI labels” | Manifest labels, catalog mapping | Drift report | Registry API + local fixtures | Entitlement policy stays authoritative |
| `dro-wrapper-author` | “Expose as MCP/wrapper” | Approved operation and schema | Apex/Flow/MCP change + tests | `@InvocableMethod`, Hosted MCP | Thin facade; human-gated writes |

## Flow / sequence

1. Route by trigger.
2. Load `dro-schema-grounding` for any Salesforce schema work.
3. Load exactly one primary domain skill.
4. Add `dro-wrapper-author` only when a supported callable interface is required.
5. Run eval fixtures and return evidence.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Hosted MCP can expose approved Apex invocable actions, Flows, and SObject tools ([Salesforce Developers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)). The official Salesforce Skills Library supports multiple agent tools, but it is a platform foundation rather than DRO-specific authority ([GitHub](https://github.com/forcedotcom/sf-skills)).

## Configuration & metadata

Store each skill in its own directory with `SKILL.md`, `references/`, and `evals/`. Pin third-party revisions. Register exact trigger phrases. Keep org aliases, credentials, and proprietary fixtures outside skill prose.

## Agent playbook

1. Parse the request into taxonomy capabilities.
2. Select the smallest skill set.
3. Confirm official Salesforce references and target-org schema.
4. Run a golden fixture before proposing code.
5. Produce machine-readable outputs plus a human diff.
6. Require a PR and sandbox validation for changes.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not merge all six capabilities into one oversized skill.
- Do not use third-party skill prose as authority for Salesforce API names.
- Do not let validators mutate records.

## Verification & tests

1. Add positive, negative, ambiguous-schema, and permission-denied evals.
2. Verify each trigger selects the expected skill.
3. Run fixtures against a sandbox schema snapshot.
4. Fail closed on missing objects/fields.
5. Review output for token or credential disclosure.
6. Validate generated Apex/metadata with Salesforce CLI.

## References

- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Build Custom MCP Servers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)
- [Salesforce Skills Library](https://github.com/forcedotcom/sf-skills)

**Retrieval keywords:** agent skills, capability taxonomy, decomposition validator, graph reviewer, entitlement auditor, OCI label sync, wrapper author, schema grounding

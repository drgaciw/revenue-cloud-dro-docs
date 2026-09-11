---
title: "Agentic Interactions with DRO"
description: "Apply CLI agents to DRO validation and review while preserving deterministic platform execution and human-controlled writes."
agent_use: "Load for any third-party coding-agent workflow that reads, validates, or proposes DRO configuration."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["agentic-skills-inventory", "decomposition-viewer", "dro-claude-code-research", "dro-mapping", "dro-rules-management-research", "token-auditability", "wrapper-patterns"]
last_reviewed: 2026-09-09
sources: ["https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html", "https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html"]
---

## Purpose

Set the operating boundary: agents inspect, compare, generate diffs, and validate; DRO performs decomposition and orchestration. Agents do not silently rewrite rules or submit production orders.

## When to use this doc (agent trigger conditions)

- “Use Claude/Cursor/Codex on DRO.”
- “Validate decomposition against entitlements.”
- “Review compensation or audit events.”

## Key concepts

- **Propose-only:** generate source changes and evidence.
- **Read adapter:** normalize Salesforce state.
- **Golden fixture:** expected technical products/attributes for a synthetic order.
- **Cross-system validation:** compare Salesforce output with project-owned entitlement/OCI/audit data.

## Data model & objects

Ground reads in documented APIs. Start with `ProductFulfillmentDecompRule`, `ProductDecompEnrichmentRule`, `FulfillmentLineSourceRel`, `FulfillmentLineAttribute`, `FulfillmentPlan`, and `FulfillmentStep` as available in the org.

## Flow / sequence

1. Parse intent and select one canonical skill.
2. Describe/query the org read-only.
3. Load project fixtures.
4. Produce a deterministic diff.
5. Generate a proposed code/metadata change.
6. Run tests and dry-run deployment.
7. Hand off for human approval.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Expose only approved read/validation actions through Hosted MCP. Salesforce supports custom MCP tools backed by invocable Apex, Flow, and SObject tools ([Salesforce Developers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)).

## Configuration & metadata

Maintain project policy—entitlement schema, OCI labels, Nexus paths, audit correlation—in version-controlled fixtures and schemas. Do not hide this proprietary contract inside generic skill prose.

## Agent playbook

1. Load [Skills Inventory](./agentic-skills-inventory.md).
2. Select the task skill.
3. Confirm target org and environment.
4. Collect evidence with read-only queries.
5. Return a plan and diff before implementation.
6. Implement locally with tests.
7. Validate in sandbox.
8. Require explicit approval for deploy/submit.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- No live production order firing.
- No silent mutation after validation.
- No use of OCI labels as a substitute for the project’s entitlement authority.

## Verification & tests

1. Run static checks and Apex tests.
2. Validate the deployment with `sf project deploy start --dry-run --test-level RunLocalTests`.
3. Submit a synthetic, non-production order and inspect decomposition, fulfillment lines, plan, steps, and fallout.
4. Repeat the request with the same correlation/idempotency key and verify no duplicate external effect.
5. Exercise a negative path and confirm the failure is visible and recoverable.
6. Run golden decomposition, graph, audit, and permission-denied evals for the selected skill.

## References

- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Build Custom MCP Servers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)
- [Monitor Decomposition During Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)
- [Salesforce CLI project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)

**Retrieval keywords:** agentic DRO, propose-only, validation, Claude Code, Cursor, Codex, golden fixture, deterministic decomposition

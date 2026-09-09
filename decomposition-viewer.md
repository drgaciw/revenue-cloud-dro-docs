---
title: "Decomposition Viewer"
description: "Use the supported read-only viewer and runtime records to validate decomposition output and troubleshoot fallout."
agent_use: "Load when validating a product launch, diagnosing unexpected fulfillment lines, or designing a headless validation equivalent."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["dro-mapping", "wrapper-patterns", "agentic-dro"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html"]
---

## Purpose

Treat the Decomposition Viewer as an inspection surface. Salesforce says it shows how fulfillment order items decompose and supports validation and fallout troubleshooting ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)).

## When to use this doc (agent trigger conditions)

- “Open or reproduce the Decomposition Viewer result.”
- “Validate a SKU before launch.”
- “Explain Rollback, Field Amendment, or Start Date Adjustment subactions.”

## Key concepts

- **Fulfillment Lines tab:** lists orders and decomposed products.
- **Subaction:** shows the operation applied during decomposition; documented examples include Rollback, Field Amendment, and Start Date Adjustment.
- **Viewer:** read-only diagnostic UI, not a rule authoring API.

## Data model & objects

Inspect documented runtime objects such as `FulfillmentLineSourceRel`, `FulfillmentLineAttribute`, `FulfillmentPlan`, and the relevant fulfillment-order-line records. The Viewer page does not publish exact field APIs for its UI columns.

> **Unverified:** A complete programmatic query that reproduces every Viewer column is not documented on the cited page. Confirm fields with target-org object describe.

## Flow / sequence

1. Submit a non-production order.
2. Wait for decomposition.
3. Open the order’s decomposition monitoring surface.
4. Inspect source line, resulting fulfillment line, rule, enrichment, orchestration plan, and subaction.
5. Compare with the golden entitlement fixture.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Prefer SOQL over UI driving for repeatable diagnostics, but only query fields returned by target-org describe. A custom read-only `@InvocableMethod` can expose a normalized result through Hosted MCP ([MCP invocable actions](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html)).

## Configuration & metadata

No Viewer-specific deployable metadata or documented permission name is established by the cited Viewer page. Record the actual permission set and page assignment from the target org.

> **Unverified:** The exact user permission required to monitor decomposition is omitted from the public page extract.

## Agent playbook

1. Ask for org alias and test order ID; refuse production submission by default.
2. Describe relevant objects and save the schema snapshot.
3. Query runtime lineage and mapped attributes.
4. Normalize results into deterministic JSON.
5. Diff against expected technical products and attributes.
6. Return mismatches with record IDs and rule IDs; do not edit.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not represent the Viewer as a pre-commit simulator; Salesforce describes monitoring during fulfillment.
- Do not automate a live order solely to obtain a screenshot.

## Verification & tests

1. Run static checks and Apex tests.
2. Validate the deployment with `sf project deploy start --dry-run --test-level RunLocalTests`.
3. Submit a synthetic, non-production order and inspect decomposition, fulfillment lines, plan, steps, and fallout.
4. Repeat the request with the same correlation/idempotency key and verify no duplicate external effect.
5. Exercise a negative path and confirm the failure is visible and recoverable.
6. Compare headless JSON with the human Viewer for the same test order and document any fields that cannot be reproduced.

## References

- [Monitor Decomposition During Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)
- [Define How a Product Decomposes](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Hosted MCP Invocable Actions](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html)

**Retrieval keywords:** Decomposition Viewer, Fulfillment Lines tab, Subaction, decomposition monitoring, source lineage, launch validation

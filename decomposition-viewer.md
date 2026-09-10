---
title: "Decomposition Viewer"
description: "Use the supported read-only viewer and runtime records to validate decomposition output and troubleshoot fallout."
agent_use: "Load when validating a product launch, diagnosing unexpected fulfillment lines, or designing a headless validation equivalent."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["agentic-dro", "dro-claude-code-research", "dro-mapping", "interface-coverage", "wrapper-patterns"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_order_line_item_actions.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlinesourcerel.htm", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlineattribute.htm", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html"]
---

## Purpose

Treat the Decomposition Viewer as an inspection surface. Salesforce says it shows how fulfillment order items decompose and supports validation and fallout troubleshooting ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)).

## When to use this doc (agent trigger conditions)

- “Open or reproduce the Decomposition Viewer result.”
- “Validate a SKU before launch.”
- “Explain Rollback, Field Amendment, or Start Date Adjustment subactions.”

## Key concepts

- **Fulfillment Lines tab:** lists orders and decomposed products; “child products grouped under their parent in the bundle hierarchy” ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)).
- **Action:** the operation the fulfillment order line performs (`Add`, `Amend`, `NoChange`, `Renew`, `Cancel`) as documented on [Fulfillment Order Line Item Actions](https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_order_line_item_actions.htm&language=en_US&type=5).
- **Subaction:** “identifies the operation applied to an order or decomposed product during decomposition, such as Rollback, Field Amendment, or Start Date Adjustment.”
- **Reason for Action:** “a window containing links to the order products and enrichment rules that led to the action on the decomposed product.”
- **Viewer:** read-only diagnostic UI, not a rule-authoring API.

## Data model & objects

Every Viewer row is backed by standard SObjects that a coding agent can query directly through the Data API. Column-to-object mapping:

| Viewer column | Source object | Source field(s) |
|---|---|---|
| Order product | `OrderItem` | `Id`, `Product2Id`, `Quantity` |
| Decomposed product | `FulfillmentOrderLineItem` | `Id`, `FulfillmentOrderId`, `FulfillmentOrderLineItemNumber`, `Quantity`, `EndDate` |
| Bundle hierarchy grouping | `FulfillmentLineSourceRel` | `SourceType` (`SourceBundleRoot` \| `SourceLineItem`), `SourceLineItemId` |
| Source lineage | `FulfillmentLineSourceRel` | `FulfilmentOrderLineId`, `SourceLineItemId` (polymorphic to `OrderItem` or `FulfillmentOrderLineItem`) |
| Action | `FulfillmentOrderLineItem` action field | `Add`, `Amend`, `NoChange`, `Renew`, `Cancel` |
| Subaction (Rollback, Field Amendment, Start Date Adjustment, …) | `FulfillmentLineSourceRel.SupplementalAction` (API v62+) | `Add`, `Amend`, `Cancel`, `NoChange` |
| Mapped attributes | `FulfillmentLineAttribute` | `AttributeDefinitionId`, `AttributeName`, `AttributePicklistValueId`, `AttributeValue`, `ExternalId` |
| Reason for Action | `ProductFulfillmentDecompRule` + `ProductDecompEnrichmentRule` reached via `FulfillmentLineSourceRel` | Rule name and enrichment mappings that led to the action |

Canonical queries for a coding agent:

```sql
-- Bundle-hierarchy + action per fulfillment line for one order.
SELECT Id, FulfillmentOrderId, FulfillmentOrderLineItemNumber, Product2Id, Quantity
FROM   FulfillmentOrderLineItem
WHERE  FulfillmentOrderId IN (SELECT Id FROM FulfillmentOrder WHERE OrderId = :orderId)

-- Provenance and subaction for each fulfillment line.
SELECT Id, FulfilmentOrderLineId, SourceLineItemId, SourceType, SupplementalAction
FROM   FulfillmentLineSourceRel
WHERE  FulfilmentOrderLineId IN :folliIds

-- Mapped attributes to reproduce the attribute column of the Viewer.
SELECT Id, FulfillmentOrderLineItemId, AttributeDefinitionId, AttributeName,
       AttributePicklistValueId, AttributeValue, ExternalId
FROM   FulfillmentLineAttribute
WHERE  FulfillmentOrderLineItemId IN :folliIds
```

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

DRO ships a documented set of permission sets under the `DynamicRevenueOrchestratorUserPsl` permission set license ([Revenue Management permission sets table](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5)). The two that grant access to the Decomposition Viewer are:

| Persona | Permission Set Label | Permission Set API Name | Viewer access | Source |
|---|---|---|---|---|
| Fulfillment Operator / Manager | Fulfillment Manager/Operator | `DFOManagerOperatorUser` | “Allows you to change the state of a step, and view Fulfillment plan UI and decomposition UI.” | [Permission sets table](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5) |
| DRO Admin | DRO Admin User | `DfoAdminUser` | “Provides maximum access to the user … CRUD access on all DRO entities.” Covers the Decomposition Viewer via full DRO access. | [Permission sets table](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5) |

The [“Monitor Decomposition During Fulfillment” Help page](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5) presents a “User Permissions Needed … To monitor decomposition” row whose exact permission label is rendered only in the logged-in Salesforce Help viewer. Because the [Permission Sets Table](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5) is the authoritative Salesforce listing and states that `DFOManagerOperatorUser` grants “view … decomposition UI,” assign that permission set (or `DfoAdminUser`) as the least-privilege permission for coding agents that need to reproduce the Viewer.

```bash
# Assign the least-privilege Viewer permission to an agent test user.
sf org assign permset --target-org "$ORG_ALIAS" \
  --name DFOManagerOperatorUser --on-behalf-of "$AGENT_USER"
```

At run time, DRO uses the **Fulfillment User** — not the submitter — for orchestration ([Fulfillment User](https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_user.htm&language=en_US&type=5)). Ensure the configured Fulfillment User also holds `DFOManagerOperatorUser` or `DfoAdminUser` if the agent asks it to open the Viewer during a run.

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
- [Assign Agentforce Revenue Management Permission Sets](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5)
- [Fulfillment Order Line Item Actions](https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_order_line_item_actions.htm&language=en_US&type=5)
- [Fulfillment User](https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_user.htm&language=en_US&type=5)
- [FulfillmentLineSourceRel](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlinesourcerel.htm)
- [FulfillmentLineAttribute](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlineattribute.htm)
- [Define How a Product Decomposes](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Hosted MCP Invocable Actions](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html)

**Retrieval keywords:** Decomposition Viewer, Fulfillment Lines tab, Subaction, decomposition monitoring, source lineage, launch validation

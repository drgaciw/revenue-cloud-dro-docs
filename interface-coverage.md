---
title: "Interface Coverage for RCA and DRO"
description: "Route each DRO task to a verified UI, SObject, invocable, Metadata API, CLI, Flow, or Hosted MCP interface and expose gaps explicitly."
agent_use: "Load before promising that an agent can read, write, deploy, submit, or monitor a DRO capability."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator", "Hosted MCP Servers"]
related: ["wrapper-patterns", "agentic-tooling", "decomposition-viewer"]
last_reviewed: 2026-09-09
sources: ["https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_invocable_actions_parent.htm", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html", "https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5"]
---

## Purpose

Prevent plausible but unsupported interface claims. Coverage is release- and org-dependent; confirm with official docs and target-org describe before coding.

## When to use this doc (agent trigger conditions)

- “Can this be done with CLI/API/MCP?”
- “Does this require Setup or the DRO UI?”
- “How do we migrate this config?”
- “What wrapper is needed?”

## Key concepts

- **Documented SObject:** exact API name appears in the object reference.
- **Data migration:** ordered insert/update of configuration records.
- **Metadata deployment:** deployable metadata; do not equate every configuration SObject with Metadata API.
- **Hosted MCP:** configured server exposing approved SObject/Flow/invocable tools.
- **UI-only/unknown:** no verified programmatic write interface in reviewed documentation.

## Data model & objects

The DRO schema is entirely composed of **standard SObjects**. Each object listed on [DRO Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm) supports `create()`, `update()`, `upsert()`, `query()`, `retrieve()`, `delete()`, and `undelete()`. Route writes through the **Data API** (REST `/services/data/vXX.X/sobjects/<Name>`, Apex DML, or SOAP), not the Tooling API and not the Metadata API. Metadata API is used only for the DRO Setup surface (feature flags, Context Definition Settings, Fulfillment User selection) documented in the [DRO Metadata deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm).

| Capability | Verified interface | Status |
|---|---|---|
| Query DRO standard objects | REST/SOQL after object/field describe | Verified |
| Create/update DRO design-time records | **Data API** (`create()`/`update()`/`upsert()` on the SObject) with the deployment order in [DRO Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm) | Verified |
| Write rule-set references on `ProductFulfillmentDecompRule` | **Data API UPDATE** on the JSON Special Fields (INSERT does not create the reference) | Verified with constraint |
| Write decomposition-rule conditions | **Data API INSERT with empty condition, then UPDATE** with the source JSON | Verified with constraint |
| Deploy DRO Setup / feature flags | Metadata API (`Setup` and `Flag` types in the DRO metadata reference) | Verified |
| Submit order/sales transaction | DRO standard [invocable actions](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_invocable_actions_parent.htm) | Verified category; look up exact API name for the target release |
| Expose global Apex invocable action | Hosted MCP custom server | Verified |
| Deploy custom MCP server | Metadata API | Verified |
| Reproduce Decomposition Viewer columns programmatically | Query `FulfillmentOrderLineItem`, `FulfillmentLineSourceRel`, `FulfillmentLineAttribute` (see [Decomposition Viewer](./decomposition-viewer.md)) | Verified with constraint |
| Write arbitrary DRO configuration via Tooling API | Not established anywhere in the reviewed docs; SObject describe lists Data API calls only | Not supported |

## Flow / sequence

1. Classify operation as read, validate, configure, submit, monitor, or operate.
2. Search official object/action/metadata reference.
3. Confirm exact API and version in the target org.
4. Select the least-privilege interface.
5. If no interface is documented, keep it manual or build a supported thin wrapper.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Standard DRO invocable actions submit an order or sales transaction for fulfillment, but the parent page does not list individual API names ([developer guide](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_invocable_actions_parent.htm)). Fetch the child action page for the target API version before generating invocation payloads.

## Configuration & metadata

Migration order and lookup dependencies for DRO configuration objects are documented separately from Metadata API. Treat `FulfillmentStepDefinitionGroup` → `FulfillmentStepDefinition` → `FulfillmentStepDependencyDef` and product/rule prerequisites as ordered data dependencies ([deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)).

## Agent playbook

1. Create a row in the coverage table for the requested operation.
2. Attach an official URL and exact API name, or label it `Unverified`.
3. Run object describe and a read-only query.
4. For write operations, create sandbox fixtures and rollback instructions.
5. Prefer CLI deployment for source metadata, documented data migration for configuration records, and MCP only over approved actions.
6. Update this matrix when a Salesforce release changes reachability.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not classify an object as Tooling API just because it is design-time.
- Do not classify a UI page as a supported API.
- Do not convert a read-only diagnostic UI into an undocumented mutation path.

## Verification & tests

1. Run static checks and Apex tests.
2. Validate the deployment with `sf project deploy start --dry-run --test-level RunLocalTests`.
3. Submit a synthetic, non-production order and inspect decomposition, fulfillment lines, plan, steps, and fallout.
4. Repeat the request with the same correlation/idempotency key and verify no duplicate external effect.
5. Exercise a negative path and confirm the failure is visible and recoverable.
6. Record API version, object describe output, permission context, and command transcript for each “Verified” row.

## References

- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Dynamic Revenue Orchestrator Objects Deployment Reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)
- [Dynamic Revenue Orchestrator Additional Deployment Information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)
- [Dynamic Revenue Orchestrator Metadata Deployment Reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm)
- [DRO Standard Invocable Actions](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_invocable_actions_parent.htm)
- [Build Custom MCP Servers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)
- [Salesforce CLI project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)
- [Monitor Decomposition During Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)

**Retrieval keywords:** interface coverage, CLI, SOQL, Data API, Metadata API, Hosted MCP, invocable action, UI gap, reachability

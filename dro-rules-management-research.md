---
title: "Agentic DRO Rules Management"
description: "Evidence-based patterns for analyzing, creating, deploying, and safely modifying Salesforce Dynamic Revenue Orchestrator rule records with coding-agent CLIs."
agent_use: "Use when an agent must inspect, author, migrate, diff, or validate DRO decomposition, enrichment, fallout, task-assignment, scenario, rule-set-reference, or decision-table artifacts."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["agentic-dro", "agentic-tooling", "decomposition-viewer", "dro-claude-code-research", "dro-mapping", "external-agentic-skills", "fulfillment-orchestration", "interface-coverage", "wrapper-patterns"]
last_reviewed: 2026-09-10
sources:
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_valtfrmgrp.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_valtfrm.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productdecompenrichmentrule.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_prodtdecompenrchvarmap.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentfalloutrule.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmenttaskassignmentrule.htm"
  - "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentscenario.htm"
  - "https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5"
  - "https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5"
  - "https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5"
  - "https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5"
  - "https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin"
  - "https://developer.salesforce.com/blogs/2026/05/connect-claude-with-salesforce-hosted-mcp-servers"
  - "https://code.claude.com/docs/en/best-practices"
  - "https://code.claude.com/docs/en/hooks"
  - "https://github.com/forcedotcom/sf-skills"
  - "https://github.com/SalesforceAIResearch/agentforce-adlc"
  - "https://github.com/lzdravkov/rlm-skills"
  - "https://github.com/bgaldino/rlm-base-dev"
---

# Agentic Salesforce DRO Rules Management

This report covers the **rules layer** of Salesforce Dynamic Revenue Orchestrator (DRO): design-time authoring, migration, inspection, and runtime evaluation of decomposition, enrichment, fallout, task-assignment, scenario, rule-set-reference, and decision-table artifacts. It intentionally does not repeat the general Claude Code installation and end-to-end order-submission material in [`dro-claude-code-research`](./dro-claude-code-research.md).

**Evidence labels used below**

- **Official** — Salesforce or Anthropic documentation.
- **Shipped community pattern** — public repository content showing an implemented workflow, not a Salesforce product guarantee.
- **Recommended control** — a derived operating pattern; validate it in the target org.
- **Unverified** — the public evidence reviewed did not establish the claim.

## 1. Executive summary

- DRO rules are **data records with dependencies**, not a single deployable metadata type: Salesforce documents a strict orchestration-object order and a separate five-object decomposition/enrichment order ([Salesforce DRO deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)).
- Four condition-bearing objects require a special **INSERT shell, then UPDATE JSON** pattern: `ProductFulfillmentDecompRule.ConditionData`, `ProductFulfillmentScenario.ConditionData`, `FulfillmentTaskAssignmentRule.ConditionData`, plus condition fields on `FulfillmentStepDefinition`; an insert alone does not create the BRE rule-set reference ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).
- An agent should treat `sf sobject describe` as the schema authority, SOQL as the precondition/assertion layer, and a git-tracked normalized export as the review surface; Salesforce's own CLI-oriented skill explicitly says to verify identifiers with `sf sobject describe` instead of guessing ([Salesforce SOQL skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-soql-query/SKILL.md)).
- `FulfillmentFalloutRule` is operationally coupled to the **Fulfillment Fallout Rules decision table**: Salesforce requires that table to be refreshed after fallout-rule creation or update, and its migration guidance repeats the refresh requirement after migration ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5), [Salesforce migration notes](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).
- Public object references reviewed do **not** expose an `IsActive` field on the required rule objects. “`IsActive` gating” should therefore mean activation of the governing Business Rules Engine rule-library version, non-production targeting, and explicit conditions—not an invented per-rule field ([ProductFulfillmentDecompRule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm), [ProductFulfillmentScenario reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentscenario.htm)).
- Salesforce's official development plugin and Hosted MCP can ground an agent in org schema, run queries, and validate deployments, but the reviewed `forcedotcom/sf-skills` commit contains **no dedicated DRO rule-authoring skill**; DRO-specific prompts must add the dependency order and two-phase write discipline themselves ([Salesforce development agent](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/agents/salesforce-dev.md), [Salesforce headless development](https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin)).
- The strongest public GitHub implementation pattern is `bgaldino/rlm-base-dev`: it exports DRO records through SFDMU into CSV, uses stable keys such as `Name` and product `StockKeepingUnit`, and sequences upserts, creating a useful git-diff surface; its sample still contains source-specific identifiers and omissions, so it is evidence of technique rather than a turnkey migration package ([SFDMU export configuration](https://github.com/bgaldino/rlm-base-dev/blob/7999013b2b505e0d20b78021583ff99a8ae59566/datasets/sfdmu/mfg/en-US/mfg-dro/export.json)).
- Community experience is positive about plan-first, metadata-aware agents and rapid test/fix cycles, but also records destructive-command risk and poor results from oversized prompts; none of the reviewed first-person accounts demonstrates production DRO-rule authoring end to end, so the community signal is **general Salesforce-plus-agent evidence**, not DRO validation ([Salesforce Ben](https://www.salesforceben.com/10-lessons-for-admins-using-claude-code-to-build-salesforce-flows/), [r/Codex deletion incident](https://www.reddit.com/r/codex/comments/1ol3iaj/codex_just_ran_sfdx_delete_from_project_and_org/)).

## 2. The DRO rules surface

### 2.1 Product Fulfillment Decomposition Rule family

#### `ProductFulfillmentDecompRule`

`ProductFulfillmentDecompRule` defines how a commercial or technical product decomposes to a destination technical product; the object is available from API version 61.0 and supports create, query, update, upsert, delete, undelete, retrieve, search, and describe operations ([Salesforce object reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm)).

| Field | Agent-relevant meaning |
|---|---|
| `SourceProductId` / `SourceProductClassificationId` | Scope to a source product or product classification. |
| `DestinationProductId` | Technical product produced by decomposition. |
| `Priority` | Evaluation/write precedence. Salesforce states that rules run in priority order; lower numeric priority wins when multiple mappings write the same target, with most-recently-modified as the same-priority tiebreaker ([Salesforce decomposition guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)). |
| `ConditionData` | Large-text JSON representation of the condition; createable, updateable, and documented from API 66.0 ([Salesforce object reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm)). |
| `SourceIdentifier`, `SourceClassIdentifier`, `DestinationIdentifier` | Stable identifier-oriented fields documented from API 65.0; prefer them over copied record IDs when the target release and data model support them ([Salesforce object reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm)). |
| `Name` | Human review key; not a substitute for checking uniqueness in the target org. |

**Lifecycle.** Design begins by resolving source scope and destination product, then optionally building a BRE-backed condition. Unless an execution condition exists and evaluates false, Salesforce says products decompose by default; product-class rules are considered before product-specific rules, with priority and modification time controlling order within those groups ([Salesforce decomposition guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)). Deploy the shell record before its condition JSON, then deploy its enrichment children after the parent ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm), [Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). At runtime, the selected rule creates the destination technical product, after which enrichment rules map values into the decomposition output ([Salesforce mapping guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)).

#### `ValTfrmGrp` and `ValTfrm`

`ValTfrmGrp` is a named list-mapping group with source and destination primitive types, enumerated-value flags, and `UsageType`; the documented `UsageType` value is `DFOListMapping`, and supported operations include full CRUD/upsert plus query/search/describe ([Salesforce ValTfrmGrp reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_valtfrmgrp.htm)). Its primitive-type fields permit Boolean, Currency, Date, Datetime, Number, Percent, and Text values ([Salesforce ValTfrmGrp reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_valtfrmgrp.htm)).

`ValTfrm` is a child row in a transformation group and provides typed input/output slots—Boolean, Date, Datetime, Number, String, and picklist-value references—linked by `ValueTransformGroupId`; the object supports create, query, update, upsert, delete, undelete, retrieve, and describe operations ([Salesforce ValTfrm reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_valtfrm.htm)). Salesforce describes List Mapping as a source-to-destination value-pair transformation, distinct from exact-copy “As Is” and Expression Set transformation ([Salesforce mapping guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)).

**Lifecycle.** Create the group, create its rows, then reference the group from an enrichment rule; deploy `ValTfrmGrp` before `ValTfrm` and before enrichment rules that consume them ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)). At runtime, the selected `ValTfrm` row converts one enumerated or primitive source value to the configured destination value ([Salesforce mapping guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)).

#### `ProductDecompEnrichmentRule`

`ProductDecompEnrichmentRule` is the child mapping definition for a decomposition rule; it supports create, query, update, upsert, delete, undelete, retrieve, and describe operations from API 61.0 ([Salesforce enrichment-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productdecompenrichmentrule.htm)).

| Field group | Confirmed fields and semantics |
|---|---|
| Parent | `DecompositionRuleId` references `ProductFulfillmentDecompRule`. |
| Method | `CalculationMethod`: `Ad-verbatim` (As Is), `Static-Lookup` (List Mapping), or `Expression-Set` from API 64.0; `CalculationDefinitionId` can reference a decision-matrix definition or expression set ([Salesforce enrichment-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productdecompenrichmentrule.htm)). |
| Source | `SourceType` (`Attribute` or `Field`), `SourceApiName`, `SourceContextTag`, and identifier/definition fields. |
| Destination | `DestinationType` (`Attribute` or `Field`), `DestinationApiName`, `DestinationContextTag`, and identifier/definition fields. |
| List mapping | `ListMappingGroupId` references `ValTfrmGrp`. |
| Enforcement | `RuleEnforcement`: `AllFulfillmentRequests` or `InitialFulfillmentRequest` from API 63.0 ([Salesforce enrichment-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productdecompenrichmentrule.htm)). |

Salesforce requires compatible source and destination data types, defines As Is as exact copy, List Mapping as paired translation, and Expression Set as the complex-transformation path ([Salesforce mapping guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)). During migration, Salesforce warns that identifier fields can contain source-org IDs and recommends saving again or setting them to null so target values can be populated ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

**Lifecycle.** Assert the parent decomposition rule and all referenced fields, attributes, context tags, value-transform groups, decision matrices, or expression sets; insert the enrichment rule only after those dependencies; then re-read it to verify identifiers. At runtime, mappings enrich the destination decomposition record under their selected method and enforcement mode ([Salesforce mapping guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)).

#### `ProdtDecompEnrchVarMap`

`ProdtDecompEnrchVarMap` connects an enrichment rule to Expression Set input/output variables, exposing `ExpressionSetVarName`, `VariableType` (`Input` or `Output`), product-attribute/context-tag information, and master-detail `ProductDecompEnrichmentRuleId` ([Salesforce variable-map reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_prodtdecompenrchvarmap.htm)). The official page lists describe, query, retrieve, `getDeleted`, and `getUpdated` as supported calls, but does not list create, update, or upsert—even though one identifier field carries create/update properties—so direct Data API mutation is **not a verified supported workflow** and must be feature-detected in the target org ([Salesforce variable-map reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_prodtdecompenrchvarmap.htm)).

**Agent rule:** inspect and diff these mappings freely; do not generate a direct `sf data create record` command for them unless target-org describe plus an official supported interface establishes mutability. Deploy them after `ProductDecompEnrichmentRule` in Salesforce's documented sequence ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)).

### 2.2 Fulfillment Fallout Rule

`FulfillmentFalloutRule` maps failure characteristics to retry and routing behavior; the object supports create, query, update, upsert, delete, undelete, retrieve, and describe operations from API 61.0 ([Salesforce fallout-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentfalloutrule.htm)).

| Field | Confirmed role |
|---|---|
| `ErrorCode` | Error selector. |
| `StepType` | `AutoTask`, `Callout`, `ManualTask`, `Milestone`, or `Pause`. |
| `RetriesAllowed` | Maximum retries. |
| `RetryPolicy` | `Immediate`, `Monotonous`, or `Staggered`. |
| `RetryIntervals` | Retry timing data, documented from API 62.0. |
| `FalloutQueueId` | Queue (`Group`) target, documented from API 62.0. |
| `FlowDefinitionName` | Flow definition invoked by the rule. |
| `IntegrationDefinitionId` | Referenced integration definition. |
| `Name` | Auto-numbered rule name. |

Salesforce describes fallout rules as controlling retry count/timing and queue assignment, and requires refresh of the **Fulfillment Fallout Rules** decision table after rule creation or update ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)). The documented object deployment order places `FulfillmentFalloutRule` after workspaces and before jeopardy and task-assignment rules ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)).

**Lifecycle.** Design failure match → retry policy → routing/automation; insert or update the record; refresh the referenced decision table; test one matching failure and one non-match; verify the runtime path. A public, supported CLI endpoint for the refresh action was not found in the reviewed sources, so any “decision-table refresh script” is **Unverified** until backed by an org-specific API trace or official interface.

### 2.3 Fulfillment Task Assignment Rule

`FulfillmentTaskAssignmentRule` routes tasks from a source queue to a user or queue and supports create, query, update, upsert, delete, undelete, retrieve, search, and describe operations from API 63.0 ([Salesforce task-assignment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmenttaskassignmentrule.htm)).

| Field | Confirmed role |
|---|---|
| `SourceId` | Source `Group`. |
| `DestinationId` | Destination `Group` or `User`. |
| `TaskAllocationType` | `ContextBased`, `LeastLoaded`, or `RoundRobin`. |
| `Priority` | Rule precedence. |
| `ConditionData` | Condition JSON; createable/updateable from API 66.0. |
| `ConditionId` | Polymorphic reference to an `ExpressionSet`. |
| `UsageType` | Usage discriminator; default documented as `Fulfillment`. |

**Lifecycle.** Resolve queue/user references and allocation policy, create the shell record, then update `ConditionData` so Salesforce creates the associated BRE reference ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). Deploy it after `FulfillmentStepJeopardyRule` in the orchestration sequence ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)). At runtime the rule selects a destination under the configured allocation type when its source and condition match; the exact tie-break behavior beyond documented priority should be verified in the target release rather than inferred.

### 2.4 Product Fulfillment Scenario

`ProductFulfillmentScenario` scopes a product or product classification and order action to a `FulfillmentStepDefinitionGroup`; it supports full create/query/update/upsert/delete/undelete/retrieve/search/describe operations from API 61.0 ([Salesforce scenario reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentscenario.htm)).

| Field | Confirmed role |
|---|---|
| `ProductId` / `ProductClassificationId` | Product or classification scope; `ProductId` is nillable from API 64.0. |
| `FulfillmentStepDefnGroupId` | Referenced fulfillment step-definition group. |
| `Action` | Multi-select action scope including `Add`, `Amend`, `Cancel`, `NoChange`, and `Renew`. |
| `ConditionData` | Condition JSON, documented from API 66.0. |
| `SourceIdentifier`, `SourceClassIdentifier` | Identifier-oriented source fields from API 65.0. |
| `UsageType` | Usage discriminator from API 66.0. |

**Lifecycle.** Create the step-definition group first, create the scenario shell with product/classification and action scope, then update `ConditionData`; Salesforce places the scenario before workspaces in the orchestration deployment sequence ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm), [Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). Runtime selection should be tested across every included action and with both condition outcomes because the scenario determines which orchestration group is applicable.

### 2.5 Rule-set references and the UPDATE-only condition attachment

The public deployment note names the exact pairings below; there is no generic documented field named `RuleSetReferencesJson` on these DRO objects ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

| Owning object | JSON field | Related rule field |
|---|---|---|
| `ProductFulfillmentDecompRule` | `ConditionData` | `ExecuteOnRule` |
| `ProductFulfillmentScenario` | `ConditionData` | `ScenarioRule` |
| `FulfillmentTaskAssignmentRule` | `ConditionData` | `Condition` |
| `FulfillmentStepDefinition` | `ExecuteOnConditionData` | `ExecuteOnRule` |
| `FulfillmentStepDefinition` | `ResumeOnConditionData` | `ResumeOnRule` |

These text-area JSON fields encode the condition that Salesforce materializes as a Business Rules Engine rule-set reference; Salesforce explicitly says the reference is not created during INSERT and requires a subsequent UPDATE of the JSON field ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). Attributes and picklist values used by the condition must already exist in the target org; Salesforce resolves attribute definitions by attribute code and picklist values by name during migration ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

**Safe two-phase algorithm**

1. Assert all target references and condition vocabulary with SOQL and describe.
2. Insert the owning record **without** the JSON condition field.
3. Capture the created ID from JSON output.
4. Update that same record with the source-derived, reviewed condition JSON.
5. Re-query both the JSON field and related rule field.
6. Fail the run if the related rule field remains null or the normalized JSON differs.

The agent must not invent the JSON schema. Export a known-good record from the same release, preserve its structure, and substitute only identifiers already proven in the target org.

### 2.6 Decision tables referenced by fallout rules

The fallout record and its decision table are a consistency pair, not independent artifacts: Salesforce instructs administrators to refresh the Fulfillment Fallout Rules decision table after fallout-rule creation/update and again after migration ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5), [Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). Therefore the deployment transaction is not “complete” when the DML call succeeds; completion is **DML + table refresh + probe of a representative match**.

## 3. Agentic rule analysis

### 3.1 Verification-first inventory

Start every session with an explicit target and API-shape check. Salesforce's official development agent requires target-org verification and JSON CLI output, while its SOQL skill says not to guess objects or fields ([Salesforce development agent](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/agents/salesforce-dev.md), [Salesforce SOQL skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-soql-query/SKILL.md)).

```bash
set -euo pipefail
ORG="dro-dev"

sf org display --target-org "$ORG" --json | tee evidence/org-display.json

for obj in \
  ProductFulfillmentDecompRule ValTfrmGrp ValTfrm \
  ProductDecompEnrichmentRule ProdtDecompEnrchVarMap \
  FulfillmentFalloutRule FulfillmentTaskAssignmentRule \
  ProductFulfillmentScenario; do
  sf sobject describe --sobject "$obj" --target-org "$ORG" --json \
    > "evidence/describe-${obj}.json"
done
```

The `sf sobject describe` command is Salesforce's supported CLI surface for object metadata inspection ([Salesforce CLI reference](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_sobject_describe.html)). A reviewer should compare `createable`, `updateable`, reference targets, picklists, and API availability—not just field names.

### 3.2 SOQL recipes that avoid guessed fields

The following projections use fields documented on the official object pages; still run describe first because release and permission differences can reduce visibility ([Salesforce CLI data-query reference](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_query.html)).

```bash
sf data query -o "$ORG" --json -q \
"SELECT Id, Name, SourceProductId, SourceProductClassificationId,
        DestinationProductId, Priority, ConditionData,
        SourceIdentifier, SourceClassIdentifier, DestinationIdentifier,
        LastModifiedDate
 FROM ProductFulfillmentDecompRule
 ORDER BY SourceProductClassificationId, SourceProductId, Priority, LastModifiedDate"

sf data query -o "$ORG" --json -q \
"SELECT Id, Name, SourcePrimitiveType, DestinationPrimitiveType,
        IsSourceEnumerated, IsDestinationEnumerated, UsageType
 FROM ValTfrmGrp ORDER BY Name"

sf data query -o "$ORG" --json -q \
"SELECT Id, Name, ValueTransformGroupId,
        InputString, OutputString, InputNumber, OutputNumber,
        InputPicklistValueId, OutputPicklistValueId
 FROM ValTfrm ORDER BY ValueTransformGroupId, Name"

sf data query -o "$ORG" --json -q \
"SELECT Id, DecompositionRuleId, CalculationMethod,
        CalculationDefinitionId, ListMappingGroupId,
        SourceType, SourceApiName, SourceContextTag,
        DestinationType, DestinationApiName, DestinationContextTag,
        RuleEnforcement
 FROM ProductDecompEnrichmentRule
 ORDER BY DecompositionRuleId, DestinationType, DestinationApiName"

sf data query -o "$ORG" --json -q \
"SELECT Id, ProductDecompEnrichmentRuleId, ExpressionSetVarName,
        VariableType, ProductAttributeIdentifier, FieldContextTagName
 FROM ProdtDecompEnrchVarMap
 ORDER BY ProductDecompEnrichmentRuleId, VariableType, ExpressionSetVarName"

sf data query -o "$ORG" --json -q \
"SELECT Id, Name, ErrorCode, StepType, RetriesAllowed,
        RetryPolicy, RetryIntervals, FalloutQueueId,
        FlowDefinitionName, IntegrationDefinitionId, LastModifiedDate
 FROM FulfillmentFalloutRule ORDER BY ErrorCode, StepType"

sf data query -o "$ORG" --json -q \
"SELECT Id, Name, SourceId, DestinationId, TaskAllocationType,
        Priority, UsageType, ConditionData, ConditionId, LastModifiedDate
 FROM FulfillmentTaskAssignmentRule ORDER BY SourceId, Priority, LastModifiedDate"

sf data query -o "$ORG" --json -q \
"SELECT Id, Name, ProductId, ProductClassificationId,
        FulfillmentStepDefnGroupId, Action, UsageType, ConditionData,
        SourceIdentifier, SourceClassIdentifier, LastModifiedDate
 FROM ProductFulfillmentScenario ORDER BY ProductClassificationId, ProductId, Name"
```

### 3.3 Normalize, export, and diff

A public Revenue Cloud repository ships an SFDMU configuration that extracts these objects into CSV and upserts them in dependency order, demonstrating a practical git-tracked data workflow ([`rlm-base-dev` SFDMU configuration](https://github.com/bgaldino/rlm-base-dev/blob/7999013b2b505e0d20b78021583ff99a8ae59566/datasets/sfdmu/mfg/en-US/mfg-dro/export.json)). Its files include decomposition, scenario, and fallout data, but sample values include environment-specific identifiers; copying the rows unchanged would be unsafe ([sample decomposition export](https://github.com/bgaldino/rlm-base-dev/blob/7999013b2b505e0d20b78021583ff99a8ae59566/datasets/sfdmu/mfg/en-US/mfg-dro/ProductFulfillmentDecompRule.csv), [sample fallout export](https://github.com/bgaldino/rlm-base-dev/blob/7999013b2b505e0d20b78021583ff99a8ae59566/datasets/sfdmu/mfg/en-US/mfg-dro/FulfillmentFalloutRule.csv)).

Recommended diff discipline:

```bash
mkdir -p rules/raw rules/normalized evidence
# Write each query result as JSON/CSV, then normalize:
# - sort records by semantic key
# - sort JSON object keys recursively
# - retain priority and LastModifiedDate explicitly
# - replace org IDs only through a reviewed key map
# - never redact condition operators or values that affect behavior

git diff -- rules/normalized/
```

For review, classify every change as **scope**, **condition**, **destination**, **transformation**, **priority/order**, **retry/routing**, or **reference repair**. A raw line diff is insufficient when one JSON string contains an entire condition tree.

### 3.4 “Explain this rule set” slash command

The following custom command is a **recommended project command**, not a command found in the reviewed official plugin:

```text
/dro-explain-rules <org-alias> <scope>

Enter plan/read-only mode. Confirm the org alias and refuse production aliases.
Run sf sobject describe for every object you plan to query.
Query the matching ProductFulfillmentScenario, ProductFulfillmentDecompRule,
ProductDecompEnrichmentRule, ValTfrmGrp/ValTfrm, ProdtDecompEnrchVarMap,
FulfillmentFalloutRule, and FulfillmentTaskAssignmentRule records.

Produce:
1. the scenario-to-step-group scope;
2. source product/class -> destination product decomposition graph;
3. every condition as normalized JSON plus a plain-English predicate;
4. every field/attribute mapping and transformation method;
5. priority collisions, same-target writes, dangling references, and source IDs;
6. fallout retry/routing matrix and whether DT refresh evidence exists;
7. task-assignment routes and allocation types;
8. exact SOQL evidence for each conclusion.

Do not write, deploy, refresh, or activate anything.
Mark any field not present in describe as Unverified.
```

Claude Code recommends separating exploration/planning from implementation and using concrete verification criteria ([Claude Code best practices](https://code.claude.com/docs/en/best-practices)). This command turns that guidance into DRO-specific evidence collection.

### 3.5 What the Salesforce Development plugin actually contributes

The official Salesforce skills repository supports Claude Code, Cursor, Codex, and other skill-capable agents, and its Salesforce development agent uses a hierarchy of skills, then CLI, then API ([`forcedotcom/sf-skills` README](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/README.md), [Salesforce development agent](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/agents/salesforce-dev.md)). The reviewed commit provides generic SOQL/schema validation and deployment-validation skills, plus a deploy gate, but repository-wide searches did not find the requested DRO rule-object names; consequently, “the plugin bundles DRO rule-authoring skills” is **Unverified/false for the reviewed commit** ([SOQL skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-soql-query/SKILL.md), [deployment validation skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-deploy-validate/SKILL.md)).

Hosted MCP adds authenticated, org-scoped tools to Claude, which is valuable for describe/query grounding; Salesforce documents connecting Claude to Hosted MCP servers and positions the integration as a headless-development surface ([Salesforce Hosted MCP blog](https://developer.salesforce.com/blogs/2026/05/connect-claude-with-salesforce-hosted-mcp-servers), [Salesforce Hosted MCP setup](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/claude.html)). It does not eliminate DRO deployment ordering, two-phase DML, or the decision-table refresh checkpoint.

## 4. Agentic rule creation

### 4.1 Common creation contract

Every mutation prompt should force the agent through this contract:

1. **Target proof:** print org alias, instance, username, sandbox status, API version, and relevant permission visibility.
2. **Schema proof:** describe all objects and reject requested fields that are absent or not createable/updateable.
3. **Reference proof:** resolve every product, classification, queue, user, group, step group, context tag, attribute, transform group, decision matrix, or expression set by a stable semantic key.
4. **Collision proof:** query existing records with the same scope/destination/priority or error/step pair.
5. **Plan artifact:** show ordered DML, before/after records, rollback DML, and tests.
6. **Non-production mutation:** execute only after approval.
7. **Postconditions:** re-query, normalize, diff, and test runtime behavior.

### 4.2 Product decomposition creation prompt

```text
Create a ProductFulfillmentDecompRule in <sandbox-alias> for source
<product-or-classification-key> -> destination <technical-product-key>.
Use priority <n> and the business predicate: <plain-English condition>.

Before writing:
- run sf sobject describe on ProductFulfillmentDecompRule and Product2;
- resolve source/destination by stable identifiers and show exact SOQL;
- list colliding rules ordered by Priority and LastModifiedDate;
- export one known-good API-66+ ConditionData example from this org;
- translate the predicate by modifying that proven shape, never by inventing JSON;
- show an INSERT-without-ConditionData followed by UPDATE-ConditionData plan;
- show rollback and positive/negative decomposition tests.

Do not mutate until the plan and normalized condition diff are approved.
After approval, use JSON CLI output, capture the ID, update ConditionData,
re-query ConditionData and ExecuteOnRule, and stop if the rule reference is null.
```

This prompt reflects Salesforce's default-decompose behavior, priority rules, and required second update for the condition reference ([Salesforce decomposition guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5), [Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

### 4.3 List-mapping and enrichment creation prompt

```text
Create the mapping for decomposition rule <semantic-key>:
source <Field|Attribute>:<name> -> destination <Field|Attribute>:<name>,
method <As Is|List Mapping|Expression Set>, enforcement <mode>.

Plan only first. Describe ValTfrmGrp, ValTfrm,
ProductDecompEnrichmentRule, and ProdtDecompEnrchVarMap.
Verify source/destination data types and all context tags/attribute identifiers.

If List Mapping:
1. propose a ValTfrmGrp with primitive types and DFOListMapping usage;
2. show every typed ValTfrm input/output pair;
3. create group, rows, then enrichment rule.
If Expression Set:
1. resolve CalculationDefinitionId by semantic key;
2. inspect existing variable mappings;
3. do not directly create ProdtDecompEnrchVarMap unless describe and an
   official supported interface prove mutation is supported in this org.

After creation, re-query identifiers. If source-org IDs appear in enrichment
identifier fields, apply Salesforce's null-and-save/re-save migration repair,
then verify the target-populated identifiers.
```

This prompt follows Salesforce's documented method semantics, type-compatibility rule, deployment sequence, and identifier-repair note ([Salesforce mapping guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5), [Salesforce deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

### 4.4 Fallout-rule creation prompt

```text
Create a FulfillmentFalloutRule in <sandbox-alias> for:
ErrorCode=<code>, StepType=<type>, RetriesAllowed=<n>,
RetryPolicy=<policy>, RetryIntervals=<value>, FalloutQueue=<queue-key>,
Flow=<flow-name>, IntegrationDefinition=<semantic-key>.

Describe the object and validate every picklist/reference. Query overlapping
ErrorCode + StepType rules and show the effective matrix. Produce rollback.
After insert, re-query the exact record and stop at a mandatory checkpoint:
refresh the Fulfillment Fallout Rules decision table through a verified
supported interface. Do not claim completion until refresh evidence and one
matching-failure probe are recorded. If no scriptable refresh API is proven,
request the documented UI refresh and mark automation Unverified.
```

The mandatory refresh is an official requirement, whereas a CLI automation command for it was not established by this research ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)).

### 4.5 Task-assignment creation prompt

```text
Create a FulfillmentTaskAssignmentRule in <sandbox-alias> from source group
<key> to destination <user-or-group-key>, allocation <ContextBased|
LeastLoaded|RoundRobin>, priority <n>, usage <value>, condition <predicate>.

First describe the rule, Group, and User; resolve both endpoints; query all
rules for the source ordered by Priority and LastModifiedDate; and export a
known-good ConditionData shape. Show two phases: INSERT without ConditionData,
then UPDATE ConditionData. After approval, execute in non-production,
re-query ConditionData and ConditionId, and run match/non-match plus routing
checks. Do not assume undocumented tie-break behavior.
```

The fields, allocation values, and two-phase condition requirement are documented in Salesforce's task-assignment object and deployment references ([Salesforce task-assignment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmenttaskassignmentrule.htm), [Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

### 4.6 Scenario creation prompt

```text
Create a ProductFulfillmentScenario in <sandbox-alias> for product or
classification <key>, actions <Add|Amend|Cancel|NoChange|Renew>, step group
<key>, usage <value>, and condition <predicate>.

Describe ProductFulfillmentScenario and confirm the exact official fields:
ProductId or ProductClassificationId, FulfillmentStepDefnGroupId, Action,
UsageType, ConditionData. Reject Product2Id, FulfillmentStepDefinitionId, or
IsActive unless target describe independently exposes them.
Query overlapping scenarios. Show INSERT without ConditionData and subsequent
UPDATE. Re-query ConditionData and ScenarioRule, then test each selected action
and both condition outcomes.
```

The explicit rejection matters because a community DRO skill uses `Product2Id`, `FulfillmentStepDefinitionId`, and `IsActive`, while Salesforce's object reference documents `ProductId`, `FulfillmentStepDefnGroupId`, and no per-record `IsActive` field ([community DRO skill](https://github.com/lzdravkov/rlm-skills/blob/e891e839b697e391df9532d29d2a029cc52fb4f7/skills/rlm-dynamic-revenue-orchestrator/references/dro-objects-reference.md), [Salesforce scenario reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentscenario.htm)).

### 4.7 Safe metadata deployment and agent hosting

Not every DRO artifact is a data record: referenced Flows, Expression Sets, Decision Matrices, context definitions, and other metadata can require source deployment before DML. The official Salesforce deployment skill prescribes `sf project deploy validate` for production and dry-run validation for non-production, while the development agent requires explicit confirmation before production deployment ([Salesforce deploy-validation skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-deploy-validate/SKILL.md), [Salesforce development agent](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/agents/salesforce-dev.md)).

```bash
# Metadata dependencies only; validate before any actual deploy.
sf project deploy validate --target-org "$ORG" --source-dir force-app --json 
# For a sandbox, a dry run is the preferred preflight:
sf project deploy start --target-org "$ORG" --source-dir force-app --dry-run --json
```

Hosted MCP can provide the agent authenticated org context, and the Salesforce Development plugin can supply schema/query/deployment mechanics; the DRO prompt must still encode the data dependency graph and post-DML steps ([Salesforce Hosted MCP blog](https://developer.salesforce.com/blogs/2026/05/connect-claude-with-salesforce-hosted-mcp-servers), [Salesforce headless-development blog](https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin)).

## 5. Agentic rule modification

### 5.1 Amendment transaction

Use an append-and-verify workflow rather than an in-place blind edit:

1. Export the current rule family, dependent transformations, condition JSON, and governing BRE library/version.
2. Normalize and commit the baseline.
3. Query every scope and priority collision.
4. Produce a semantic before/after explanation.
5. Validate metadata dependencies.
6. Apply the smallest DML change in a sandbox.
7. Re-query and git-diff.
8. Refresh fallout decision tables when applicable.
9. Run positive, negative, boundary, and regression probes.
10. Promote through an ordered, auditable package.

### 5.2 Versioning and activation control

Salesforce's migration note says the active context rule-library version must point to the current DRO Admin context definition; when the context definition changes, clone the **latest** version, because cloning an older version can omit newer rule sets and cause evaluation to return false, then deactivate the old version and activate the clone ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

That BRE-library lifecycle is the verified activation gate. `IsActive` on `ProductFulfillmentDecompRule`, `ProductDecompEnrichmentRule`, `FulfillmentFalloutRule`, `FulfillmentTaskAssignmentRule`, or `ProductFulfillmentScenario` is **Unverified and absent from the reviewed official field tables** ([Salesforce decomposition-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm), [Salesforce fallout-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentfalloutrule.htm), [Salesforce task-assignment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmenttaskassignmentrule.htm)).

### 5.3 Priority and condition reordering

For decomposition, lower numeric priority has precedence for same-target writes, with most recently modified as the tie-breaker; therefore editing any tied record can silently change behavior even when `Priority` is untouched ([Salesforce decomposition guidance](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)). Use sparse increments such as 100, 200, 300 to permit insertion; treat a priority renumber as a behavior change; and prohibit ties unless a regression test proves the desired result.

For task assignment, `Priority` is documented but equivalent detailed tie-break semantics were not found in the reviewed public docs, so preserve uniqueness and mark tie behavior **Unverified** ([Salesforce task-assignment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmenttaskassignmentrule.htm)).

When editing conditions:

- compare normalized abstract predicates, not only escaped JSON strings;
- verify every attribute code and picklist value in the target before update;
- preserve the two-phase rule only for creation, but re-query the related rule field after any later condition update;
- test `true`, `false`, null/missing, boundary, and mutually overlapping inputs.

Salesforce's migration guidance explicitly makes target attributes and picklist values prerequisites for condition resolution ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

### 5.4 Deployment-order changes

For a new dependency, deploy references before consumers. The documented decomposition order is `ProductFulfillmentDecompRule` → `ValTfrmGrp` → `ValTfrm` → `ProductDecompEnrichmentRule` → `ProdtDecompEnrchVarMap`; the orchestration segment places `ProductFulfillmentScenario` before workspaces, then `FulfillmentFalloutRule`, jeopardy rules, and `FulfillmentTaskAssignmentRule` ([Salesforce deployment order](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)). For deletion, reason in reverse dependency order and require a reference query before each delete.

### 5.5 Fallout migration and refresh

A fallout change is incomplete until the Fulfillment Fallout Rules decision table is refreshed, and a migration pipeline should record who refreshed it, when, in which org, and which post-refresh probe passed ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)). If the interface is manual, the agent should emit a blocking checklist item rather than silently skip it. If an organization automates the UI or an internal endpoint, label that implementation organization-specific until Salesforce documents it as supported.

## 6. Best practices

### 6.1 Least privilege

Salesforce's permission-set catalog distinguishes a complete design-time **Fulfillment Designer**, a runtime-oriented **Fulfillment Manager/Operator**, and **DRO Admin User** with broad DRO access ([Salesforce Revenue Cloud permission sets](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5)). Salesforce's field reference also exposes user-permission flags `PermissionsDFOManagerOperatorUser`, `PermissionsDfoAdminUser`, and `PermissionsDFODesignerUser`, confirming the underlying capability names ([Salesforce User permission field reference](https://developer.salesforce.com/docs/atlas.en-us.sfFieldRef.meta/sfFieldRef/salesforce_field_reference_UserPermissionAccess.htm)).

Use `DFOManagerOperatorUser` for runtime operations and `DfoAdminUser` only where design-time administration is actually required, but verify the assignable permission-set API names in each org before scripting because the Help page primarily presents UI labels. Do not grant `DfoAdminUser` merely to make an agent's first failing command pass.

### 6.2 Verification-first prompting

Every prompt should include: “Describe before querying; query before generating; assert target records before writing; fail closed on an unknown field.” This is aligned with the official Salesforce SOQL skill's explicit instruction to verify identifiers with describe and not guess ([Salesforce SOQL skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-soql-query/SKILL.md)).

### 6.3 Plan mode before mutation

Claude Code recommends exploration and planning before implementation for nontrivial work, and Salesforce's published Headless 360 demonstration used a plan-first flow before file changes and deployment ([Claude Code best practices](https://code.claude.com/docs/en/best-practices), [Salesforce Developers Headless 360 video](https://www.youtube.com/watch?v=lbMCBJsaERk)). Require plan mode for all rule creates, priority edits, JSON condition changes, deletes, activation changes, and production targets.

### 6.4 Hooks and guardrails

Claude Code hooks can deterministically inspect tool calls and block them before execution ([Claude Code hooks](https://code.claude.com/docs/en/hooks)). Salesforce AI Research's Agentforce ADLC repository ships `PreToolUse` guardrails and a plan-oriented orchestrator, including checks around dangerous deployment behavior and hard-coded IDs; it is Agentforce-specific, but the hook pattern transfers cleanly to DRO ([Agentforce ADLC hooks](https://github.com/SalesforceAIResearch/agentforce-adlc/blob/b280f6346fa70aaf4fbd0e0bc5d18582c1fb039a/hooks/hooks.json), [Agentforce ADLC guardrails](https://github.com/SalesforceAIResearch/agentforce-adlc/blob/b280f6346fa70aaf4fbd0e0bc5d18582c1fb039a/shared/hooks/scripts/guardrails.py)).

Recommended blocking rules:

- reject org aliases matching production patterns unless a change ticket and confirmation token are present;
- reject `sf data delete`, `sf project delete`, destructive manifests, and unscoped bulk operations by default;
- reject `deploy start` unless a matching validation artifact exists and is fresh;
- reject DRO condition writes when no describe artifact and source condition specimen exist;
- reject copied Salesforce IDs unless the plan contains a verified source→target key map;
- reject fallout completion if decision-table refresh evidence is absent;
- reject direct `ProdtDecompEnrchVarMap` mutation unless target capabilities prove support.

### 6.5 Small-diff iteration

Salesforce's Revenue Cloud implementation guidance recommends starting with a small usable catalog and iterating rather than encoding every edge case immediately ([Salesforce Developers Revenue Cloud tips](https://www.youtube.com/watch?v=wmGq7rox-b4)). Apply the same principle at the rules layer: one scope, one condition delta, one mapping family, one test matrix, one reviewed diff.

### 6.6 “Assert with SOQL before generating the rule”

The agent should emit the assertion query and its result before it emits create/update syntax. At minimum, prove:

- exactly one source and destination product/classification;
- exactly one referenced step group, queue/user, transform group, expression set, flow, and integration definition;
- no unintended same-scope/same-priority collision;
- every condition attribute code and picklist value exists;
- no identifier field contains a source-org ID after migration.

These checks operationalize Salesforce's target-prerequisite and identifier-remediation guidance ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).

### 6.7 Non-production targeting and refresh discipline

Use a scratch org or sandbox for first mutation, print the resolved target in every command log, and deny production by hook. The official Salesforce development agent requires explicit production confirmation and validation ([Salesforce development agent](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/agents/salesforce-dev.md)). Make fallout refresh a blocking deployment phase rather than a release-note reminder ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)).

## 7. User experience and community signal

### 7.1 Signal map

| Source | What users/practitioners report | Relevance boundary |
|---|---|---|
| Salesforce Ben | Admin-oriented Claude Code work can retrieve org metadata, accelerate building, and benefit from explicit safeguards and incremental verification ([Salesforce Ben](https://www.salesforceben.com/10-lessons-for-admins-using-claude-code-to-build-salesforce-flows/)). | General Salesforce metadata/Flow evidence; not DRO rules. |
| Salesforce Ben, Data Cloud | Claude Code plus MCP is presented as a way to ground implementation work in a Salesforce environment ([Salesforce Ben](https://www.salesforceben.com/implementing-salesforce-data-cloud-with-claude-code-and-mcp/)). | General MCP pattern; different product surface. |
| SalesforceDevops.net | Headless development and agent skills widen the gap between disciplined, tool-grounded builders and users relying on opaque prompting ([SalesforceDevops.net](https://salesforcedevops.net/index.php/2026/04/15/tdx-2026-reporters-notebook-salesforce-goes-headless-and-widens-the-builder-gap/)). | General Salesforce development commentary. |
| SalesforceMonday | Plan mode is emphasized before Salesforce metadata changes, aligning with the need to inspect complex dependencies first ([SalesforceMonday](https://salesforcemonday.com/2026/05/11/claude-code-for-salesforce-development/)). | General Salesforce agent workflow. |
| r/salesforce | Practitioners report productivity gains when agents can run tests and iterate, but also weaker outcomes from broad, underspecified prompts ([r/salesforce](https://www.reddit.com/r/salesforce/comments/1nowfc9/codex_salesforce_is_pretty_game_changing/)). | Anecdotal; not independently verified and not DRO-specific. |
| r/SalesforceDeveloper | Discussion comparing Cursor and Claude Code includes skepticism about reliability and the need for developer oversight ([r/SalesforceDeveloper](https://www.reddit.com/r/SalesforceDeveloper/comments/1lfgi2h/what_is_better_for_salesforce_development_cursor/)). | Anecdotal general development signal. |
| r/Codex | A user reports an agent issuing an `sfdx delete from project and org` command, illustrating destructive-tool risk ([r/Codex](https://www.reddit.com/r/codex/comments/1ol3iaj/codex_just_ran_sfdx_delete_from_project_and_org/)). | Anecdotal incident; directly relevant to guardrails. |
| Salesforce Developers YouTube | A Headless 360 build demonstrates planning, skill use, smoke testing, diagnosis, redeployment, and regression checks ([Salesforce Developers](https://www.youtube.com/watch?v=lbMCBJsaERk)). | Official demonstration, but not DRO. |
| Salesforce Developers DRO demo | DRO's visual decomposition and mapping surfaces make commercial→technical transformations and fallout observable to administrators ([Salesforce Developers](https://www.youtube.com/watch?v=WOtsN9Qyb1o)). | DRO-specific product UX; no coding-agent authoring. |
| Salesforce+ | A Dreamforce order-orchestration recording exists, but the reviewed public page did not expose evidence of coding-agent DRO-rule management ([Salesforce+](https://www.salesforce.com/plus/experience/dreamforce_2025/series/sales_at_dreamforce_2025/episode/episode-s1e28)). | DRO/order-orchestration context only. |

### 7.2 GitHub-side skill ecosystem

| Repository | Concrete value | DRO-rules verdict |
|---|---|---|
| `forcedotcom/sf-skills` | Official generic Salesforce development, SOQL/schema, deployment validation, and gating patterns; compatible with multiple coding agents ([README](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/README.md)). | No requested DRO rule-object skill found at reviewed commit. |
| `SalesforceAIResearch/agentforce-adlc` | Plan-oriented orchestration, hooks, guardrails, and org-description utilities ([ADLC README](https://github.com/SalesforceAIResearch/agentforce-adlc/blob/b280f6346fa70aaf4fbd0e0bc5d18582c1fb039a/README.md), [org describe script](https://github.com/SalesforceAIResearch/agentforce-adlc/blob/b280f6346fa70aaf4fbd0e0bc5d18582c1fb039a/scripts/org_describe.py)). | Transferable control plane; Agentforce, not DRO. |
| `lzdravkov/rlm-skills` | Includes a named DRO skill and deployment references ([DRO skill](https://github.com/lzdravkov/rlm-skills/blob/e891e839b697e391df9532d29d2a029cc52fb4f7/skills/rlm-dynamic-revenue-orchestrator/SKILL.md)). | Useful orientation, but its scenario field example conflicts with official schema; validate every field. |
| `bgaldino/rlm-base-dev` | Shipped SFDMU extraction/upsert dataset and Summer ’26 documentation mirror ([SFDMU export](https://github.com/bgaldino/rlm-base-dev/blob/7999013b2b505e0d20b78021583ff99a8ae59566/datasets/sfdmu/mfg/en-US/mfg-dro/export.json)). | Best concrete public rules-data pattern; sample is not portable unchanged. |
| `arohitu/salesforce-revenue-cloud-skills` | Skills for product catalog, pricing, configuration, transactions, and decision tables ([README](https://github.com/arohitu/salesforce-revenue-cloud-skills/blob/6c9ce0673225559adbedf8555ba8404d2d01265a/README.md)). | No requested DRO rule objects found; generic decision-table support may assist inspection. |
| `PranavNagrecha/AwesomeSalesforceSkills` | Broad Salesforce skills inventory and MCP-oriented org grounding ([README](https://github.com/PranavNagrecha/AwesomeSalesforceSkills/blob/4af2a2ccdaf10f9271745d5ede28ccafd905018e/README.md)). | No requested DRO rule-management implementation found. |
| `alexkwitko/Salesforce-Agentforce` | Broad Agentforce reference material ([README](https://github.com/alexkwitko/Salesforce-Agentforce/blob/70486e75eb82db91f9247ed014cea1ca3a7fbcad/README.md)). | No requested DRO rule objects found. |

The required GitHub searches—repository searches for “DRO decomposition rules,” “revenue cloud advanced skills,” and “fulfillment fallout rule,” plus code searches for the four named SObjects and `ValTfrmGrp`—produced sparse results dominated by documentation mirrors and the repositories above. That absence is evidence of a public implementation gap, not evidence that private accelerators do not exist.

## 8. Proven techniques

### 8.1 Git-diff-driven review — shipped community pattern

`bgaldino/rlm-base-dev` stores SFDMU query definitions and extracted CSV by SObject, with ordered upserts and semantic external IDs, making rule data reviewable in git ([SFDMU export configuration](https://github.com/bgaldino/rlm-base-dev/blob/7999013b2b505e0d20b78021583ff99a8ae59566/datasets/sfdmu/mfg/en-US/mfg-dro/export.json)). Adopt the structure, but replace source-specific IDs and extend it to include condition JSON and transformation children.

### 8.2 MCP-mediated org describe — official platform pattern

Salesforce Hosted MCP gives Claude authenticated access to Salesforce-hosted tools, while the official development skills require schema verification ([Salesforce Hosted MCP setup](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/claude.html), [Salesforce SOQL skill](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/skills/platform-soql-query/SKILL.md)). Use MCP/CLI describe to generate an allowed-field manifest before the model composes SOQL or DML.

### 8.3 Plan-mode-first slash commands — official general guidance, DRO adaptation

Claude Code's best-practices guide supports an explore/plan/implement/verify separation, and Salesforce's demonstration follows that loop ([Claude Code best practices](https://code.claude.com/docs/en/best-practices), [Salesforce Developers](https://www.youtube.com/watch?v=lbMCBJsaERk)). Package `/dro-explain-rules`, `/dro-plan-create`, `/dro-plan-amend`, and `/dro-verify-migration` as read-only-first commands whose write phase requires explicit approval.

### 8.4 Hooks blocking production and unvalidated deploys — shipped pattern

The official Salesforce development agent requires explicit production confirmation and supplies a deploy gate, while Agentforce ADLC shows pre-tool guardrails in code ([Salesforce deploy gate](https://github.com/forcedotcom/sf-skills/blob/bfca400cddef350aba61abc1be548cf9f1f1e2fc/plugins/builder/salesforce-development/scripts/sf-deploy-gate), [ADLC guardrails](https://github.com/SalesforceAIResearch/agentforce-adlc/blob/b280f6346fa70aaf4fbd0e0bc5d18582c1fb039a/shared/hooks/scripts/guardrails.py)). Extend that shipped pattern to demand a successful `sf project deploy validate` or sandbox `--dry-run` artifact before `deploy start`, and to block production aliases by default.

### 8.5 Decision-table refresh checkpoint — official operational technique

Salesforce documents refresh after fallout changes and migration ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5), [Salesforce migration guidance](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). A deploy script can reliably enforce a **checkpoint**, evidence capture, and failure if missing; a supported automated refresh command remains **Unverified**.

### 8.6 Condition-JSON generation from describe plus a specimen — derived from official constraints

Salesforce documents the JSON fields, UPDATE-only reference creation, and target attribute/picklist prerequisites ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)). The safe generation technique is therefore: describe → query known-good same-release specimen → resolve semantic identifiers → produce normalized JSON diff → human approval → update → re-query related rule field. Generating JSON from prose without a specimen is not evidence-based.

### 8.7 Rule-authoring skills in the Salesforce plugin — gap, not a proven feature

The plugin has proven generic authoring primitives, but a repository search at commit `bfca400…` did not find `ProductFulfillmentDecompRule`, `ProductDecompEnrichmentRule`, `FulfillmentFalloutRule`, or `FulfillmentTaskAssignmentRule`. A dedicated DRO rules skill should be added externally or contributed; claiming it is already bundled would overstate the evidence ([`forcedotcom/sf-skills` repository](https://github.com/forcedotcom/sf-skills/tree/bfca400cddef350aba61abc1be548cf9f1f1e2fc)).

## 9. Open questions and gaps

1. **Condition JSON schema stability — Unverified.** Public docs identify `ConditionData` and the two-phase behavior but do not publish a complete, versioned JSON schema for agent generation; use target-org specimens and contract tests ([Salesforce additional deployment information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).
2. **Automated fallout decision-table refresh — Unverified.** The refresh requirement is official, but no supported CLI/REST action was established in the reviewed public sources ([Salesforce fallout design](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)).
3. **`ProdtDecompEnrchVarMap` mutation — Unverified.** The object page omits create/update/upsert from supported calls despite one field's create/update properties; treat the aggregate as platform-managed until Salesforce clarifies ([Salesforce variable-map reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_prodtdecompenrchvarmap.htm)).
4. **Per-record activation — Unverified.** No `IsActive` appears in the reviewed field references for the required rule families; library-version activation is the documented control ([Salesforce migration guidance](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).
5. **Task-assignment tie-break semantics — Unverified.** `Priority` is documented, but a same-priority tie rule equivalent to decomposition's modification-time rule was not found ([Salesforce task-assignment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmenttaskassignmentrule.htm)).
6. **Atomic rollback across metadata, data, BRE references, and decision tables — Unverified.** Public guidance gives ordering and repair steps but no single transaction spanning all artifact types.
7. **Official DRO-specific agent skill — absent in reviewed public commit.** Official skills provide generic Salesforce primitives, while community DRO skills are incomplete and can conflict with official fields ([Salesforce skills repository](https://github.com/forcedotcom/sf-skills/tree/bfca400cddef350aba61abc1be548cf9f1f1e2fc), [community DRO reference](https://github.com/lzdravkov/rlm-skills/blob/e891e839b697e391df9532d29d2a029cc52fb4f7/skills/rlm-dynamic-revenue-orchestrator/references/dro-objects-reference.md)).
8. **End-to-end production case studies — not found.** The reviewed community, YouTube, Salesforce+, and GitHub sources show DRO UX or general Salesforce-agent practice, but not a documented production pipeline that creates all five rule families, attaches conditions, refreshes fallout tables, and proves runtime behavior.
9. **Release drift.** Several important identifier and condition fields appear only in API 65.0/66.0, so any reusable skill must branch on target describe rather than assume Summer ’26 shape ([Salesforce decomposition-rule reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm), [Salesforce scenario reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentscenario.htm)).

## Recommended operating model

The defensible agent architecture is a **read-evaluate-plan-execute-verify pipeline**: official Salesforce skills and Hosted MCP for org grounding; a custom DRO skill for object order, condition attachment, and semantic validation; deterministic hooks for target and command safety; SFDMU or equivalent normalized exports for review; and a human-approved checkpoint for BRE activation and fallout-table refresh. This model exploits agent speed without allowing the model to become the schema authority.

**Retrieval keywords:** Salesforce DRO rules, Dynamic Revenue Orchestrator, ProductFulfillmentDecompRule, ProductDecompEnrichmentRule, ProdtDecompEnrchVarMap, ValTfrmGrp, ValTfrm, FulfillmentFalloutRule, FulfillmentTaskAssignmentRule, ProductFulfillmentScenario, ConditionData, ExecuteOnRule, ScenarioRule, decision table refresh, Salesforce CLI, Claude Code, Cursor, Codex, Salesforce Development plugin, Hosted MCP, Agentforce ADLC, SFDMU, rule migration, two-phase insert update

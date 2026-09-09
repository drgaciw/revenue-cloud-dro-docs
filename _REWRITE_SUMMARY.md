# DRO Documentation Full Rewrite Summary

## Result

Rewrote all 21 requested source documents into agent-consumable Markdown under `/home/user/workspace/dro-docs-improved/`. The source directory was not modified.

Every rewritten document includes YAML retrieval metadata, the stable section skeleton, agent triggers, an agent playbook, guardrails, verification steps, official references, relative cross-links, and retrieval keywords.

## Files rewritten

- `README.md`
- `agentic-dro.md`
- `agentic-skills-gap-analysis.md`
- `agentic-skills-inventory.md`
- `agentic-tooling.md`
- `amendments-renewals.md`
- `audit-system.md`
- `azure-middleware.md`
- `chaos-testing.md`
- `decomposition-viewer.md`
- `dro-mapping.md`
- `external-agentic-skills.md`
- `fulfillment-orchestration.md`
- `index.md`
- `interface-coverage.md`
- `licensing.md`
- `nfr-parking-lot.md`
- `observability.md`
- `rlm-skills-porting.md`
- `token-auditability.md`
- `wrapper-patterns.md`

## Key sources cited

- [Dynamic Revenue Orchestrator Essentials](https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Dynamic Revenue Orchestrator Objects Deployment Reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)
- [Dynamic Revenue Orchestrator Additional Deployment Information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)
- [Define How a Product Decomposes](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)
- [Design Your Order Orchestration](https://help.salesforce.com/s/articleView?id=ind.dro_design_time_orchestration.htm&language=en_US&type=5)
- [Fulfillment Step Types](https://help.salesforce.com/s/articleView?language=en_US&id=sf.dro_fulfillment_step_types.htm&type=5)
- [Monitor Decomposition During Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)
- [Callouts in Dynamic Revenue Orchestrator](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_callouts_overview.htm)
- [Build Custom MCP Servers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)
- [Salesforce CLI deployment reference](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)
- [Amending, Renewing, and Canceling Assets](https://help.salesforce.com/s/articleView?id=ind.qocal_manage_assets_in_revenue_lifecycle_management.htm&language=en_US&type=5)
- [Fallout Design and Management](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)
- [SLA Jeopardy Administration](https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5)
- Microsoft Learn documentation for Azure Event Grid, Service Bus, and Key Vault where the subject is Azure-specific.

## Unverified items for owner review

- **amendments-renewals.md:** The exact API/event that should trigger the project token-revocation path has not been selected. Confirm supported Revenue Cloud asset-management APIs in the target release.
- **audit-system.md:** The event schema, storage technology, retention period, and “download receipt” semantics are project architecture decisions, not Salesforce features.
- **external-agentic-skills.md:** Earlier source files claimed exact skill names, installation commands, object counts, stars, and feature coverage for these third-party repositories. Confirm against a pinned commit before reuse.
- **licensing.md:** Token TTL, revocation SLA, deny-list design, Nexus content-selector semantics, and automatic future-version inclusion require security/product approval.
- **token-auditability.md:** Nexus Cloud tenancy, content-selector configuration, token format, TTL, and revocation implementation must be confirmed with the deployed services.

## Resolved items (previously flagged Unverified)

- **wrapper-patterns.md / interface-coverage.md / dro-mapping.md — DRO interface question.** Every DRO design-time and runtime entity is a standard SObject with `create()`/`update()`/`upsert()`/`query()` supported calls per the [DRO Standard Objects reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm). Interface = **standard Data API** (REST/SOAP/Apex DML). Metadata API is used only for the Setup surface documented in the [DRO Metadata deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm). Rule-set references and condition data use INSERT + UPDATE per the [Additional Deployment Information page](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm). Tooling API is not used for DRO record data.
- **decomposition-viewer.md — Viewer column-to-object mapping.** Viewer columns are backed by `FulfillmentOrderLineItem`, `FulfillmentLineSourceRel` (with `SupplementalAction` valid values `Add`/`Amend`/`Cancel`/`NoChange`), and `FulfillmentLineAttribute`. Action valid values (`Add`/`Amend`/`NoChange`/`Renew`/`Cancel`) come from the [Fulfillment Order Line Item Actions page](https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_order_line_item_actions.htm&language=en_US&type=5).
- **decomposition-viewer.md — monitoring permission.** Per the [Revenue Management permission-sets table](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5), the least-privilege permission set that grants “view … decomposition UI” is `DFOManagerOperatorUser` (Fulfillment Manager/Operator). `DfoAdminUser` also grants it. Both live under the `DynamicRevenueOrchestratorUserPsl` permission set license.

## High-priority confirmations (remaining)

1. Select the supported Revenue Cloud event/API that initiates project token revocation for amendments, renewals, and cancellations.
2. Approve the project-specific entitlement JSON schema, token TTL/revocation SLA, Nexus isolation model, and OCI-label authority boundary.
3. Re-audit third-party skill repositories at pinned commits before relying on claimed skill names, installation commands, object counts, or feature coverage.

## Validation performed

- Confirmed a one-to-one filename match for all 21 source files.
- Confirmed all required headings and retrieval-keyword lines.
- Confirmed relative Markdown links resolve.
- Confirmed no prohibited web-interaction terminology appears.
- Confirmed every body citation URL is also present in the file’s frontmatter `sources` list.
- Kept project-specific architecture separate from native Salesforce behavior and marked unresolved details as `Unverified`.

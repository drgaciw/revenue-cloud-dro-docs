---
title: "Licensing and Entitlement Schema Evolution"
description: "Keep project entitlement JSON, token revocation, and repository access aligned with DRO outputs without presenting them as native Salesforce licensing features."
agent_use: "Load when changing entitlement schema, token policy, version inclusion, revocation, or licensed technical-product mapping."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["dro-mapping", "token-auditability", "azure-middleware", "amendments-renewals"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm"]
---

## Purpose

Define the boundary between Salesforce fulfillment data and the project-owned entitlement contract. Salesforce provides technical products, mapped fulfillment-line data, and fulfillment assets; the entitlement JSON and token service are custom.

## When to use this doc (agent trigger conditions)

- “Change entitlement JSON.”
- “Add image/version constraints.”
- “Revoke repository access.”
- “Compare licensed and downloadable products.”

## Key concepts

- **Entitlement contract:** versioned project JSON consumed by downstream services.
- **Technical product:** Salesforce fulfillment representation.
- **Schema version:** explicit compatibility identifier.
- **Revocation:** project transition that blocks new authorization and records evidence.
- **OCI labels:** discovery metadata, not licensing authority.

## Data model & objects

Use `Product2`, `ProductFulfillmentDecompRule`, `FulfillmentLineAttribute`, `FulfillmentAsset`, and `FulfillmentAssetAttribute` as verified Salesforce-side entities. Map them to the custom entitlement schema through documented enrichment/value-transform configuration.

## Flow / sequence

1. Sales transaction decomposes to technical products.
2. Mapped values appear on fulfillment lines.
3. Approved middleware builds/version-validates entitlement JSON.
4. License/repository services enforce the contract.
5. Audit ledger records issuance, denial, use, expiry, and revocation.

## APIs & extension points

Salesforce Field and Attribute Mapping enriches target fulfillment data ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)). Exact middleware, license-system, Nexus, token, and OCI APIs are project-specific.

> **Unverified:** Token TTL, revocation SLA, deny-list design, Nexus content-selector semantics, and automatic future-version inclusion require security/product approval.

## Configuration & metadata

Store JSON Schema files in source control. Add fields compatibly when possible; use explicit schema versions for breaking changes. Define Salesforce-to-JSON mapping, required fields, normalization, and hashing.

## Agent playbook

1. Locate the schema and all consumer versions.
2. Trace each JSON field to a verified Salesforce source or project source.
3. Generate backward/forward compatibility tests.
4. Compare the technical-product set with repository paths and OCI labels.
5. Model expiry/revocation state transitions.
6. Require security and product approval for TTL/SLA/policy changes.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not call custom entitlement JSON a Salesforce standard schema.
- Do not treat OCI labels as authorization.
- Do not log secrets or reusable tokens.
- Do not add breaking fields without a migration plan.

## Verification & tests

1. JSON Schema validation for every fixture.
2. Consumer contract tests across supported versions.
3. Licensed/unlicensed technical-product tests.
4. Expiry and revocation tests.
5. Replay and duplicate issuance tests.
6. Audit-event completeness checks.

## References

- [Dynamic Revenue Orchestrator Essentials](https://help.salesforce.com/s/articleView?id=ind.dynamic_revenue_orchestration_essentials.htm&language=en_US&type=5)
- [Define Field and Attribute Mapping](https://help.salesforce.com/s/articleView?id=ind.dro_define_field_and_attribute_mapping.htm&language=en_US&type=5)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)

**Retrieval keywords:** licensing, entitlement JSON, schema evolution, revocation, token TTL, deny list, technical product, OCI labels

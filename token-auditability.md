---
title: "Token-Service Auditability"
description: "Specify auditable token lifecycle and negative-event evidence without storing credential material."
agent_use: "Load when implementing mint/revoke/deny events, correlation, multi-tenant access evidence, or token incident analysis."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["agentic-dro", "audit-system", "azure-middleware", "licensing", "observability"]
last_reviewed: 2026-09-09
sources: ["https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts_named_credentials.htm"]
---

## Purpose

Define project-owned token evidence linked to Salesforce fulfillment records. This is an integration control, not a standard DRO token feature.

## When to use this doc (agent trigger conditions)

- “Audit token mint/revoke.”
- “Trace a denied download.”
- “Prove tenant isolation.”
- “Investigate credential exposure.”

## Key concepts

- **Token fingerprint:** irreversible identifier safe for logs.
- **Scope:** allowed repository/path/action.
- **Negative event:** expiry, revocation, selector/scope mismatch, or denied request.
- **Chain of custody:** order → fulfillment → entitlement → authorization → repository outcome.

## Data model & objects

Reference Salesforce IDs such as order, `FulfillmentPlan`, `FulfillmentStep`, and relevant fulfillment line/asset records. Token and download records live in the project audit system.

## Flow / sequence

1. Validate entitlement and scope.
2. Mint a short-lived credential according to approved policy.
3. Log fingerprint, issuer, scope hash, correlation ID, and expiry—not the token.
4. Record use/deny outcomes.
5. Revoke or expire and append evidence.

## APIs & extension points

Use Named Credentials for Salesforce callout authentication ([Apex guide](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts_named_credentials.htm)). Token-service and Nexus APIs are project-specific.

> **Unverified:** Nexus Cloud tenancy, content-selector configuration, token format, TTL, and revocation implementation must be confirmed with the deployed services.

## Configuration & metadata

Define immutable audit event types: `TOKEN_MINTED`, `TOKEN_DENIED`, `TOKEN_REVOKED`, `TOKEN_EXPIRED`, `DOWNLOAD_SUCCEEDED`, `DOWNLOAD_FAILED`. Names are project conventions, not Salesforce APIs.

## Agent playbook

1. Search by correlation ID and token fingerprint.
2. Join to Salesforce records by stored IDs.
3. Check entitlement effective window and scope.
4. Identify first missing or contradictory transition.
5. Return evidence and recommended response; redact credentials.
6. Propose replay/revocation only with approval.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Never persist or print raw tokens.
- Never infer tenant isolation from a successful request alone.
- Never reuse an authorization result across a changed entitlement.

## Verification & tests

1. Secret-detection test on logs.
2. Expired/revoked/out-of-scope tests.
3. Duplicate mint idempotency test.
4. Cross-tenant isolation test.
5. Full chain-of-custody query test.
6. Incident export with redaction.

## References

- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Named Credentials as Callout Endpoints](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts_named_credentials.htm)

**Retrieval keywords:** token audit, token fingerprint, mint, revoke, deny, negative event, chain of custody, tenant isolation

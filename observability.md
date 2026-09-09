---
title: "Observability"
description: "Build correlation-driven views and alerts spanning DRO runtime state and project middleware without conflating telemetry with audit evidence."
agent_use: "Load when defining dashboards, logs, metrics, traces, alerts, or SLOs."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["audit-system", "token-auditability", "azure-middleware", "nfr-parking-lot"]
last_reviewed: 2026-09-09
sources: ["https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5"]
---

## Purpose

Use Salesforce jeopardy and fallout for platform operations, then join them with external telemetry through stable correlation IDs.

## When to use this doc (agent trigger conditions)

- “Build a dashboard or alert.”
- “Find stuck/late fulfillment.”
- “Trace one entitlement end to end.”

## Key concepts

- **Jeopardy:** warning before Estimated Duration is exceeded.
- **Fallout:** failure/retry/queue handling.
- **Metric/log/trace:** operational telemetry.
- **Audit event:** durable evidence; do not use mutable dashboards as the ledger.

## Data model & objects

Primary Salesforce dimensions: `FulfillmentPlan`, `FulfillmentStep`, `FulfillmentStepSource`, `FulfillmentFalloutRule`, and `FulfillmentStepJeopardyRule`. External dimensions: correlation ID, idempotency key, schema version, service, operation, and outcome.

## Flow / sequence

1. Emit correlation at request entry.
2. Persist Salesforce record IDs in downstream events.
3. Export logs/metrics/traces without secrets.
4. Join views by correlation ID.
5. Alert on documented state transitions and project SLOs.
6. Link to immutable audit evidence.

## APIs & extension points

Query supported objects through read-only SOQL/SObject tools. DRO jeopardy uses Estimated Duration and Jeopardy Threshold ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5)); fallout queues can contain fatally failed `FulfillmentStep` records ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)).

## Configuration & metadata

Define metric names, cardinality budgets, dashboards, alert ownership, runbooks, and retention outside Salesforce metadata unless using supported platform observability products.

## Agent playbook

1. Start with a user-visible failure or SLO.
2. Identify Salesforce and external signals.
3. Define low-cardinality metrics and structured logs.
4. Add correlation links and record IDs.
5. Create alerts with owner and runbook.
6. Test with controlled failures.
7. Review false positives.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not put order IDs, correlation IDs, or image paths into unbounded metric labels without a cardinality review.
- Do not alert on every retry.
- Do not log tokens or entitlement payloads.

## Verification & tests

1. Synthetic happy path.
2. Stuck-step and jeopardy threshold test.
3. Fatal fallout queue test.
4. Missing downstream outcome test.
5. Dashboard link-to-audit test.
6. Secret/redaction and cardinality checks.

## References

- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [SLA Jeopardy Administration](https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5)
- [Fallout Design and Management](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)

**Retrieval keywords:** observability, telemetry, correlation ID, dashboard, alert, jeopardy, fallout, stuck fulfillment, SLO

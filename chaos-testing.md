---
title: "Chaos Testing and Failure Injection"
description: "Run controlled resilience experiments against the DRO-to-middleware boundary and verify retries, fallout, idempotency, and evidence."
agent_use: "Load when planning failure injection, resilience tests, game days, or release gates."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["audit-system", "azure-middleware", "fulfillment-orchestration", "nfr-parking-lot", "observability"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dro_callout.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/atlas.en-us/platform_events.meta/platform_events/platform_event_apex_tests.htm", "https://learn.microsoft.com/en-us/azure/event-grid/delivery-and-retry"]
---

## Purpose

Test documented Salesforce failure surfaces and project-owned integration behavior in non-production environments.

## When to use this doc (agent trigger conditions)

- “Test middleware outage.”
- “Inject duplicate/delayed/failing fulfillment.”
- “Verify compensation or dead letters.”

## Key concepts

- **Steady-state hypothesis:** observable expected behavior before injection.
- **Blast radius:** bounded test scope.
- **Abort condition:** automatic stop.
- **Fallout/jeopardy:** native Salesforce operational responses.
- **Compensation:** explicit project or plan action, never assumed.

## Data model & objects

Observe `FulfillmentPlan`, `FulfillmentStep`, configured fallout/jeopardy rules, and audit events. Use synthetic products/orders and dedicated queues.

## Flow / sequence

1. Define steady state and expected records/events.
2. Bound org, tenant, products, duration, and volume.
3. Inject one fault.
4. Observe step state, retries, queue, jeopardy, downstream messages, and audit evidence.
5. Abort if thresholds breach.
6. Recover and verify convergence.

## APIs & extension points

Use `HttpCalloutMock` in Apex unit tests, platform-event test delivery patterns, sandbox callout stubs, and Azure delivery controls. Event Grid retry and dead-letter behavior is documented by Microsoft ([delivery guide](https://learn.microsoft.com/en-us/azure/event-grid/delivery-and-retry)).

## Configuration & metadata

Keep experiment manifests in source control: hypothesis, injection, target, duration, abort, expected Salesforce state, expected external state, recovery, and evidence links.

## Agent playbook

1. Refuse production by default.
2. Confirm rollback and on-call owner.
3. Seed synthetic order and expected decomposition.
4. Apply one fault: timeout, 5xx, inactive integration, duplicate message, malformed payload, or delayed audit.
5. Capture timestamps and record IDs.
6. Recover explicitly.
7. File gaps and add regression tests.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- No uncontrolled production experiments.
- No multi-fault test before single-fault behavior is understood.
- No assertion that a custom “compensation event” is native DRO behavior.

## Verification & tests

- Token/authorization succeeds but repository grant fails: verify approved compensation path.
- Integration inactive: verify callout hold behavior.
- Retry exhaustion: verify fallout queue/dead-letter evidence.
- Malformed entitlement: verify fail-closed behavior and no retry storm.
- Late completion: verify jeopardy and eventual reconciliation.
- Duplicate delivery: verify one external side effect.

## References

- [Callout Fulfillment Step](https://help.salesforce.com/s/articleView?id=ind.dro_callout.htm&language=en_US&type=5)
- [Fallout Design and Management](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)
- [SLA Jeopardy Administration](https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5)
- [Testing Platform Events in Apex](https://developer.salesforce.com/docs/atlas.en-us/platform_events.meta/platform_events/platform_event_apex_tests.htm)
- [Azure Event Grid Delivery and Retry](https://learn.microsoft.com/en-us/azure/event-grid/delivery-and-retry)

**Retrieval keywords:** chaos testing, failure injection, game day, fallout, jeopardy, retry, dead letter, idempotency

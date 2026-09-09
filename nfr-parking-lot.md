---
title: "Non-Functional Requirements Parking Lot"
description: "Turn deferred reliability, performance, security, operability, and recovery concerns into measurable DRO release gates."
agent_use: "Load when defining NFRs, test budgets, SLOs, load tests, smoke tests, or capacity assumptions."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator"]
related: ["observability", "chaos-testing", "fulfillment-orchestration"]
last_reviewed: 2026-09-09
sources: ["https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5", "https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html"]
---

## Purpose

Keep deferred NFRs visible without inventing Salesforce limits. Every numeric target must come from business requirements, org limits, or measured baselines.

## When to use this doc (agent trigger conditions)

- “Define performance/load/smoke tests.”
- “Set fulfillment SLOs.”
- “Plan capacity or disaster recovery.”

## Key concepts

- **SLO/SLI:** target and measured indicator.
- **Governor/org limit:** platform constraint confirmed for the org/release.
- **Throughput/latency/backlog:** workload dimensions.
- **RTO/RPO:** recovery objectives supplied by owners, not inferred.

## Data model & objects

Measure volumes and latency around `FulfillmentPlan`, `FulfillmentStep`, fallout queues, callout middleware, and the audit ledger. Avoid adding unindexed/high-cardinality fields without design review.

## Flow / sequence

1. Inventory critical journeys.
2. Capture baseline.
3. Obtain business targets and org limits.
4. Define smoke, load, soak, spike, and recovery tests.
5. Run in production-like non-production.
6. Analyze bottlenecks and revise.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Use Salesforce jeopardy rules for configured step lateness visibility; they do not replace end-to-end SLO measurement.

## Configuration & metadata

Parking-lot matrix: availability, latency, throughput, backlog, resilience, recovery, security, privacy, audit retention, observability, deployment safety, and cost. Assign owner, target, measurement, environment, and due release.

## Agent playbook

1. Convert each vague NFR into measurable acceptance criteria.
2. Query current org limits and measured baseline.
3. Build synthetic data without customer secrets.
4. Separate Salesforce transaction limits from middleware limits.
5. Execute incrementally.
6. Save reports and update release gates.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not invent throughput or SLA numbers.
- Do not run bulk/load tests against production without formal approval.
- Do not use average latency alone; include tails and failures.

## Verification & tests

1. Deployment smoke test.
2. Representative decomposition/orchestration latency.
3. Callout throughput and backpressure.
4. Retry/fallout saturation.
5. Audit ingestion/reconciliation lag.
6. Recovery from downstream outage.
7. Permission and data-leakage checks.

## References

- [SLA Jeopardy Administration](https://help.salesforce.com/s/articleView?id=ind.dro_sla_jeopardy_administration.htm&language=en_US&type=5)
- [Fallout Design and Management](https://help.salesforce.com/s/articleView?id=ind.dro_fallout_design_and_management.htm&language=en_US&type=5)
- [Salesforce CLI project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)

**Retrieval keywords:** NFR, performance, load test, smoke test, soak test, SLO, throughput, latency, recovery

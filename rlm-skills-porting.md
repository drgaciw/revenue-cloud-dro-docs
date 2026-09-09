---
title: "Porting RLM Skills to CLI Agent Format"
description: "Port selected schema and procedure skills into a consistent SKILL.md package without importing stale org assumptions."
agent_use: "Load when converting a third-party Cursor/Claude/Codex skill for this project."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator", "Salesforce Platform"]
related: ["agentic-skills-inventory", "external-agentic-skills", "agentic-skills-gap-analysis"]
last_reviewed: 2026-09-09
sources: ["https://github.com/forcedotcom/sf-skills", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html", "https://github.com/lu-zhengda/skill-port"]
---

## Purpose

Port selectively: schema grounding and generic skill mechanics first; keep proprietary decomposition, entitlement, OCI, and Nexus logic in custom project skills.

## When to use this doc (agent trigger conditions)

- “Port this RLM skill.”
- “Convert Cursor rules to SKILL.md.”
- “Strip CCI/SFDMU assumptions.”
- “Add evals and triggers.”

## Key concepts

- **Mechanical port:** format/frontmatter conversion.
- **Semantic port:** API, release, and workflow revalidation.
- **Harness-neutral skill:** commands and references do not assume one editor.
- **Canonical taxonomy:** use the six names in [Skills Inventory](./agentic-skills-inventory.md).

## Data model & objects

Any ported schema content must be checked against the [official object reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm) and a target-org describe. Remove fields or objects that cannot be verified.

## Flow / sequence

1. Pin source commit and license.
2. Inventory files, scripts, hooks, and dependencies.
3. Convert frontmatter and directory layout.
4. Rewrite triggers to canonical names.
5. Replace org-specific commands with `sf` CLI parameters.
6. Revalidate every Salesforce API claim.
7. Add evals and run without the source editor.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Use `npx skills add forcedotcom/sf-skills` only as documented by the official repository; third-party conversion tools remain unverified until pinned and reviewed.

## Configuration & metadata

Recommended package: `SKILL.md`, `references/official-sources.md`, `evals/cases.jsonl`, and optional scripts with locked dependencies. Preserve attribution and license notices.

## Agent playbook

1. Produce a port manifest: kept, rewritten, dropped.
2. Remove CCI, SFDMU, org-name, and API-version assumptions unless they are explicit project requirements.
3. Add `agent_use`/trigger vocabulary.
4. Link official Salesforce references.
5. Add no-org and missing-field failure behavior.
6. Run evals in the intended harness.
7. Submit the port through code review.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not bulk-convert a library with unknown license or scripts.
- Do not preserve stale API v61-v64 claims as current without target-org checks.
- Do not rename canonical project skills during a port.

## Verification & tests

1. Lint frontmatter.
2. Confirm folder name matches skill name.
3. Run trigger selection tests.
4. Run golden outputs with and without org access.
5. Verify all links and command examples.
6. Diff semantic outputs against the source skill.

## References

- [Salesforce Skills Library](https://github.com/forcedotcom/sf-skills)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- [Salesforce CLI project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)
- Candidate converter: [`lu-zhengda/skill-port`](https://github.com/lu-zhengda/skill-port) (third party; pin and review before use).

**Retrieval keywords:** port skills, SKILL.md, Cursor, Claude Code, Codex, semantic port, trigger, eval, schema grounding

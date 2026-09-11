---
title: "Agentic Tooling for DRO Engineering"
description: "Select Salesforce CLI, official Salesforce skills, Hosted MCP, and project wrappers for efficient, governed agent workflows."
agent_use: "Load when choosing tools, configuring an agent harness, or reducing unsafe direct-org access."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator", "Hosted MCP Servers"]
related: ["agentic-skills-inventory", "dro-claude-code-research", "dro-rules-management-research", "external-agentic-skills", "interface-coverage", "wrapper-patterns"]
last_reviewed: 2026-09-09
sources: ["https://github.com/forcedotcom/sf-skills", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html", "https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html", "https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_apex.html"]
---

## Purpose

Use a minimal stack: local source + `sf` CLI for change control, official Salesforce skills for platform procedures, Hosted MCP for approved live tools, and custom wrappers only for proven gaps.

## When to use this doc (agent trigger conditions)

- “Set up Claude Code/Cursor/Codex for DRO.”
- “Which MCP tools should be exposed?”
- “How should the agent deploy and test?”

## Key concepts

- **Authoring plane:** repository and CLI.
- **Inspection plane:** read-only SOQL/SObject MCP.
- **Action plane:** explicit invocable/Flow tools.
- **Progressive disclosure:** expose only tools required for the current task.

## Data model & objects

Tool access should start with describes and least-privilege reads for objects relevant to the task, such as `ProductFulfillmentDecompRule`, `FulfillmentPlan`, and `FulfillmentStep`.

## Flow / sequence

1. Agent edits source locally.
2. CLI runs lint/tests and dry-run deployment.
3. Read-only MCP/SOQL verifies org state.
4. Approved invocable tool performs a bounded action if required.
5. Agent records evidence in the PR.

## APIs & extension points

Use the documented object APIs only after confirming availability in the target org and API version. Query schema first; do not infer fields from labels.

```bash
sf org display --target-org "$ORG_ALIAS"
sf data query --target-org "$ORG_ALIAS" --query "SELECT Id FROM FulfillmentPlan LIMIT 1" --json
sf apex run test --target-org "$ORG_ALIAS" --test-level RunLocalTests --wait 30 --result-format json
sf project deploy start --target-org "$ORG_ALIAS" --source-dir force-app --dry-run --test-level RunLocalTests
```

`sf project deploy start --dry-run` validates without saving; use `sf project deploy validate` when a validation job and later quick deploy are required ([Salesforce CLI](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).
Hosted MCP custom servers can combine SObject tools, Flow, and Apex invocable actions and are deployable through Metadata API ([Salesforce Developers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)).

## Configuration & metadata

Configure org aliases outside source control. Scope MCP servers to personas. Require `global @InvocableMethod` for Apex-backed Hosted MCP tools. Pin skill repository versions.

## Agent playbook

1. Detect an `sfdx-project.json`.
2. Load the official platform skill plus one DRO domain skill.
3. Authenticate with approved org alias; never print tokens.
4. Retrieve/describe before editing.
5. Run unit tests and dry-run deploy.
6. Query runtime state read-only.
7. Require approval for writes or submission.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not load hundreds of irrelevant MCP tools.
- Do not rely on generic agent frameworks for Salesforce authorization semantics.
- Do not expose destructive tools to a general-purpose persona.

## Verification & tests

1. Verify CLI and plugin versions.
2. Test least-privilege read and denied write.
3. Run Apex tests and deployment validation.
4. Confirm MCP tool schema matches invocable DTOs.
5. Run an audit test for every mutating call.

## References

- [Salesforce Skills Library](https://github.com/forcedotcom/sf-skills)
- [Build Custom MCP Servers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/custom-servers.html)
- [Hosted MCP Invocable Actions](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/invocable-actions.html)
- [Salesforce CLI project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)
- [Salesforce CLI Apex Commands](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_apex.html)

**Retrieval keywords:** agent tooling, Salesforce CLI, sf-skills, Hosted MCP, Claude Code, Cursor, Codex, progressive disclosure

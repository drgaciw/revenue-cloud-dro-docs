---
title: "External Agentic Skills for DRO"
description: "Evaluate third-party Salesforce/RLM agent skills as accelerators while keeping Salesforce documentation and target-org schema authoritative."
agent_use: "Load when considering an external repository, install method, license, version pin, or adoption risk."
salesforce_products: ["Revenue Cloud Advanced", "Dynamic Revenue Orchestrator", "Salesforce Platform"]
related: ["agentic-skills-inventory", "agentic-skills-gap-analysis", "rlm-skills-porting", "agentic-tooling"]
last_reviewed: 2026-09-09
sources: ["https://github.com/forcedotcom/sf-skills", "https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm", "https://github.com/lzdravkov/rlm-skills", "https://github.com/bgaldino/rlm-base-dev", "https://github.com/arohitu/salesforce-revenue-cloud-skills", "https://github.com/PranavNagrecha/AwesomeSalesforceSkills", "https://github.com/alexkwitko/Salesforce-Agentforce"]
---

## Purpose

Catalog external starting points without asserting unverified DRO depth.

## When to use this doc (agent trigger conditions)

- “Which public skill repo should we use?”
- “Can we install or port this skill?”
- “Does this repo cover DRO?”

## Key concepts

- **Official foundation:** `forcedotcom/sf-skills`, verified as Salesforce’s curated multi-tool skills library.
- **Third-party candidate:** inspect content, license, commit, and tests before adoption.
- **Schema authority:** official docs and target-org describe, not repository prose.

## Data model & objects

External skills should resolve object names against the [DRO standard object reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm) and the canonical taxonomy in [Skills Inventory](./agentic-skills-inventory.md).

## Flow / sequence

1. Pin repository URL and commit.
2. Read license and skill files.
3. Map claimed capabilities to the canonical taxonomy.
4. Compare every API/object claim with official docs.
5. Run isolated evals.
6. Adopt, adapt, or reject.

## APIs & extension points

| Repository | Intended role | Status |
|---|---|---|
| [`forcedotcom/sf-skills`](https://github.com/forcedotcom/sf-skills) | Official Salesforce platform foundation | Verified repository |
| [`lzdravkov/rlm-skills`](https://github.com/lzdravkov/rlm-skills) | Candidate DRO/RLM domain skill | Unverified content in this review |
| [`bgaldino/rlm-base-dev`](https://github.com/bgaldino/rlm-base-dev) | Candidate schema/data migration reference | Unverified content in this review |
| [`arohitu/salesforce-revenue-cloud-skills`](https://github.com/arohitu/salesforce-revenue-cloud-skills) | Candidate Revenue Cloud context | Unverified content in this review |
| [`AwesomeSalesforceSkills`](https://github.com/PranavNagrecha/AwesomeSalesforceSkills) | Candidate index/library | Unverified content in this review |
| [`Salesforce-Agentforce`](https://github.com/alexkwitko/Salesforce-Agentforce) | Candidate pattern reference | Unverified content in this review |

> **Unverified:** Earlier source files claimed exact skill names, installation commands, object counts, stars, and feature coverage for these third-party repositories. Confirm against a pinned commit before reuse.

## Configuration & metadata

Install third-party skills only in a sandboxed project scope. Record repository URL, commit SHA, license, local changes, supported harness, and eval results.

## Agent playbook

1. Use `git ls-remote` or GitHub API to resolve a commit.
2. Review `LICENSE`, `SKILL.md`, scripts, hooks, and network calls.
3. Search for hardcoded org names and obsolete API versions.
4. Compare to [Skills Inventory](./agentic-skills-inventory.md).
5. Run read-only evals before enabling tools.
6. Fork and pin; do not install mutable main-branch code in CI.

## Guardrails & anti-patterns

- Do not invent field API names, status values, permission-set names, or endpoints.
- Do not write directly to production from an agent session. Generate a diff, validate in a sandbox, and require human approval.
- Do not bypass sharing, CRUD, or field-level security in Apex wrappers.
- Do not treat a UI label as an API name. Confirm with object describe, retrieved metadata, or the target-org schema.
- Do not mark downstream fulfillment successful merely because an asynchronous message was accepted.
- Do not repeat popularity metrics without dated evidence.
- Do not execute third-party install scripts before review.
- Do not treat an “RLM” name as proof of Dynamic Revenue Orchestrator coverage.

## Verification & tests

1. License check.
2. Dependency and script review.
3. Trigger-routing tests.
4. Schema-hallucination tests.
5. Permission-denied and no-org tests.
6. Re-audit quarterly or on version change.

## References

- [Salesforce Skills Library](https://github.com/forcedotcom/sf-skills)
- [Dynamic Revenue Orchestrator Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/dynamic_revenue_orchestrator_std_objects_parent.htm)
- Third-party candidate URLs are linked in the table above; they are not Salesforce documentation.

**Retrieval keywords:** external skills, sf-skills, rlm-skills, third-party, pin commit, license review, supply chain, adoption

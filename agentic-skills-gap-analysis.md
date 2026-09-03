# Agentic Skills Gap Analysis and Custom Skill Efficacy for DRO

## Purpose
Evaluate public GitHub repos and skills for agentic engineering with Dynamic Revenue Orchestrator (DRO) and Revenue Cloud Advanced (RCA). Decide whether to adopt, extend, or build custom skills.

## Gap Analysis of Public Repos

| Repo | Stars (approx) | DRO / Decomposition Coverage | Strengths | Gaps for Our Use Case | Recommendation
|------|----------------|------------------------------|-----------|-----------------------|---------------
| bgaldino/rlm-base-dev | 31 | High — explicit DRO data plans (qb-dro), decomposition rule updates, fulfillment step definitions, idempotency tests | Real metadata, CCI tasks, SFDMU plans, E2E Robot tests, feature flags | No Claude Code SKILL.md format; it's a CumulusCI launchpad, not an agent skill | Fork the data plans and tasks as reference; wrap key tasks as custom skills
| alexkwitko/Salesforce-Agentforce | 7 | Low — RLM skill covers catalog/pricing/asset lifecycle, no DRO or decomposition | Reusable DX-first skills, multi-cloud agents, engine-gated playbook | No fulfillment orchestration or decomposition rules | Use as pattern reference for RLM skills, not DRO
| arohitu/salesforce-revenue-cloud-skills | low | None — pricing, PCM, configurator APIs, decision tables | Good structure (evals, references, gotchas) | No DRO, no fulfillment, no decomposition | Adopt structure and eval pattern; build DRO skills from scratch
| PranavNagrecha/AwesomeSalesforceSkills | 27 | Mention only — Integration domain lists "Revenue Lifecycle Management DRO" as one of 35 skills | 1000+ skills, shared templates, golden evals, MCP server, anti-patterns | DRO skill is shallow (one line in a list); no decomposition or fulfillment depth | Monitor; use as broad Salesforce knowledge layer, not DRO specialist
| lzdravkov/rlm-skills (earlier match) | low | Claimed dedicated rlm-dynamic-revenue-orchestrator skill | Closest named match | Verify actual content; may be thin | Validate before heavy reliance
| forcedotcom/sf-skills | 655+ | None specific to DRO | Official, governed, hooks, taxonomy | Generic platform skills; no RCA/DRO domain depth | Foundation layer for Apex/metadata; pair with custom DRO skills

## Key Finding
No public repo provides production-grade, decomposition-aware, fulfillment-orchestration skills for DRO. The closest (rlm-base-dev) is a metadata launchpad, not an agent skill. Broad libraries mention DRO but lack depth on commercial-to-technical mapping, step graphs, or entitlement-driven fulfillment.

## Efficacy of Generating Our Own Skills

**High efficacy — recommended.** Reasons:

1. **Domain specificity is the moat.** Our hundred-image catalog, OCI label conventions, entitlement JSON schema, and Sonatype Nexus path rules are proprietary. No public skill encodes them. Custom skills that reference our index.md, wrapper-patterns.md, and licensing.md will outperform generic ones.

2. **Agentic validation is the highest-value use.** Agents excel at reading decomposition rules, comparing output to entitlement JSON, and flagging drift — exactly the Decomposition Viewer pattern automated. This is deterministic enough to be reliable and high enough leverage to justify the build.

3. **Low marginal cost once structure exists.** Using the Agent Skills spec (SKILL.md + references/ + evals/), we can author a skill in a day. The hard part is the domain knowledge, which we already captured in this repo.

4. **Governance fits our wrapper pattern.** Custom skills should call our Apex/Flow wrappers, never raw Tooling API or browser automation. This keeps agents inside the governed surface.

## Recommended Custom Skill Set

| Skill | Purpose | Grounded In | Priority
|-------|---------|-------------|----------
| dro-decomposition-validator | Read commercial product + rules, simulate decomposition, compare to entitlement JSON and OCI labels | dro-mapping.md, wrapper-patterns.md | P0
| dro-fulfillment-graph-reviewer | Inspect step graph, dependencies, compensation paths; flag deep or untestable graphs | fulfillment-orchestration.md | P0
| dro-entitlement-auditor | Trace entitlement JSON from contract through token mint to download receipt; flag negative-event gaps | licensing.md, token-auditability.md, audit-system.md | P1
| dro-oci-label-sync | Validate OCI manifest labels match technical product catalog; detect drift | agentic-dro.md | P1
| dro-wrapper-author | Generate or review Apex/Flow wrappers for Tier 4 gaps | wrapper-patterns.md, interface-coverage.md | P2

## Build vs Adopt Decision

- **Adopt:** sf-skills (foundation), arohitu structure (evals + references pattern), rlm-base-dev data plans (reference only).
- **Extend:** AwesomeSalesforceSkills DRO mention — contribute depth if it stays shallow.
- **Build:** All five skills above. Start with dro-decomposition-validator; it delivers the most immediate value against our catalog.

## Risks

- Skill drift: public repos update faster than we can track. Pin versions and re-audit quarterly.
- Over-reliance on thin skills: a skill that mentions DRO without encoding our mapping rules will hallucinate plausible-but-wrong decomposition logic. Always ground in our parked markdowns.
- Eval coverage: every custom skill needs golden evals (input order → expected fulfillment lines) before production use.

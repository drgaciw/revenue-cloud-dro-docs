# Porting RLM Skills to Claude Format

Assessment of porting bgaldino/rlm-base-dev (and similar RLM skills) into Claude Code `SKILL.md` format, including effort, risk, and recommendation. Claude Code is the preferred harness; Cursor is not in scope.

## Short answer

**Yes, worth doing — but as a selective port, not a bulk conversion.** The skills are already plain markdown with no editor-specific dependencies, so the mechanical port is cheap. The value is in the schema grounding and skill-authoring patterns, not in the CCI/SFDMU-specific procedures.

## Why it is feasible

- rlm-base-dev skills explicitly state they are consumable by Cursor, Claude Code, Copilot, Codex, Windsurf, Aider — "plain markdown files... no Cursor-specific dependencies."
- They live under `.cursor/skills/` for historical reasons only.
- skill-authoring even documents a `.claude/skill-manifest.yml` registration path.
- Tooling exists: `lu-zhengda/skill-port` converts between Cursor, Claude Code, and Codex formats while preserving unknown frontmatter and non-skill files.
- Cursor-to-Claude conversion is mostly: add YAML frontmatter (`name`, `description` with trigger phrases), ensure folder name matches `name`, drop or ignore `.mdc` Cursor rules (they are supplemental; canonical guidance stays in the skill).

## What ports cleanly

| Source skill | Port value | Effort |
|---|---|---|
| `revenue-cloud-data-model` (+ `domains/dro.md`) | High — object map, relationships, query patterns | Low (add frontmatter, trigger phrases) |
| `skill-authoring` | High — lifecycle, registration, progressive disclosure | Low |
| `revenue-cloud-docs` | Medium — Help grounding | Low |
| `rlm-business-apis` | Medium — REST API usage | Low-Medium |
| `expression-sets` | Medium — step-graph CRUD patterns (adapt, don't copy) | Medium (strip CCI assumptions) |

## What does not port well or is low value

- CCI orchestration, SFDMU data plans, Robot Framework, PDE org build, UX assembly — these assume a CumulusCI/QuantumBit repo layout and scripts that won't exist in our environment.
- Pricing-wiring, constraint-models, decision-tables — adjacent to RCA but outside our DRO/container scope for now.
- Anything hardcoding instance names (QB-*, RLM_*) — port the mechanism, not the names.

## Recommended porting approach

1. **Select, don't bulk-convert.** Start with data-model, skill-authoring, and revenue-cloud-docs. Use `skill-port` for the mechanical frontmatter pass, then hand-edit descriptions for our trigger vocabulary.
2. **Rewrite descriptions for Claude.** Claude relies on description matching. Add phrases like "use when validating DRO decomposition" or "use when authoring a fulfillment step."
3. **Strip CCI/SFDMU assumptions.** Replace "run this CCI task" with "query the org via MCP/Tooling API" or "read the local metadata." Keep the object knowledge; drop the build-system coupling.
4. **Add our proprietary layers on top.** Ported skills become the schema foundation. Custom skills (decomposition validator, entitlement auditor, OCI sync, wrapper author) sit above them and own the hundred-image, OCI-label, Sonatype logic.
5. **Register in both indexes.** Update `.claude/skills` discovery and our `index.md` / `agentic-skills-inventory.md` so Claude routes correctly.
6. **Test non-Cursor consumption.** Open each ported skill as plain markdown and confirm a Claude session can follow it without Cursor context.

## Effort estimate

- Mechanical port of 3-5 core skills: a few hours with `skill-port` + description edits.
- Adaptation (strip CCI, add triggers, register): 1-2 days.
- Full library port: not recommended — low ROI, high maintenance, and most skills are out of scope.

## Risks

- **Stale schema.** rlm-base-dev is pinned to Release 262 / API v67. Re-verify object fields against our org before relying on them.
- **Silent format drift.** Some skills use Cursor-only fields (`globs`, `alwaysApply` in rules). Ignore them; do not let them leak into SKILL.md frontmatter.
- **Over-trusting public procedures.** Public skills encode generic Salesforce patterns. Our entitlement JSON, OCI labels, and Nexus isolation are not in them — do not assume coverage.
- **License/attribution.** Confirm the repo license allows internal use and modification before committing ported copies.

## Opinion

Porting is an effective, low-cost way to bootstrap Claude with Revenue Cloud schema literacy and a proven skill-authoring discipline. It is not a substitute for custom DRO skills. Do the selective port, then invest the saved time in the custom validator, auditor, and OCI-sync skills that actually differentiate our design.

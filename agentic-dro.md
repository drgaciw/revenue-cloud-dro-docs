# Agentic Interactions with DRO

## Scope
Third-party agentic harnesses (e.g., Claude Code, Build) interacting with DRO processes — not Salesforce Agentforce. Focus: validation and review, never silent execution.

## Where agents add value
- **Decomposition validation:** read commercial product, rules, execution conditions, mapped attributes; flag mismatches against the entitlement JSON or OCI labels before commit.
- **Entitlement catalog checks:** verify the separate catalog stays aligned with OCI manifests and commercial definitions.
- **OCI label discovery:** agents reading manifests is a different skill than reading Salesforce metadata — encode catalog conventions in a skill.
- **Audit-chain review:** agents checking the same correlation IDs and negative events defined in token-auditability.md.
- **Compensation review:** agents proposing reverse actions for committed steps on transient failure.

## Hard line
The agent proposes or validates; DRO keeps decomposition deterministic. No silent rule rewrites, no live order firing. Same pattern as the Decomposition Viewer, automated and repeatable.

## Harness and skills
Parked for deeper treatment: which harness (Claude Code, Build, etc.) fits best, skill definitions for catalog conventions, and guardrails for read-only vs propose-only modes.

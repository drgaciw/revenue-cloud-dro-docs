---
title: Using Salesforce DRO Product Decomposition from Claude Code
description: Steps, best practices, and community experience for driving Salesforce Dynamic Revenue Orchestrator (DRO) Product Decomposition from Claude Code as the coding-agent CLI.
last_reviewed: 2026-09-10
sources:
  - https://help.salesforce.com/s/articleView?id=ind.dro_dynamic_revenue_orchestrator.htm&language=en_US&type=5
  - https://help.salesforce.com/s/articleView?id=ind.dro_design_time_decomposition.htm&language=en_US&type=5
  - https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5
  - https://help.salesforce.com/s/articleView?id=ind.dro_run_time_administration.htm&language=en_US&type=5
  - https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5
  - https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5
  - https://help.salesforce.com/s/articleView?id=sf.dro_permissions_in_dynamic_revenue_orchestrator.htm&language=en_US&type=5
  - https://developer.salesforce.com/docs/atlas.en-us.sfFieldRef.meta/sfFieldRef/salesforce_field_reference_UserPermissionAccess.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productdecompenrichmentrule.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlinesourcerel.htm
  - https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlineattribute.htm
  - https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data.html
  - https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_query.html
  - https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html
  - https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_retrieve_start.html
  - https://code.claude.com/docs/en/setup
  - https://code.claude.com/docs/en/memory
  - https://code.claude.com/docs/en/best-practices
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/headless
  - https://code.claude.com/docs/en/mcp
  - https://code.claude.com/docs/en/hooks
  - https://developer.salesforce.com/blogs/2026/05/connect-claude-with-salesforce-hosted-mcp-servers
  - https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin
  - https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/claude.html
  - https://www.salesforce.com/claudeforce/
  - https://github.com/SalesforceAIResearch/agentforce-adlc
  - https://www.salesforceben.com/10-lessons-for-admins-using-claude-code-to-build-salesforce-flows/
  - https://www.salesforceben.com/implementing-salesforce-data-cloud-with-claude-code-and-mcp/
  - https://salesforcedevops.net/index.php/2026/04/15/tdx-2026-reporters-notebook-salesforce-goes-headless-and-widens-the-builder-gap/
  - https://salesforcemonday.com/2026/05/11/claude-code-for-salesforce-development/
  - https://www.reddit.com/r/salesforce/comments/1s5cnit/salesforce_development_work_claude/
  - https://www.reddit.com/r/salesforce/comments/1ro8h3h/claude_code_integration_with_your_salesforce_org/
  - https://www.reddit.com/r/SalesforceDeveloper/comments/1lfgi2h/what_is_better_for_salesforce_development_cursor/
  - https://www.reddit.com/r/codex/comments/1ol3iaj/codex_just_ran_sfdx_delete_from_project_and_org/
  - https://www.reddit.com/r/salesforce/comments/1nowfc9/codex_salesforce_is_pretty_game_changing/
---

## Executive summary

Salesforce Dynamic Revenue Orchestrator (DRO) treats Product Decomposition as a design-time authoring problem plus a run-time inspection problem, and every artifact on both sides is a standard SObject with the ordinary Data-API supported-calls surface ([Salesforce Developers — DRO Objects deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm)). That means Claude Code — Anthropic's terminal coding agent, installed with the native installer or Homebrew ([Claude Code — Advanced setup](https://code.claude.com/docs/en/setup)) — can drive an end-to-end decomposition workflow the same way it drives any SFDX project: `CLAUDE.md` for persistent context ([Claude Code — Best practices](https://code.claude.com/docs/en/best-practices)), plan mode before mutating anything ([Claude Code — Permission modes](https://code.claude.com/docs/en/permission-modes)), and either the Salesforce CLI or the new Salesforce-hosted MCP servers for the actual writes and reads ([Salesforce Developers — Connect Claude with Salesforce Hosted MCP Servers](https://developer.salesforce.com/blogs/2026/05/connect-claude-with-salesforce-hosted-mcp-servers)).

Salesforce released the first official **Salesforce Development plugin for Claude Code** in August 2026, bundling ~40 skills, MCP servers, specialized agents, hooks, and commands, and auto-activating a `salesforce-dev` agent when Claude sees a Salesforce DX project ([Salesforce Developers — Headless Development with Skills and a Claude Code Plugin](https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin)). Practitioners on r/salesforce, Salesforce Ben, and SalesforceDevops.net report that Claude Code shortens flow-build and Apex loops materially when it is grounded in project context and scoped to non-production orgs; the same threads warn about coding agents deleting sandbox content when run unattended in permissive modes ([r/salesforce — "Salesforce Development Work + Claude"](https://www.reddit.com/r/salesforce/comments/1s5cnit/salesforce_development_work_claude/); [r/codex — SFDX delete incident](https://www.reddit.com/r/codex/comments/1ol3iaj/codex_just_ran_sfdx_delete_from_project_and_org/)).

The workflow that follows walks through: bootstrap → author the decomposition rule family → deploy in Salesforce's documented sequence → assign the correct DRO permission sets → submit a test order → verify with the Decomposition Viewer or equivalent SOQL over `FulfillmentOrderLineItem`, `FulfillmentLineSourceRel`, and `FulfillmentLineAttribute` — then debug and iterate. Best practices at the end are grounded in Salesforce's own deployment reference (rule-set references UPDATE-only, condition data update-only, decision tables refreshed after migration) and Claude Code's permission and hook primitives.

## DRO Product Decomposition primer

### What Product Decomposition is

Dynamic Revenue Orchestrator "decomposes" commercial products into technical products that a fulfillment team can deliver. The design-time surface — the **Decomposition Workspace** — lets designers set up the rules that control that breakdown, including conditions and priorities; by default every product decomposes into fulfillment line items unless an execution rule evaluates to false for a given sales-transaction item ([Salesforce Help — Define How a Product Decomposes](https://help.salesforce.com/s/articleView?id=ind.dro_define_how_a_product_decomposes.htm&language=en_US&type=5)). The design guide frames it end-to-end: "Define how commercial products decompose into technical products … Technical products can inherit field and attribute data from related commercial products" ([Salesforce Help — Design Your Order Decomposition](https://help.salesforce.com/s/articleView?id=ind.dro_design_time_decomposition.htm&language=en_US&type=5)).

At run time DRO takes a submitted order, decomposes its line items into `FulfillmentOrderLineItem` records, instantiates the orchestration plan, and lets operators track fulfillment states through the Fulfillment Step History related list ([Salesforce Help — Submit Orders for Decomposition and Order Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_run_time_administration.htm&language=en_US&type=5)). The **Decomposition Viewer** — reached from the Fulfillment Lines tab — visualises the resulting hierarchy, and its Subaction column "identifies the operation applied to an order or decomposed product during decomposition" (Rollback, Field Amendment, Start Date Adjustment, …) ([Salesforce Help — Monitor Decomposition During Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)).

### The SObject surface a coding agent must know

Product Decomposition and everything downstream of it is standard-SObject metadata that answers the ordinary Data API. Both `ProductFulfillmentDecompRule` and `ProductDecompEnrichmentRule` explicitly declare `create() / update() / upsert() / query() / retrieve() / delete() / undelete()` as supported calls ([Salesforce Developers — ProductFulfillmentDecompRule](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm); [ProductDecompEnrichmentRule](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productdecompenrichmentrule.htm)). The runtime tables the Viewer reads — `FulfillmentLineSourceRel` and `FulfillmentLineAttribute` — do the same ([FulfillmentLineSourceRel](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlinesourcerel.htm); [FulfillmentLineAttribute](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlineattribute.htm)).

Deployment sequence for the decomposition family (from Salesforce's own reference):

| Sequence | Object | Type |
|---|---|---|
| 1 | `FulfillmentStepDefinitionGroup` | Configuration |
| 2 | `FulfillmentStepDefinition` | Configuration |
| 3 | `FulfillmentStepDependencyDef` | Configuration |
| 4 | `ProductFulfillmentScenario` | Configuration |
| … | `ProductFulfillmentDecompRule` → `ValTfrmGrp` → `ValTfrm` → `ProductDecompEnrichmentRule` → `ProdtDecompEnrchVarMap` | Decomposition family |

The full ordered table is in the [DRO Objects deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm).

Two verbatim special-field rules constrain how Claude Code must write:

- On `ProductFulfillmentDecompRule` (and other rule-owning objects), "Rule set references are created in the target org by using `UPDATE` operation on the JSON fields as listed in the Special Fields section. Any rule set records and references aren't created on `INSERT` operation." ([DRO Additional Deployment Information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).
- After migrating `FulfillmentFalloutRule`, "refresh the related Decision Tables" (same page).

The Metadata API surface for DRO is only the Setup layer — feature settings and the Fulfillment User selector — not the record data ([DRO Metadata deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm)). Tooling API is not required for DRO record data; the Salesforce CLI record commands default to the ordinary Data API and use `--use-tooling-api` only when explicitly opted in ([Salesforce CLI — `data update record`](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_update_record.html)).

### Permission sets that gate authoring vs. monitoring

The permission field reference is the authoritative list of DRO permissions:

| API name | Label |
|---|---|
| `PermissionsDfoAdminUser` | System Administrator access for DRO |
| `PermissionsDFODesignerUser` | Allows user to gain Fulfillment Designer level access to DRO |
| `PermissionsDFOManagerOperatorUser` | Allows user to gain Fulfillment Manager or Fulfillment Operator level access to DRO |
| `PermissionsDROOrchestrateTransactionUser` | Submit Transactions and Orchestrate User |
| `PermissionsDROOrderSubmitInitiateUser` | Submit Transactions User |
| `PermissionsOrderSubmitUser` | Submit Transactions and Fulfillment User |

Source: [Salesforce Field Reference — UserPermissionAccess](https://developer.salesforce.com/docs/atlas.en-us.sfFieldRef.meta/sfFieldRef/salesforce_field_reference_UserPermissionAccess.htm).

The Revenue Management permission-sets table lists **Fulfillment Manager/Operator** as an admin-assignable permission set for DRO ([Assign Agentforce Revenue Management Permission Sets](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5)), and the DRO permissions page states that **"during fulfillment, Dynamic Revenue Orchestrator uses the permissions associated with the fulfillment user rather than those of the user who submits an order … The fulfillment user is selected from the Dynamic Revenue Orchestrator Settings pane"** ([Permissions in Dynamic Revenue Orchestrator](https://help.salesforce.com/s/articleView?id=sf.dro_permissions_in_dynamic_revenue_orchestrator.htm&language=en_US&type=5)). A Claude Code test user therefore needs `DFOManagerOperatorUser` (or `DfoAdminUser`) to reach the Decomposition Viewer, and the org's configured Fulfillment User needs the equivalent permissions at run time.

## Claude Code setup for Salesforce

### Install and baseline

Claude Code installs from the native installer (`curl -fsSL https://claude.ai/install.sh | bash` on macOS/Linux/WSL; PowerShell and Windows CMD variants exist) or via `brew install --cask claude-code`; the native install auto-updates in the background ([Claude Code — Advanced setup](https://code.claude.com/docs/en/setup)). It runs against any project directory, including SFDX projects, note vaults, and docs folders ([Claude Code — Common workflows](https://code.claude.com/docs/en/common-workflows)).

### Persistent project context via `CLAUDE.md`

`CLAUDE.md` is "a special file that Claude reads at the start of every conversation … persistent context Claude cannot infer from code alone" and typically lists bash commands, code style, and workflow rules ([Claude Code — Best practices](https://code.claude.com/docs/en/best-practices); [How Claude remembers your project](https://code.claude.com/docs/en/memory)). Run `/init` to generate a first draft. Additional directories can be pulled into memory with `--add-dir` when `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` is set ([Claude Code — Memory](https://code.claude.com/docs/en/memory)).

For DRO work the memory file should encode:

- The DRO object family and their deployment sequence (link to the reference above).
- The rule-set / condition / enrichment-identifier special-field write rules.
- The `DFOManagerOperatorUser` / `DfoAdminUser` guidance for both the agent test user and the Fulfillment User.
- Hard rules like "never target production", "always start in plan mode", and "verify with a SOQL query before generating the rule".

### Permission modes and per-tool allow-lists

Claude Code exposes a documented permission model with `plan` and `--permission-mode` flags ([Claude Code — Permissions](https://code.claude.com/docs/en/permissions); [Choose a permission mode](https://code.claude.com/docs/en/permission-modes)). The classifier that gates auto-approval only sees user messages, non-read-only tool calls, and `CLAUDE.md` content; tool results are stripped, so hostile content in a fetched page cannot rewrite the mode ([Permission modes](https://code.claude.com/docs/en/permission-modes)). Plan mode "is particularly valuable before touching metadata," as an independent Salesforce practitioner puts it ([SalesforceMonday — Claude Code for Salesforce Development](https://salesforcemonday.com/2026/05/11/claude-code-for-salesforce-development/)).

### Hooks for hard guardrails

Claude Code hooks are shell/PowerShell scripts wired via `.claude/settings.json` that run at deterministic points in the tool loop ([Claude Code — Hooks reference](https://code.claude.com/docs/en/hooks)). They are the right place to enforce, for example, "block any `sf` command whose `--target-org` alias contains `prod`" or "always run `sf project deploy validate` before `sf project deploy start`".

### MCP: the Salesforce-hosted path

Salesforce shipped **Hosted MCP Servers** in 2026 and documents the Claude connection in detail. Claude authenticates via OAuth 2.0 through an **External Client App (ECA)** created in the target org, and Claude Code's local callback URL is `http://localhost:38000/callback` when the user is running with an enterprise-provisioned Anthropic API key ([Salesforce Developers — Connect Claude with Salesforce Hosted MCP Servers](https://developer.salesforce.com/blogs/2026/05/connect-claude-with-salesforce-hosted-mcp-servers)). Server URLs are `https://api.salesforce.com/platform/mcp/v1/<SERVER-NAME>` for production and `.../v1/sandbox/<SERVER-NAME>` for sandbox/scratch ([Configure Claude — Hosted MCP Servers](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/claude.html)).

MCP servers can be added to Claude Code with `claude mcp add --transport http <name> <url>` ([Claude Code — MCP](https://code.claude.com/docs/en/mcp)). Salesforce's Headless 360 page frames this pattern as "no new permissions model … Claude sees only what a user is authorized to see, and can only do what that user is authorized to do" ([Claudeforce](https://www.salesforce.com/claudeforce/)).

### The official Salesforce Development plugin for Claude Code

Salesforce's August 2026 blog announced its first Claude Code plugin. Verbatim highlights ([Headless Development with Skills and a Claude Code Plugin](https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin)):

| Item | Detail |
|---|---|
| Distribution | Installed from the Claude Plugin Marketplace |
| Bundle | Salesforce development skills, MCP servers, specialized agents, hooks, and commands |
| Auto-activation | Detects a Salesforce DX project and activates the `salesforce-dev` agent |
| Initial scope | ~40 skills from the Salesforce skills library |
| Prerequisites | Claude Code, Node.js LTS 22 or 24, Salesforce CLI, Python 3.8+ (for org-detection / deployment-safety hooks) |

Adjacent to it, Salesforce AI Research maintains an open-source Agentforce ADLC repository — `SalesforceAIResearch/agentforce-adlc` — that packages Claude Code skills for the Agentforce agent lifecycle and ships plan-mode orchestrator, authoring, engineer, and QA sub-agents ([GitHub — SalesforceAIResearch/agentforce-adlc](https://github.com/SalesforceAIResearch/agentforce-adlc)). **Unverified:** neither the plugin nor the ADLC repo advertises DRO-specific skills at the time of writing; the DRO workflow below therefore drives the Salesforce CLI and Data API directly and layers the plugin only where it clearly helps (org detection, safety hooks, generic Apex/SOQL skills).

## End-to-end workflow

Every command below is grounded in official CLI, API, or Anthropic docs. Prompts are illustrative and follow patterns explicitly documented as best practice by Anthropic and the community sources cited.

### 0. Repo bootstrap

```bash
# In an SFDX project root
claude /init                     # generate CLAUDE.md
```

Add to `CLAUDE.md`:

```markdown
# DRO project rules
- Never target an org whose alias contains "prod".
- Start every session in plan mode. Only leave plan mode after I approve the plan.
- All decomposition-rule writes go through the Data API (sf data … or REST).
  Do NOT use --use-tooling-api for DRO record data.
- Deployment sequence for the decomposition family:
  ProductFulfillmentDecompRule → ValTfrmGrp → ValTfrm →
  ProductDecompEnrichmentRule → ProdtDecompEnrchVarMap
- Rule-set references on ProductFulfillmentDecompRule, FulfillmentStepDefinition,
  ProductFulfillmentScenario and FulfillmentTaskAssignmentRule are created ONLY on
  UPDATE, never on INSERT.
- After migrating FulfillmentFalloutRule, refresh related Decision Tables.
- Agent test users need DFOManagerOperatorUser (or DfoAdminUser) to open the
  Decomposition Viewer. The org's Fulfillment User needs the same.
- Verification-first: write the SOQL that will assert the outcome BEFORE
  generating the rule.
```

`CLAUDE.md` is loaded fresh on every conversation and does not need to be re-attached each turn ([Claude Code — Memory](https://code.claude.com/docs/en/memory); [Best practices](https://code.claude.com/docs/en/best-practices)).

Wire in the Salesforce Development plugin (optional but recommended when it exists in your Claude Code environment):

```text
/plugin marketplace add anthropics/claude-plugins-official
/plugin install salesforce-dev@claude-plugins-official   # Unverified plugin id — confirm from the Claude Plugin Marketplace
/reload-plugins
```

The exact plugin id and marketplace path is published on the Salesforce blog announcement ([Headless Development with Skills and a Claude Code Plugin](https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin)); the `/plugin` slash commands themselves are documented on the [Claude Code MCP page](https://code.claude.com/docs/en/mcp).

### 1. Create a safe target org

```bash
sf org create scratch --edition developer --alias dro-dev --set-default
# or
sf org create sandbox --alias dro-sandbox --set-default
```

Source-tracking is enabled by default on scratch and sandbox orgs, and never allowed on production ([Salesforce CLI — project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html)).

### 2. Assign DRO permission sets to the users the workflow needs

```bash
# The user Claude Code will act as (least privilege to reach the Viewer)
sf org assign permset --name DFOManagerOperatorUser --target-org dro-dev

# The Fulfillment User configured in DRO Settings
sf org assign permset --name DFOManagerOperatorUser \
    --target-org dro-dev --on-behalf-of "$FULFILLMENT_USER"
```

Both `DFOManagerOperatorUser` and `DfoAdminUser` are listed in the Revenue Management permission-sets table ([Salesforce Help](https://help.salesforce.com/s/articleView?id=ind.revenue_cloud_permission_sets_table.htm&language=en_US&type=5)), and the DRO permissions page confirms that DRO uses the Fulfillment User at run time ([Permissions in Dynamic Revenue Orchestrator](https://help.salesforce.com/s/articleView?id=sf.dro_permissions_in_dynamic_revenue_orchestrator.htm&language=en_US&type=5)).

### 3. Author the Product Decomposition Rule family

Ask Claude Code in plan mode:

> "Plan a Product Decomposition Rule that decomposes the commercial product `Fiber-Plan-500` into technical products `ONT-Device` and `Fiber-Access-Service`, with an enrichment rule that copies the `Bandwidth` attribute from the commercial product to `Fiber-Access-Service`. Draft it as: (1) SOQL that will assert the resulting `FulfillmentOrderLineItem` and `FulfillmentLineAttribute` rows after a test order runs, (2) the ordered sequence of `sf data create record` / `sf data update record` calls, and (3) the deployment order per the Salesforce reference."

Plan-mode approval before mutation is the pattern the Claude Code docs treat as the default safe behaviour ([Claude Code — Permission modes](https://code.claude.com/docs/en/permission-modes)).

Two-phase write pattern for the special-field objects:

```bash
# Phase 1 — INSERT the rule shell (no rule-set refs on INSERT)
sf data create record --sobject ProductFulfillmentDecompRule \
  --values "Name='Fiber-Plan-500 Decomp' \
            ProductId=$PROD_ID \
            IsActive=false" \
  --target-org dro-dev

# Phase 2 — UPDATE with the rule-set reference JSON fields
sf data update record --sobject ProductFulfillmentDecompRule \
  --record-id $NEW_RULE_ID \
  --values "RuleSetReferencesJson='<...>' IsActive=true" \
  --target-org dro-dev
```

`sf data create record` and `sf data update record` are the documented Data API commands and default to the ordinary Salesforce API (Tooling API only when `--use-tooling-api` is passed) ([CLI reference](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data.html); [data update record](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_update_record.html)). The INSERT-then-UPDATE split is required by the [DRO Additional Deployment Information page](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm). **Unverified:** the exact JSON payload for `RuleSetReferencesJson` and the current field names for enrichment identifiers vary by org and API version — always confirm with a target-org `sf sobject describe` before generating them; the field list on the `ProductFulfillmentDecompRule` reference is intentionally sparse in the public page content ([reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm)).

### 4. Deploy Setup metadata (feature flags, Fulfillment User) when it changes

```bash
sf project deploy start --source-dir force-app/main/default/settings \
    --target-org dro-dev
```

The Metadata layer for DRO covers the Setup surface — `Setup > Feature Settings > Dynamic Revenue Orchestrator`, In-flight Amendments, Future Dated Steps, Link Task to Step Source, Fulfillment User — as documented in the [DRO Metadata deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_metadata.htm). The CLI command and its source-tracking behaviour are on the [project deploy start](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_project_deploy_start.html) reference.

### 5. Submit a test order

Salesforce documents the run-time steps: sales reps submit orders in DRO, orders are submitted for decomposition, the orchestration plan is instantiated, and operators track decomposition and orchestration plans ([Submit Orders for Decomposition and Order Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_run_time_administration.htm&language=en_US&type=5)). In a Claude Code session the cleanest path is to submit the order via the standard `Order` REST endpoint (Claude Code drives that through `sf data create record` on `Order`/`OrderItem` or through Apex anonymous execute — both are ordinary Data API calls). **Unverified:** Salesforce's public runtime-administration page describes UI submission and does not give the CLI-only sequence for order submission; if your org has an established Apex or Flow entry point for submitting orders to DRO, use that.

### 6. Verify with the Decomposition Viewer or equivalent SOQL

The Viewer reads three run-time SObjects a coding agent can query directly:

- `FulfillmentOrderLineItem` — the decomposed line item; action valid values are `Add`, `Amend`, `NoChange`, `Renew`, `Cancel` per the [Fulfillment Order Line Item Actions page](https://help.salesforce.com/s/articleView?id=ind.dro_fulfillment_order_line_item_actions.htm) (previously verified in the RCA docs project).
- `FulfillmentLineSourceRel` — provenance and subaction; `SourceType` is `SourceBundleRoot` or `SourceLineItem`, and `SupplementalAction` (v62+) is `Add|Amend|Cancel|NoChange` per the [reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlinesourcerel.htm).
- `FulfillmentLineAttribute` — mapped attributes carrying `AttributeDefinitionId`, `AttributeName`, `AttributePicklistValueId`, `AttributeValue`, `ExternalId`, `FulfillmentOrderLineItemId` per the [reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_fulfillmentlineattribute.htm).

Sample Claude Code prompt and grounded SOQL:

> "Query the fulfillment lines for order `$ORDER_ID`, join their FulfillmentLineSourceRel and FulfillmentLineAttribute rows, and tell me whether the expected `Bandwidth` attribute landed on `Fiber-Access-Service`. Use `sf data query`."

```bash
sf data query --target-org dro-dev --result-format json --query "
SELECT Id, FulfillmentOrderId, FulfillmentOrderLineItemNumber, Product2Id, Quantity
FROM   FulfillmentOrderLineItem
WHERE  FulfillmentOrderId IN
       (SELECT Id FROM FulfillmentOrder WHERE OrderId = '$ORDER_ID')"

sf data query --target-org dro-dev --result-format json --query "
SELECT Id, FulfilmentOrderLineId, SourceLineItemId, SourceType, SupplementalAction
FROM   FulfillmentLineSourceRel
WHERE  FulfilmentOrderLineId IN ($FOLLI_IDS)"

sf data query --target-org dro-dev --result-format json --query "
SELECT AttributeName, AttributeValue, FulfillmentOrderLineItemId
FROM   FulfillmentLineAttribute
WHERE  FulfillmentOrderLineItemId IN ($FOLLI_IDS)"
```

`sf data query` is the documented SOQL command; results over 10,000 rows should switch to `sf data export bulk` ([CLI — data query](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_query.html)). The Viewer itself lives on the Fulfillment Lines tab and shows Subaction and Reason for Action columns for troubleshooting fallout ([Monitor Decomposition During Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_monitor_decomposition_during_fulfillment.htm&language=en_US&type=5)).

### 7. Debug and iterate

Typical loop that community posts describe positively for coding agents:

1. Ask Claude Code to enumerate failing lines and print the corresponding `Subaction` / `Reason for Action`.
2. Ask it to hypothesize a rule or enrichment change; approve the plan.
3. Apply the change with the two-phase INSERT+UPDATE pattern above; re-submit a test order; re-run the verification SOQL.
4. Stage the diff — the plugin's hooks or your own can enforce that a deploy first passes `sf project deploy validate`.

## Best practices

1. **Least-privilege permission sets.** Give the agent user `DFOManagerOperatorUser`, not `DfoAdminUser`, unless the task actually needs admin-only DRO objects; DRO explicitly evaluates run-time work against the Fulfillment User's permissions ([Permissions in Dynamic Revenue Orchestrator](https://help.salesforce.com/s/articleView?id=sf.dro_permissions_in_dynamic_revenue_orchestrator.htm&language=en_US&type=5)).
2. **Two-phase writes for special-field objects.** Never let Claude Code craft an INSERT that carries rule-set references — those fields are silently ignored on INSERT and only bound on UPDATE ([DRO Additional Deployment Information](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_additional_info.htm)).
3. **Refresh downstream artifacts after data migration.** For `FulfillmentFalloutRule`, decision tables must be refreshed after migration; encode that as a post-deploy checklist in `CLAUDE.md` (same page).
4. **Deploy in the documented sequence.** The [Objects deployment reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/deployment_dynamic_revenue_orchestrator_objects.htm) numbers the deployment order; wrap it in a Claude Code hook that rejects out-of-order deployments.
5. **Never point the agent at production without an explicit override.** Use Claude Code's `--permission-mode` and per-tool allow-lists ([Permissions](https://code.claude.com/docs/en/permissions); [Permission modes](https://code.claude.com/docs/en/permission-modes)). Salesforce Ben's admin post ("add safeguards so dependencies … are deployed first, and instruct Claude Code to verify that deployments are successful afterward") captures the pattern well ([Salesforce Ben — 10 Lessons](https://www.salesforceben.com/10-lessons-for-admins-using-claude-code-to-build-salesforce-flows/)).
6. **Verification-first prompting.** Have Claude write the SOQL assertion before generating the rule. Community posts specifically call out that coding agents behave better when they can execute tests and iterate against the outcome ([r/salesforce — "Codex + Salesforce is pretty game changing…"](https://www.reddit.com/r/salesforce/comments/1nowfc9/codex_salesforce_is_pretty_game_changing/)).
7. **Prefer plan mode for anything that touches metadata.** "Plan mode is particularly valuable before touching metadata" ([SalesforceMonday — Claude Code for Salesforce Development](https://salesforcemonday.com/2026/05/11/claude-code-for-salesforce-development/)).
8. **Ground Claude in concrete examples.** Tom Bassett reports that Claude Code "performs best when it can reference concrete examples of Flows carrying out specific actions" ([Salesforce Ben — 10 Lessons](https://www.salesforceben.com/10-lessons-for-admins-using-claude-code-to-build-salesforce-flows/)); apply the same to decomposition — commit reference rules and Viewer output as fixtures Claude can imitate.
9. **Do not use `--use-tooling-api` for DRO record data.** The CLI reference is clear that Tooling API mode is opt-in only ([data update record](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_update_record.html)); DRO record objects live on the ordinary Data API.
10. **Small diffs, single tests.** The [Claude Code best-practices](https://code.claude.com/docs/en/best-practices) example memory advises "Prefer running single tests, and not the whole test suite, for performance" — the same rule applies to `sf project deploy start --tests`.
11. **Use headless mode for CI.** `claude -p` runs headlessly and, without `--bare`, loads the working directory's `.claude/settings.json` hooks and `.mcp.json` servers ([Claude Code — Headless](https://code.claude.com/docs/en/headless)). This is the right form factor for a CI-side deploy that still needs Claude Code guardrails.

## User experiences and community signal

### Positive signal

- **Tom Bassett (Salesforce Ben, Aug 2025 — currency Unverified).** Uses the Claude Code extension in VS Code with the Salesforce Extension Pack and Salesforce CLI to generate Flows to company standards, retrieves org metadata automatically with a custom rule ("seed my local project with metadata from Salesforce"), and deploys back through CLI. Says Claude performs best when it can reference concrete flow examples and iterates well when told to include descriptions on Assignments, Variables, Get Records, etc. ([Salesforce Ben — 10 Lessons](https://www.salesforceben.com/10-lessons-for-admins-using-claude-code-to-build-salesforce-flows/)).
- **StatisticianVivid915 (r/salesforce, 62 upvotes).** "It feels like I can tackle anything within Salesforce now, which is fantastic! Tasks that used to drag on for ages can now be completed in a fraction of the time. I hardly need to engage in UI development anymore, except for testing to ensure everything functions properly." ([Salesforce Development Work + Claude](https://www.reddit.com/r/salesforce/comments/1s5cnit/salesforce_development_work_claude/)).
- **Wise-Glass-4425 (r/salesforce, Codex + Salesforce thread).** Codex's ability to "run tests" and "refine code until everything passes" flipped the author from "not particularly impressed" to describing the outcomes as "truly astonishing" once connected to a scratch org. Relevant to Claude Code because Anthropic's docs describe the same tool-loop pattern ([r/salesforce](https://www.reddit.com/r/salesforce/comments/1nowfc9/codex_salesforce_is_pretty_game_changing/)).
- **SalesforceDevops.net (TDX 2026 notebook).** "The message is clear: if you're already vibe coding with Claude or Cursor, everything you know still works." Vernon Keenan quotes a Salesforce claim that the DevOps Center MCP can reduce cycle times "up to 40%" ([TDX 2026 Reporter's Notebook](https://salesforcedevops.net/index.php/2026/04/15/tdx-2026-reporters-notebook-salesforce-goes-headless-and-widens-the-builder-gap/)).
- **SalesforceMonday.** "One of the most underappreciated aspects of working with Claude Code is learning to manage context deliberately … Plan mode is particularly valuable before touching metadata." ([SalesforceMonday](https://salesforcemonday.com/2026/05/11/claude-code-for-salesforce-development/)). No first-person attribution; treat as an editorial claim.
- **Salesforce Ben — Data 360 with Claude Code + MCP.** The author programmatically designed an identity-resolution pipeline "from scratch" via Claude Code and Headless Data 360, describing traditional equivalents as "multiple meetings, spreadsheets, and weeks of back-and-forth." Not DRO-specific but a directly-comparable coding-agent workflow against a data-heavy Salesforce surface ([Salesforce Ben — Data Cloud with Claude Code and MCP](https://www.salesforceben.com/implementing-salesforce-data-cloud-with-claude-code-and-mcp/)).

### Negative and cautionary signal

- **Willing_Ad2724 (r/codex — SFDX delete incident).** While iterating on an Apex test in a sandbox, Codex silently executed `SFDX: delete from project and org` and removed the Apex class and its test file. Author's rule: prefer chat mode, avoid agent mode for destructive commands. Directly generalizable to Claude Code's permission modes — this is precisely why plan mode and per-tool allow-lists exist ([r/codex](https://www.reddit.com/r/codex/comments/1ol3iaj/codex_just_ran_sfdx_delete_from_project_and_org/); [Claude Code — Permissions](https://code.claude.com/docs/en/permissions)).
- **tommeh5491 (r/SalesforceDeveloper, Cursor vs. Claude Code, 24 upvotes).** "We stick to traditional methods and rely on a developer." Sample of one, but a real signal that a slice of production Salesforce teams are still not using coding agents ([Reddit](https://www.reddit.com/r/SalesforceDeveloper/comments/1lfgi2h/what_is_better_for_salesforce_development_cursor/)).
- **Wise-Glass-4425 (same Codex thread, negative side).** "Extensive configurations requested in a single prompt can produce subpar outputs" — the author attributed this to the agent struggling to generate "a large number" of artifacts at once. Encodes the "small diffs" best practice above.

### DRO-specific community signal

**Unverified — thin.** No first-person Claude-Code-plus-DRO write-up surfaced in the sources gathered here. The closest analogues are the general Salesforce+Claude Code posts above and the Salesforce AI Research Agentforce ADLC repo, which targets Agentforce agents rather than DRO ([GitHub — SalesforceAIResearch/agentforce-adlc](https://github.com/SalesforceAIResearch/agentforce-adlc)). The workflow in this report therefore extends documented Salesforce + Claude Code patterns to DRO using Salesforce's own DRO reference material rather than a community walkthrough.

## Open questions and gaps

1. **Concrete `RuleSetReferencesJson` payload shape** for `ProductFulfillmentDecompRule` and siblings. The reference page lists supported calls but not the fields ([reference](https://developer.salesforce.com/docs/atlas.en-us.revenue_lifecycle_management_dev_guide.meta/revenue_lifecycle_management_dev_guide/sforce_api_objects_productfulfillmentdecomprule.htm)); resolve with a target-org `sf sobject describe`.
2. **Official CLI/Apex path for submitting an order to DRO.** The Salesforce runtime page describes UI submission; the CLI/API sequence should be confirmed against your org's specific submit path ([Submit Orders for Decomposition and Order Fulfillment](https://help.salesforce.com/s/articleView?id=ind.dro_run_time_administration.htm&language=en_US&type=5)).
3. **DRO-aware MCP tools.** Whether the Salesforce Hosted MCP servers or the Salesforce Development Claude Code plugin expose DRO-specific tools (Decomposition Viewer, Fulfillment Lines) directly is not stated in the sources gathered here — verify against the current plugin release notes ([Headless Development with Skills and a Claude Code Plugin](https://developer.salesforce.com/blogs/2026/08/headless-development-with-skills-and-a-claude-code-plugin)).
4. **First-person Claude-Code + DRO write-ups.** None surfaced. When one is published (Salesforce Ben, SalesforceDevops.net, and Reddit are the likely first venues based on adjacent coverage), fold its exact prompts and pitfalls back into `CLAUDE.md`.
5. **Plugin id for `salesforce-dev`.** The exact plugin id and marketplace path in the `/plugin install` command should be confirmed from the Claude Plugin Marketplace before scripting it into onboarding.

**Retrieval keywords:** DRO, Dynamic Revenue Orchestrator, Product Decomposition, Decomposition Viewer, FulfillmentOrderLineItem, FulfillmentLineSourceRel, FulfillmentLineAttribute, ProductFulfillmentDecompRule, ProductDecompEnrichmentRule, Claude Code, CLAUDE.md, plan mode, Salesforce Hosted MCP, Salesforce Development plugin, DFOManagerOperatorUser, DfoAdminUser, Fulfillment User, sf data query, sf project deploy start, headless.

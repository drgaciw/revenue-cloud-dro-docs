# Wrapper Patterns for DRO / RCA Agentic Engineering

**Keywords:** wrappers, invocable methods, Tooling API, MCP tools, browser automation, agentic engineering, decomposition rules, fulfillment steps, validation, gap closure

## Why wrappers matter

Most DRO design-time config is metadata and reachable via CLI / Tooling API. Runtime submission is reachable via invocable actions. But several high-value operations still live only in the browser UI: the visual step-graph editor, the Decomposition Viewer, and a few Salesforce Go toggles. For an agentic workflow those gaps are unacceptable — an agent cannot reliably drive a browser, and browser automation is brittle, slow, and unsafe in production paths.

The fix is a thin **wrapper layer**: Apex `@InvocableMethod` classes (or Flow actions) that perform the equivalent create / update / validate operation through the Tooling API or standard objects, then expose those actions as Hosted MCP tools. The agent never touches the UI; it calls a deterministic, auditable, permission-gated function.

## Recommended patterns

### Pattern 1 — Tooling API facade (design-time writes)
Use the Tooling API from Apex to create or update metadata records that the UI would otherwise create. DRO exposes these as Tooling objects: `ProductFulfillmentDecompositionRule`, `ProductFulfillmentScenario`, `FulfillmentStepDefinition`, `FulfillmentStepDefinitionGroup`, `FulfillmentStepDependencyDefinition`, `FulfillmentTaskAssignmentRule`, `OrchestrationPlanCtxMapping`, and others.

- Read via SOQL on the Tooling object.
- Write via `ToolingApi` REST or the community `apex-toolingapi` / `apex-mdapi` libraries.
- Always run as a Queueable (callouts are not allowed in triggers) and capture the async result.
- Gate with the same permission sets humans use: Fulfillment Designer for design-time, never DRO Admin in a headless context.

### Pattern 2 — Invocable validation action (read-only checks)
For the Decomposition Viewer gap, do not try to render the UI. Instead expose an Apex method that runs the same decomposition logic the platform runs and returns the resulting fulfillment line items + mapped attributes as JSON. The agent compares that output against the entitlement JSON and the OCI labels.

- Input: order Id or a synthetic sales-transaction payload.
- Output: structured decomposition result (commercial line → technical products → attributes → action reasons).
- The agent flags mismatches; a human confirms before deploy.

### Pattern 3 — Step-graph builder from JSON spec
The visual editor's real value is expressing dependencies. Replace it with a declarative spec: a JSON array of steps, each with `name`, `type`, `scope`, `dependsOn[]`, `executeOnCondition`, and `customConfigParameter`. The wrapper translates that into `FulfillmentStepDefinition` + `FulfillmentStepDependencyDefinition` records via the Tooling API.

- Idempotent: hash the spec; skip if unchanged.
- Validates scope and dependency cycles before writing.
- Returns the created step definition Ids for audit.

### Pattern 4 — MCP tool surface
Expose every wrapper as a Hosted MCP tool (Salesforce-hosted MCP servers are GA). Name tools by intent, not by object: `dro_create_decomposition_rule`, `dro_validate_decomposition`, `dro_upsert_step_graph`, `dro_list_fulfillment_scenarios`. Keep inputs small and typed; return structured JSON the agent can reason over.

### Pattern 5 — Governance and safety
- Propose, don't mutate: wrappers write to a sandbox or a change set; production deploys go through PR + CLI.
- Log every wrapper call with correlation Id, actor, and payload hash into the audit system.
- Rate-limit and require explicit confirmation for destructive operations (delete rule, deactivate scenario).
- Never let an agent hold a session that can bypass FLS or sharing.

## Code examples

### Example A — Invocable method wrapping the Tooling API to create a decomposition rule

```apex
public with sharing class DroDecompositionRuleAction {
    public class Request {
        @InvocableVariable(required=true) public String commercialProductId;
        @InvocableVariable(required=true) public String technicalProductId;
        @InvocableVariable public Integer priority;          // lower = higher priority
        @InvocableVariable public String executionCondition; // e.g. "Tier = 'Enterprise'"
    }
    public class Result {
        @InvocableVariable public Id ruleId;
        @InvocableVariable public String status;
        @InvocableVariable public String message;
    }

    @InvocableMethod(label='Create DRO Decomposition Rule'
        description='Creates a ProductFulfillmentDecompositionRule via Tooling API')
    public static List<Result> createRule(List<Request> reqs) {
        List<Result> out = new List<Result>();
        for (Request r : reqs) {
            Result res = new Result();
            try {
                // Enqueue because Tooling API callouts are not allowed in trigger context
                Id jobId = System.enqueueJob(new CreateRuleJob(r));
                res.status = 'QUEUED';
                res.message = 'Job ' + jobId;
            } catch (Exception e) {
                res.status = 'ERROR';
                res.message = e.getMessage();
            }
            out.add(res);
        }
        return out;
    }

    public class CreateRuleJob implements Queueable, Database.AllowsCallouts {
        private final Request req;
        public CreateRuleJob(Request req) { this.req = req; }
        public void execute(QueueableContext ctx) {
            // Use community apex-toolingapi or raw REST to POST /tooling/sobjects/ProductFulfillmentDecompositionRule
            HttpRequest hr = new HttpRequest();
            hr.setEndpoint(URL.getOrgDomainUrl().toExternalForm()
                + '/services/data/v62.0/tooling/sobjects/ProductFulfillmentDecompositionRule');
            hr.setMethod('POST');
            hr.setHeader('Authorization', 'Bearer ' + UserInfo.getSessionId());
            hr.setHeader('Content-Type', 'application/json');
            Map<String,Object> body = new Map<String,Object>{
                'SourceProductId' => req.commercialProductId,
                'TargetProductId' => req.technicalProductId,
                'Priority' => req.priority == null ? 10 : req.priority,
                'Condition' => req.executionCondition
            };
            hr.setBody(JSON.serialize(body));
            HttpResponse resp = new Http().send(hr);
            // Persist result + audit event (correlation Id, payload hash) here
        }
    }
}
```

### Example B — Invocable validation action (Decomposition Viewer equivalent)

```apex
public with sharing class DroDecompositionValidator {
    public class Request {
        @InvocableVariable(required=true) public Id orderId;
    }
    public class Result {
        @InvocableVariable public String decompositionJson; // commercial → technical → attributes
        @InvocableVariable public Boolean matchesEntitlement;
        @InvocableVariable public List<String> mismatches;
    }

    @InvocableMethod(label='Validate DRO Decomposition'
        description='Runs decomposition and compares output to entitlement JSON')
    public static List<Result> validate(List<Request> reqs) {
        List<Result> out = new List<Result>();
        for (Request r : reqs) {
            Result res = new Result();
            // 1. Invoke the platform decompose action (or query the resulting
            //    FulfillmentOrderLineItem + attribute records for the order).
            // 2. Load the customer's entitlement JSON (from licensing system).
            // 3. Diff: every technical product + attribute must be entitled.
            res.decompositionJson = DroDecompositionService.serialize(r.orderId);
            EntitlementCheck chk = EntitlementService.compare(res.decompositionJson, r.orderId);
            res.matchesEntitlement = chk.ok;
            res.mismatches = chk.mismatches;
            out.add(res);
        }
        return out;
    }
}
```

### Example C — MCP tool registration (conceptual)

```json
{
  "name": "dro_upsert_step_graph",
  "description": "Create or update a fulfillment step graph from a JSON spec. Validates dependency cycles and scope before writing.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "workspaceId": { "type": "string" },
      "spec": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "type": { "type": "string", "enum": ["AutoTask","Callout","ManualTask","Milestone"] },
            "scope": { "type": "string", "enum": ["Plan","Bundle","LineItem","Custom"] },
            "dependsOn": { "type": "array", "items": { "type": "string" } },
            "executeOnCondition": { "type": "string" },
            "customConfigParameter": { "type": "string" }
          },
          "required": ["name","type","scope"]
        }
      },
      "dryRun": { "type": "boolean", "default": true }
    },
    "required": ["workspaceId","spec"]
  }
}
```

## Rollout plan

1. Build Pattern 2 first — the validation action. It is read-only, low risk, and immediately useful to the agentic validation skill.
2. Add Pattern 3 — the step-graph builder — once the validation action proves the Tooling API path is stable.
3. Add Pattern 1 for decomposition rules and scenarios.
4. Register all as Hosted MCP tools; point `lzdravkov/rlm-skills` and Claude Code at them.
5. Keep a living gap table in `interface-coverage.md`; close a row only when the wrapper is tested and audited.

## Anti-patterns to avoid

- Browser automation (Playwright / Browserforce) in any production agent path.
- Direct production writes from an agent without a PR / change-set gate.
- Wrappers that bypass FLS or sharing — always run in user context.
- Fat wrappers that re-implement DRO logic; keep them thin facades over the platform.

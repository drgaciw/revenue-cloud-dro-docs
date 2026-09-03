# Fulfillment Orchestration

## Step graph
Each fulfillment action is a node — token mint, Nexus access grant, audit event — wired with dependencies so a grant cannot fire before the mint succeeds. Per-node failure handling is the lever: retry with backoff on transient errors, compensate on committed steps, park on permanent ones.

## Shallow-graph discipline
Keep the graph shallow: three to five nodes max per commercial product. Past that, the dependency web gets hard to reason about and harder to test. This is exactly where chaos-testing.md earns its keep.

## Async callout pattern
DRO fires an event with the entitlement JSON; middleware mints the token and grants access; the audit system records success or failure. DRO's fulfillment step completes on event dispatch, not on the downstream result. The audit system fills the "did it work" gap and is the better source of truth since it sees the actual download.

## Compensation (recommendation)
Automatic compensation, scoped to steps that actually committed, and only on transient failure. Token minted but grant failed → revoke the token, emit a compensation event the audit system logs. Permanent failures (malformed entitlement JSON, Nexus outage past retry) park for ops — auto-compensating bad input just loops. Rule of thumb: compensate what you can prove, escalate what you can't.

## Line-item design (sub-topic)
**Options:**
1. One line per image — cleanest per-image status and audit granularity, but explodes line count and heavy step graph.
2. One line per commercial product — lighter graph, image set as attributes; loses per-image visibility unless audit parses attributes.
3. Hybrid — one parent line per commercial product, child lines per image the customer actually pulls. Best of both, most complex to build, edge cases in status rollup.

**Recommendation:** hybrid if the audit system handles parent-child relationships cleanly; otherwise one line per commercial product with attributes, since the audit microservice already does the heavy lifting on download receipts.

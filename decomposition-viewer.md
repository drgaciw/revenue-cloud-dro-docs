# Decomposition Viewer

## What it is
A read-only UI that shows the result of decomposition after an order submits: the commercial line, the rule that fired, every technical product produced, and mapped attributes. The decomposition itself runs automatically; the viewer only displays it.

## Human interaction
A fulfillment operator or designer opens it on a submitted order and inspects the graph. Nothing in the viewer changes the outcome — it is a validation and troubleshooting surface, not a control point.

## Use as a launch gate
Run it as a gate in the product-launch checklist: no new commercial SKU ships without a viewer confirmation that the technical output matches the entitlement JSON. For the hundred-image catalog, this is where a rule exploding Product A into ninety-eight images instead of a hundred gets caught.

## Agentic extension
An agent with the right skills can automate the inspection and validation — reading commercial product, rules, execution conditions, and mapped attributes, then flagging mismatches against the entitlement JSON or OCI labels before commit. Keep execution deterministic inside DRO; the agent proposes or validates, never silently rewrites rules or fires live orders.

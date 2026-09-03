# Chaos Testing and Failure Injection

## Goal
Deliberately break middleware or Nexus mid-fulfillment to verify that compensation, retries, and the audit trail fire as designed.

## Scenarios to inject
- Token mint succeeds, Nexus grant fails → confirm token revocation and compensation event.
- Event Grid delivery drops → confirm dead-letter queue capture and safe retry via idempotency key.
- Audit system lag → confirm Grafana alert fires on missing download receipt.
- Permanent failure (malformed entitlement JSON) → confirm order parks for ops, no retry loop.

## Tie-in
Validates the shallow-graph discipline in fulfillment-orchestration.md and the compensation rules in azure-middleware.md. Run before each major catalog or rule change.

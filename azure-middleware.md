# Azure Middleware

## Async callout
DRO dispatches an event with the entitlement JSON; middleware handles token minting and Nexus access grant; the audit system closes the loop. DRO does not block on the downstream result.

## Idempotency (recommendation)
Use a deterministic key: hash entitlement ID + customer ID + image version into one string, passed as the idempotency key on every event. Middleware checks the key before minting — if seen, return the original result. Log the key alongside every event so duplicate attempts trace back to a single source request.

## Durability and delivery
Event Grid provides at-least-once delivery with a seven-day retry window but no ordering or exactly-once guarantee. Harden with a dead-letter queue on every subscription for exhausted retries, Service Bus queues (with sessions) for ordered steps, and dedupe on entitlement ID plus sequence number before writing to the audit log.

## Offline coordination
Coordinate with DRO's offline hold: when middleware or Nexus is unavailable, accept held steps, surface a clear recovery signal, and process the released batch with the same idempotency and compensation rules. Avoid duplicate grants by checking the audit system's state before replaying any held event.

## Compensation logic
On transient failure after committed steps: revoke the token, emit a compensation event. On permanent failure: park for ops. The middleware owns the idempotency contract so retries are safe, not speculative.

## Secrets (Key Vault)
Store Nexus, license-system, and Event Grid credentials in Azure Key Vault. Use managed identities on the middleware so no secret lives in code or config. Version secrets so the middleware always reads the latest without restart; rotate the Nexus service account separately from customer-facing tokens.

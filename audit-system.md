# Audit System

## Role
A middleware audit microservice consumes events from all systems in the fulfillment ecosystem and tracks the full workflow — including the customer's HTTP 200 confirming a successful download. It is the source of truth for "did the customer actually get it."

## Event chain
DRO emits an event on fulfillment-step completion; the audit system records it alongside the download receipt. Compensation events (revoked tokens, failed grants) are first-class logged records so ops can see what was undone and why.

## Negative events and reconciliation
Capture failed downloads, expired-entitlement attempts, and selector mismatches. Where DRO's fulfillment status and the audit status diverge (e.g., download fails after DRO marks complete), the audit system flags it for reconciliation — see azure-middleware.md for the retry and idempotency handling.

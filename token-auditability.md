# Token-Service Auditability

## Chain of custody
Every token mint is an auditable event, not just the download. Log entitlement ID, customer ID, image path, TTL, and the service account that issued it, tied to the same correlation ID the audit microservice uses. Full chain: contract → entitlement JSON → token → download → receipt.

## Negative events
Log the denies too — expired entitlement, selector mismatch, revoked token. These are the leak detectors, and the events nobody captures until after an incident.

## Multi-tenancy context
Nexus Cloud is shared with a degree of customer isolation. Isolation is enforced at the access layer via content selectors and path-based routing, with the token service as the single choke point minting short-lived, scoped credentials. See azure-middleware.md for the implementation pattern.

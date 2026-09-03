# Licensing and Schema Evolution

## Entitlement JSON as contract
The generated JSON — passed to both the license management system and Nexus — is the actual customer entitlement for fulfillment and licensing. The schema contract between those two consumers is the critical seam: if they interpret the JSON differently, you get a silent mismatch between what's licensed and what's downloadable. Flagged for deeper treatment.

## Schema evolution (recommendation)
Version the JSON schema. Add fields additively and keep them backward-compatible so existing entitlements continue to validate. RCA, DRO, the license system, and Nexus each consume a pinned schema version; introduce a new version only when a breaking change is unavoidable, with a migration window. Treat a new attribute (e.g., image version constraint) as flowing through the same pipeline as any other entitlement change.

## Revocation (recommendation)
Time-to-revoke SLA under five minutes, enforced at the token service, not Nexus. On revocation event: middleware stops minting new tokens for that entitlement and publishes a deny-list the audit system checks on every download attempt. Existing tokens die with their TTL — keep TTL under an hour. Log every revocation as a first-class audit event with the trigger (contract expiry, seat reduction, manual) for compliance proof. Revocation is itself a schema event — this entitlement is now invalid — flowing through the same pipeline as schema evolution.

# DRO Mapping: Commercial to Technical Products

## Objective
Decomposition turns a quoted commercial product into the technical products DRO actually fulfills — image sets, entitlement data, and downstream callouts.

## Core pattern
- Commercial product = what sales quotes (e.g., Product A).
- Technical product = what fulfillment executes (e.g., individual OCI images or a bundled set).
- Decomposition rules define the mapping; execution rules gate when it fires; attribute mapping stamps data onto fulfillment lines.

## Execution rules (recommendation)
Keep execution rules shallow and data-driven. Gate on a small set of order attributes — tier, region, contract type — never on complex logic or external lookups. Complex rules become untestable and break silently when catalog data shifts. Treat execution rules as part of the decomposition design, not a standalone concern.

## Attribute mapping (recommendation)
- **As Is** for identity fields: customer ID, entitlement ID.
- **List Mapping** for version tags and tier-to-image-set translations.
- **Expression Set** only for computed values, e.g., deriving an image subset from a seat count.
Keep expressions simple and unit-testable; complex expressions are where mapping bugs hide.

## Line-item design (sub-topic)
See fulfillment-orchestration.md for the full options analysis. Summary: prefer the hybrid (one parent line per commercial product, child lines per image the customer actually pulls) if the audit system handles parent-child cleanly; otherwise one line per commercial product with the image list as attributes, relying on the audit microservice for download granularity.

## Entitlement catalog vs hardcoded mapping
Maintain a separate entitlement catalog — a registry mapping each commercial product to its image set, versioned independently of the product. DRO reads it at decomposition time. This avoids product revisions on every catalog change and gives one place to audit licensed vs downloadable images.

## OCI labels as discovery layer
Build and release teams embed OCI labels and manifests on every image describing relationships to commercial products. Labels serve discovery and relationship; the entitlement JSON remains authoritative for licensing. An agent reading manifests is a different skill than reading Salesforce metadata — see agentic-dro.md.

## Decomposition Viewer
See decomposition-viewer.md. Run a test order, confirm the commercial line exploded into the expected image set, catch bad mappings before they hit Nexus.

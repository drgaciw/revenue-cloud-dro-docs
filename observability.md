# Observability

## Single correlation-driven view
Stream audit, DRO, and middleware events into a common store (Azure Monitor or a time-series backend). Build Grafana dashboards keyed on the correlation ID so ops can trace one entitlement from contract to download in a single view.

## Alerting
A Grafana rule fires when a fulfillment event has no matching download receipt within a defined threshold — the failure mode where a token is minted but the download never lands.

## Negative-event visibility
Surface denies and compensation events alongside successes so partial-failure states are visible, not just happy-path throughput.

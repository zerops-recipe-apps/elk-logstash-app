# elk-logstash-app

Zerops recipe that installs upstream Logstash (`logstash=1:8.17.4-1`) via apt, ingests syslog over UDP/TCP on port 1514, and forwards events to a sibling Elasticsearch service.

## Zerops service facts

- Ports: `1514` (UDP + TCP syslog)
- Siblings: `elkstorage` (Elasticsearch) — env: `ELASTICSEARCH_URL`, `ELASTICSEARCH_PASSWORD`
- Runtime base: `ubuntu@24.04`

## Zerops

No dev iteration loop — the app is an upstream Elastic binary installed via apt in `prepareCommands`. Changes in this repo only affect the `logstash/` config directory and `zerops.yaml` install pins. Each change requires a full build+deploy through the **Zerops development workflow via `zcp` MCP tools**.

## Notes

- Logstash version pinned to `logstash=1:8.17.4-1` in `prepareCommands`; bump the pin to upgrade.
- Keep version in sync with `elk-kibana-app`, `elk-apm-server-app`, and the `elkstorage` Elasticsearch service.

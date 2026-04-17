# elk-logstash-app

Zerops recipe that installs upstream Logstash (`logstash=1:8.16.6-1`) via apt, ingests syslog over UDP/TCP on port 1514, and forwards events to a sibling Elasticsearch service.

## Zerops service facts

- Ports: `1514/udp` + `1514/tcp` (syslog ingestion, no HTTP)
- Siblings: `elkstorage` (Elasticsearch) — env: `ELASTICSEARCH_URL`, `ELASTICSEARCH_PASSWORD`
- Runtime base: `ubuntu@24.04`

## Zerops

No dev iteration loop — the app is an upstream Elastic binary installed via apt in `prepareCommands`. Changes in this repo only affect the `logstash/` config directory (pipeline, JVM options) and `zerops.yaml` install pins. Each change requires a full build+deploy through the **Zerops development workflow via `zcp` MCP tools**.

## Notes

- Logstash version pinned to `logstash=1:8.16.6-1` in `prepareCommands`; bump the pin to upgrade.
- Default JVM heap is `256m`; override with `LOGSTASH_HEAP_MEMORY_OVERRIDE` env var.
- `logstash/` and `logstash/conf.d/` use `%%VAR%%` placeholders replaced by `envReplace` at deploy time; to forward Zerops project logs, target `logstash:1514` (UDP internal) or the service's public IPv6 (TCP external).

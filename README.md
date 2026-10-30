# Zabbix template: LM Studio API

Monitoring for [LM Studio](https://lmstudio.ai) local server on Zabbix 7.x.

## Endpoint

`GET http://{HOST.CONN}:{$LMSTUDIO_PORT}/api/v0/models` — deliberately the **v0**-compatible path:
it returns a flat list with a `state: loaded|not-loaded` field, while v1 (`/api/v1/models`)
nests `loaded_instances[]` and is harder to parse with JSONPath.

## Items

- `lmstudio.models.count` / `lmstudio.models.list` — all models
- `lmstudio.models.loaded.count` / `lmstudio.models.loaded.list` — only `state == loaded`
- `net.tcp.service[http,,{$LMSTUDIO_PORT}]` — simple port check

## LLD

`lmstudio.models.discovery` (from `/api/v0/models`) — per model: state item and a loaded flag.
Trigger prototype: "LM Studio: model {#MODELID} unloaded from memory".

## Triggers

| Event | Severity | Condition |
|---|---|---|
| LM Studio: port {$LMSTUDIO_PORT} unreachable | High | TCP port closed |
| LM Studio: no model loaded | Warning | port alive, loaded list empty |

## Macros

| Macro | Default |
|---|---|
| `{$LMSTUDIO_PORT}` | `1234` |

## Setup

1. Import `template_lmstudio.yaml`.
2. Host with an Agent interface (`{HOST.CONN}`), link the template.
3. LM Studio must be running as a server (Developer → Start Server).

---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: HTTP Streaming and SSE
title: HTTP Streaming and Server-Sent Events (SSE)
description: Transparent long-lived HTTP streams and SSE using the no-op proxy encoding
weight: 150
menu:
  community_current:
    parent: "040 Routing and Forwarding"
meta:
  since: v2.0
  source: https://github.com/pucora/lura
  namespace: false
  scope:
  - endpoint
  - backend
---

Pucora supports transparent HTTP streaming and Server-Sent Events (SSE) using the **no-op proxy** model. The gateway forwards response chunks as they arrive without buffering or transforming the payload.

No separate module or `extra_config` namespace is required — streaming is enabled through endpoint and backend encoding settings.

## When to use streaming vs WebSockets

| | SSE / HTTP streaming | WebSockets |
|---|---------------------|------------|
| Direction | Server → client | Bidirectional |
| Protocol | Plain HTTP | RFC-6455 upgrade |
| Config | `no-op` encoding | `extra_config.websocket` |
| Best for | Notifications, LLM tokens, logs | Chat, gaming, binary bidirectional |

## Quick start

```bash
make sse-compose-test
```

Or manually:

```bash
cd examples/streaming
docker compose up --build
curl -N http://localhost:8080/events
```

## Configuration

Three requirements:

1. **`output_encoding: "no-op"`** on the endpoint
2. **`encoding: "no-op"`** on the backend
3. **Long `timeout` on the endpoint** (not at service level)

### SSE example

```json
{
  "version": 3,
  "port": 8080,
  "write_timeout": "0s",
  "endpoints": [
    {
      "endpoint": "/weather-stream",
      "method": "POST",
      "timeout": "300s",
      "output_encoding": "no-op",
      "input_headers": ["Content-Type"],
      "backend": [
        {
          "host": ["https://weather.example.com"],
          "url_pattern": "/api/agents/weatherAgent/stream",
          "encoding": "no-op"
        }
      ]
    }
  ]
}
```

### Generic HTTP streaming (video)

```json
{
  "endpoint": "/video/{id}",
  "method": "GET",
  "timeout": "5m",
  "output_encoding": "no-op",
  "input_headers": ["Content-Type"],
  "backend": [
    {
      "host": ["https://videos.example.com"],
      "url_pattern": "/video/{id}.mkv",
      "encoding": "no-op"
    }
  ]
}
```

## Configuration reference

| Setting | Level | Description |
|---------|-------|-------------|
| `output_encoding` | Endpoint | Must be `"no-op"` |
| `encoding` | Backend | Must be `"no-op"` |
| `timeout` | Endpoint | Max stream duration (e.g. `"300s"`, `"5m"`) |
| `write_timeout` | Service | **Required `"0s"`** when streaming endpoints exist |
| `response_header_timeout` | Service | Use `"0s"` or at least `"30s"` when streaming |

**Important:** Keep the root service `timeout` short. Set long timeouts only on streaming endpoints. Pucora **rejects invalid streaming configs at startup** (`pucora check` / `pucora run`).

## Compatible middleware

| Component | Streaming-safe? |
|-----------|-----------------|
| JWT (`auth/validator`) | Yes (request phase) |
| Rate limiting | Yes |
| CORS | Yes |
| Lua post-response | **No** |
| Response JSON schema | **No** |
| Multi-backend merge | **No** (rejected at startup) |

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Events arrive all at once at end | Proxy or LB buffering | Ensure `no-op` encoding; set `write_timeout: "0s"` |
| Connection drops early | Timeout too short | Increase endpoint `timeout` |
| Config fails `pucora check` | Incompatible middleware | Remove response Lua, schema, or multi-backend |

## Requirements

- `github.com/pucora/lura/v2` **v2.0.3+** (flush-aware streaming proxy and startup validation)

## Local example

See [examples/streaming](https://github.com/pucora/velonetics-ce/tree/main/examples/streaming).

## Related

- [No-op proxy](/docs/endpoints/no-op/) — encoding fundamentals
- [WebSockets](/docs/websockets/) — bidirectional real-time
- [Pucora Configurator](/docs/configuration/configurator/) — `streaming-sse` preset

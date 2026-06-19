---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: WebSockets
title: WebSockets Integration
description: Bidirectional real-time communication using RFC-6455 WebSockets with multiplex and direct proxy modes
weight: 200
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/pucora/velonetics-websocket
  namespace:
  - websocket
  scope:
  - endpoint
---

Pucora supports bidirectional communication using the [WebSocket protocol (RFC-6455)](https://datatracker.ietf.org/doc/html/rfc6455). Clients connect to the gateway over WebSocket; the gateway connects to one or more backend hosts using `ws://` or `wss://`.

Implemented by [`velonetics-websocket`](https://github.com/pucora/velonetics-websocket) and configured per endpoint via `extra_config.websocket`.

## Operating modes

| Mode | Config | Client ↔ Gateway | Gateway ↔ Backend | Best for |
|------|--------|------------------|-------------------|----------|
| **Multiplexing** (default) | `enable_direct_communication: false` | One WebSocket per client | **One shared** WebSocket per endpoint | Chat rooms, fan-out, many clients |
| **Direct** | `enable_direct_communication: true` | One WebSocket per client | **One WebSocket per client** | Transparent proxy, binary streams, subprotocols |

Multiplexing is recommended when many clients talk to the same backend service. Direct mode is simpler but opens more backend connections.

## Quick start (direct echo)

**1.** Run any WebSocket echo server on `ws://127.0.0.1:8081/`.

**2.** Gateway config (`pucora.json`):

```json
{
  "version": 3,
  "port": 8080,
  "endpoints": [
    {
      "endpoint": "/ws/echo",
      "method": "GET",
      "backend": [
        {
          "host": ["ws://127.0.0.1:8081"],
          "url_pattern": "/",
          "disable_host_sanitize": true
        }
      ],
      "extra_config": {
        "telemetry/usage": { "enabled": false },
        "websocket": {
          "enable_direct_communication": true,
          "max_message_size": 4096
        }
      }
    }
  ]
}
```

**3.** Run and test:

```bash
pucora run -c pucora.json
websocat ws://localhost:8080/ws/echo
```

## Requirements

Every WebSocket endpoint must satisfy:

1. **`extra_config.websocket`** — present (an empty object `{}` is valid).
2. **Backend `host`** — must use `ws://` or `wss://` (not `http://`).
3. **`disable_host_sanitize: true`** on the backend block.
4. **`method: "GET"`** — WebSocket upgrades use HTTP GET.

Optional: **`auth/validator`** validates JWT on the **HTTP upgrade request** only (not on each frame).

## Configuration reference

All settings live under `endpoints[].extra_config.websocket`:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enable_direct_communication` | bool | `false` | `true` = direct proxy; `false` = multiplexing |
| `input_headers` | `[]string` | `[]` | Headers forwarded to the backend |
| `subprotocols` | `[]string` | `[]` | Subprotocols offered to clients (**direct mode**) |
| `max_message_size` | int | `512` | Max message size in bytes |
| `message_buffer_size` | int | `256` | Per-client outbound queue size (**multiplex**) |
| `max_retries` | int | `0` | Backend reconnect attempts; `0` = unlimited |
| `backoff_strategy` | string | `fallback` | `linear`, `linear-jitter`, `exponential`, `exponential-jitter`, `fallback` |
| `connect_event` | bool | `false` | Notify backend when a client connects (**multiplex**) |
| `disconnect_event` | bool | `false` | Notify backend when a client disconnects (**multiplex**) |
| `return_error_details` | bool | `false` | Send `{"error":"..."}` to client when backend write fails |
| `disable_otel_metrics` | bool | `false` | Disable OpenTelemetry WebSocket metrics |
| `ping_period` | duration | `54s` | Client ping interval |
| `pong_wait` | duration | `60s` | Pong timeout |
| `write_wait` | duration | `10s` | Max time to write a frame to the client |
| `timeout` | duration | `5m` | Per-read timeout on client and backend connections |

## Multiplexing mode

Many clients share one backend WebSocket. The gateway wraps traffic in a JSON **envelope** on the backend leg only. Clients send and receive **plain** WebSocket frames.

### Backend handshake

When the gateway opens the backend connection it sends:

```json
{"msg":"Pucora WS proxy starting"}
```

The backend **must** reply with the plain text string `OK` before client traffic is forwarded.

### Envelope format (gateway ↔ backend)

**Client → backend:**

```json
{
  "url": "/ws/chat/general",
  "session": {
    "uuid": "0b251b07-5611-49e5-b69f-cf2cb8d339d6",
    "Room": "general"
  },
  "body": "SGVsbG8gV29ybGQh"
}
```

| Field | Description |
|-------|-------------|
| `url` | Client request path on the gateway |
| `session` | Includes `uuid` plus endpoint placeholders (first letter capitalized, e.g. `{room}` → `Room`) |
| `body` | Base64-encoded payload from the client |

**Backend → clients:**

| Payload | Delivery |
|---------|----------|
| `{ "body": "<base64>" }` only | **Broadcast** to all clients |
| `{ "url": "/ws/chat/general", "body": "<base64>" }` | **Multicast** to clients on that path |
| `{ "session": { "uuid": "..." }, "body": "<base64>" }` | **Unicast** to one client |

### Example multiplex config

```json
{
  "endpoint": "/ws/{room}",
  "method": "GET",
  "input_query_strings": ["*"],
  "backend": [
    {
      "host": ["ws://127.0.0.1:8081"],
      "url_pattern": "/ws",
      "disable_host_sanitize": true
    }
  ],
  "extra_config": {
    "websocket": {
      "input_headers": ["Authorization", "Cookie"],
      "connect_event": true,
      "disconnect_event": true,
      "max_message_size": 3200000,
      "message_buffer_size": 256,
      "max_retries": 0,
      "backoff_strategy": "exponential"
    }
  }
}
```

## Direct mode

- No handshake and no envelope — binary and text frames pass through transparently.
- Supports `subprotocols` in config.
- Set `"enable_direct_communication": true` in `websocket` config.

## Authentication (JWT)

```json
"extra_config": {
  "auth/validator": {
    "alg": "RS256",
    "jwk_url": "https://your-idp/.well-known/jwks.json",
    "audience": ["your-api"],
    "roles": ["user"]
  },
  "websocket": {
    "input_headers": ["Authorization"]
  }
}
```

After a successful upgrade, frames are not re-validated. Issue short-lived tokens or reconnect when tokens expire.

## Metrics (OpenTelemetry)

| Metric | Description |
|--------|-------------|
| `pucora.websocket.connections` | Active client connections |
| `pucora.websocket.messages.in` | Messages from clients |
| `pucora.websocket.messages.out` | Messages to clients |
| `pucora.websocket.reconnects` | Backend reconnect attempts |

## What is not supported

- **Socket.IO** — not compatible with plain RFC-6455 multiplexing.
- **Per-frame JWT** — auth applies at upgrade only.
- **HTTP proxy pipeline** — WebSocket endpoints bypass the normal request/response proxy.

## Local example

From the Pucora CE repository:

```bash
make ws-compose-test
```

See [examples/websocket](https://github.com/pucora/velonetics-ce/tree/main/examples/websocket) for Docker Compose stacks covering direct, multiplex, and JWT-protected endpoints.

## Related

- [HTTP Streaming & SSE](/docs/endpoints/streaming/) — server-to-client streams over plain HTTP
- [Pucora Configurator](/docs/configuration/configurator/) — `websocket` preset for quick setup

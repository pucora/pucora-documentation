---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: gRPC Server
title: gRPC Server
description: Expose gRPC services on the same port as the Pucora HTTP gateway
weight: 220
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/pucora/velonetics-grpc
  namespace:
  - grpc.server
  scope:
  - service
---

Pucora can **publish gRPC services** on the same port as HTTP. Each method maps to one or more backends (REST, gRPC, or mixed).

gRPC reflection is enabled automatically. Use `grpcurl -protoset flights.pb -plaintext localhost:8080 list` to discover services.

## Configuration

```json
{
  "extra_config": {
    "grpc": {
      "catalog": ["./grpc/definitions"],
      "server": {
        "services": [{
          "name": "flight_finder.Flights",
          "methods": [{
            "name": "FindFlight",
            "input_headers": ["*"],
            "payload_params": { "page.cursor": "cursor" },
            "backend": [{
              "host": ["http://localhost:8000"],
              "url_pattern": "/flights"
            }]
          }]
        }]
      }
    }
  }
}
```

## Per-method JWT

```json
{
  "name": "FindFlight",
  "input_headers": ["authorization"],
  "extra_config": {
    "auth/validator": {
      "alg": "HS256",
      "audience": ["http://api.example.com"],
      "roles_key": "roles",
      "roles": ["role_a", "role_b"],
      "jwk_url": "http://identity:8081/jwk/symmetric",
      "disable_jwk_security": true
    }
  },
  "backend": [...]
}
```

## Server OpenTelemetry overrides

Under `grpc.server.opentelemetry`:

| Field | Description |
|-------|-------------|
| `disable_metrics` | Disable gRPC server metrics |
| `disable_traces` | Disable gRPC server traces |

## Local example

```bash
make grpc-compose-test   # includes server smoke via grpcurl
```

Config: `velonetics-server.json` in [examples/grpc](https://github.com/pucora/velonetics-ce/tree/main/examples/grpc).

## Related

- [Introduction to gRPC](/docs/grpc/)
- [gRPC client](/docs/backends/grpc/)

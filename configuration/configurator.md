---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: Pucora Configurator
title: Pucora Configurator
description: Generate pucora.json from YAML profiles and built-in presets for REST, gRPC, WebSockets, pub/sub, and more
weight: -900
menu:
  community_current:
    parent: "010 Configuration files"
meta:
  since: v2.0
  source: https://github.com/pucora/pucora-configurator
  scope:
  - service
---

The **Pucora Configurator** turns a simple YAML profile into a complete gateway setup — routes, CORS, allowed headers, JWT auth, pub/sub, gRPC, WebSockets, and more.

No more hand-writing `extra_config` namespaces or remembering `disable_host_sanitize` rules.

## Quick start

```bash
# Build
make build

# Interactive wizard
./bin/pucora-config init -o my-profile.yaml

# Or pick a preset
./bin/pucora-config presets list
./bin/pucora-config presets apply rest-proxy -g ./output

# Generate from your profile
./bin/pucora-config generate -f my-profile.yaml -o ./output

# Validate before generating
./bin/pucora-config validate -f my-profile.yaml
```

Run the gateway:

```bash
pucora check -c ./output/pucora.json
pucora run -c ./output/pucora.json
```

## Why this exists

Configuring Pucora directly requires knowing:

- Which `extra_config` namespace to use (`security/cors`, `backend/pubsub/publisher`, `backend/grpc`, …)
- That headers are **deny-by-default** — you must set `input_headers` per route
- Non-HTTP backends need `disable_host_sanitize: true` and scheme-specific `host` values (`kafka://`, `ws://`, …)
- gRPC needs a service-level catalog plus per-backend `input_mapping`
- WebSockets need four separate settings across endpoint and backend

The configurator hides that complexity behind a **simple profile format** and **ready-made presets**.

## Profile format

```yaml
apiVersion: configurator.pucora.in/v1
kind: GatewayProfile
metadata:
  name: My API Gateway

gateway:
  port: 8080
  timeout: 3s

cors:
  enabled: true
  allow_origins: [http://localhost:3000]
  allow_headers: [Origin, Authorization, Content-Type]

routes:
  - path: /api/{path}
    method: GET
    headers:
      forward: [Authorization, Content-Type]
    query_strings:
      forward: [page, limit]
    auth:
      jwt:
        alg: RS256
        jwk_url: https://idp.example.com/.well-known/jwks.json
        audience: [https://api.example.com]
    backend:
      type: http
      host: http://localhost:8000
      path: /{path}
```

## Built-in presets

| Preset | Description |
|--------|-------------|
| `rest-proxy` | HTTP reverse proxy with CORS and header forwarding |
| `rest-with-auth` | REST proxy with JWT, rate limiting, and CORS |
| `kafka-pubsub` | Kafka publish/subscribe REST endpoints |
| `nats-pubsub` | NATS publish/subscribe |
| `grpc-client` | REST → gRPC with catalog and input mapping |
| `websocket` | WebSocket proxy with optional JWT |
| `streaming-sse` | Server-sent events / streaming |

```bash
pucora-config presets apply kafka-pubsub -g ./output
```

## Commands

| Command | Description |
|---------|-------------|
| `init` | Interactive wizard — choose pattern, answer prompts, get a profile |
| `generate -f profile.yaml -o ./output` | Produce `pucora.json` and `.env` |
| `validate -f profile.yaml` | Check profile without generating |
| `presets list` | Show built-in presets |
| `presets apply <name>` | Copy preset and/or generate config |

## What gets generated

| Output | Contents |
|--------|----------|
| `pucora.json` | Full gateway config with correct namespaces |
| `.env` | Broker connection strings (`KAFKA_BROKERS`, `NATS_SERVER_URL`, …) |

The generator automatically sets:

- `$schema` and `version: 3`
- `security/cors`, `telemetry/logging`, `telemetry/metrics`
- `input_headers` and `input_query_strings` per route
- `auth/validator`, `qos/ratelimit/router`, `websocket` extra_config
- `disable_host_sanitize`, pub/sub namespaces, gRPC `input_mapping`
- `grpc.catalog` at service level when configured
- `async_agent` blocks for background Kafka consumers

## Backend types

| `backend.type` | Use for |
|----------------|---------|
| `http` | Standard REST proxy |
| `grpc` | Upstream gRPC services |
| `websocket` | WebSocket backends (`ws://` / `wss://`) |
| `kafka`, `nats`, `rabbit` | Message broker pub/sub |
| `gcp`, `azure`, `aws_sns`, `aws_sqs` | Cloud pub/sub |
| `graphql` | GraphQL adapter |
| `soap` | SOAP/XML backends |

## Related

- [Configuration overview](/docs/configuration/)
- [Flexible configuration](/docs/configuration/flexible-config/)
- [WebSockets](/docs/websockets/)
- [Introduction to gRPC](/docs/grpc/)

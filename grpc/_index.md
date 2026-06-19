---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: Introduction to gRPC
title: Introduction to gRPC
description: gRPC catalog setup, client backends, and server exposure on the Pucora gateway port
weight: 210
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/pucora/velonetics-grpc
  namespace:
  - grpc
  scope:
  - service
  - backend
---

Pucora exposes **gRPC client** and **gRPC server** integration with unary RPC support. Catalog services use compiled `.pb` descriptor files.

Implemented by [`velonetics-grpc`](https://github.com/pucora/velonetics-grpc) via:

- `extra_config.grpc` — service catalog and optional gRPC server
- `extra_config.backend/grpc` — gRPC upstream backends

## Catalog setup

Generate `.pb` files from `.proto`:

```bash
protoc --descriptor_set_out=flights.pb flights.proto
```

Service-level config:

```json
{
  "extra_config": {
    "grpc": {
      "catalog": ["./grpc/flights.pb", "./grpc/definitions"]
    }
  }
}
```

## Feature overview

| Capability | Doc | Namespace |
|------------|-----|-----------|
| REST → gRPC upstream calls | [gRPC client](/docs/backends/grpc/) | `backend/grpc` |
| Expose gRPC on gateway port | [gRPC server](/docs/grpc/server/) | `grpc.server` |
| Mixed HTTP + gRPC endpoints | Both docs | Combined config |

## Rules

| Rule | Value |
|------|-------|
| `host` (client backend) | `host:port` only (no `http://`) |
| `url_pattern` | `/package.Service/Method` |
| Streaming | Not supported in v1 |
| Catalog | Requires `.pb` files at runtime |

## Local examples

From the Pucora CE repository:

| Config | Make target | Smoke test |
|--------|-------------|------------|
| `pucora.json` | `grpc-compose-test` | REST `/flights` |
| `velonetics-server.json` | server variant | `grpcurl` FindFlight |
| `velonetics-mixed.json` | mixed variant | REST + `grpcurl` |
| `velonetics-jwt.json` | JWT variant | auth required |

```bash
make test-grpc
make grpc-compose-test
```

See [examples/grpc](https://github.com/pucora/velonetics-ce/tree/main/examples/grpc).

## Limitations

- Unary RPC only (no streaming)
- Catalog requires `.pb` files (not raw `.proto` at runtime)
- Arrays of objects cannot be filled via query strings

## Related

- [Pucora Configurator](/docs/configuration/configurator/) — `grpc-client` preset
- [gRPC client backend](/docs/backends/grpc/)
- [gRPC server](/docs/grpc/server/)

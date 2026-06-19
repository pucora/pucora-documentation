---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: gRPC client
title: gRPC Client and REST to gRPC Conversion
description: Call upstream gRPC services from REST endpoints using a protobuf catalog
weight: 215
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/pucora/velonetics-grpc
  namespace:
  - backend/grpc
  scope:
  - backend
---

The **gRPC client** backend lets REST clients call upstream gRPC services. Pucora maps HTTP query parameters, headers, and body fields to protobuf messages using the service catalog.

Requires a service-level `grpc.catalog` — see [Introduction to gRPC](/docs/grpc/).

## Quick start

```json
{
  "extra_config": {
    "grpc": { "catalog": ["grpcatalog/flights.pb"] }
  },
  "endpoints": [{
    "endpoint": "/flights",
    "input_query_strings": ["*"],
    "backend": [{
      "host": ["localhost:4242"],
      "url_pattern": "/flight_finder.Flights/FindFlight",
      "extra_config": { "backend/grpc": {} }
    }]
  }]
}
```

| Rule | Value |
|------|-------|
| `host` | `host:port` only (no `http://`) |
| `url_pattern` | `/package.Service/Method` |

## Key `backend/grpc` fields

| Field | Description |
|-------|-------------|
| `use_request_body` | Fill request from HTTP body |
| `input_mapping` | Map query/placeholder params to nested fields |
| `header_mapping` | Map HTTP headers to gRPC metadata |
| `request_naming_convention` | `snake_case` (default) or `camelCase` |
| `response_naming_convention` | `snake_case` (default) or `camelCase` |
| `client_tls` | TLS client settings |
| `read_buffer_size` | gRPC client read buffer (bytes); `0` = default |
| `use_alternate_host_on_error` | Skip bad connections and retry alternate hosts |

## Input mapping example

```json
{
  "endpoint": "/flights/{origin}/{destination}",
  "input_query_strings": ["date"],
  "backend": [{
    "host": ["localhost:4242"],
    "url_pattern": "/flight_finder.Flights/FindFlight",
    "extra_config": {
      "backend/grpc": {
        "input_mapping": {
          "origin": "Origin",
          "destination": "Destination",
          "date": "TravelDate"
        }
      }
    }
  }]
}
```

## TLS

```json
"extra_config": {
  "backend/grpc": {
    "client_tls": {
      "@comment": "Use allow_insecure_connections only in development",
      "allow_insecure_connections": false,
      "ca_certs": ["./certs/ca.pem"],
      "client_certs": ["./certs/client.pem"],
      "client_key": "./certs/client-key.pem"
    }
  }
}
```

## gRPC-to-gRPC passthrough

When a published gRPC server method has a single `backend/grpc` backend, the gateway forwards protobuf directly without JSON conversion. See [gRPC server](/docs/grpc/server/).

## Local example

```bash
make grpc-compose-test
```

## Related

- [Introduction to gRPC](/docs/grpc/)
- [gRPC server](/docs/grpc/server/)

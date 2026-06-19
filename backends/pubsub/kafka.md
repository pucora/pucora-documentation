---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: Kafka Advanced PubSub
title: Kafka PubSub with Extended Connectivity
description: Kafka publisher and subscriber with mTLS, SASL, Azure Event Hub, and idempotent producers
weight: 135
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/pucora/velonetics-pubsub
  namespace:
  - backend/pubsub/publisher/kafka
  - backend/pubsub/subscriber/kafka
  scope:
  - backend
---

Use the **Kafka advanced** driver instead of the basic `kafka://` scheme when you need explicit broker connection settings, mTLS, or SASL.

Namespaces: `backend/pubsub/publisher/kafka` and `backend/pubsub/subscriber/kafka`.

## Capabilities

- mTLS with CA and client certificates
- SASL PLAIN and OAUTHBEARER authentication
- Azure Event Hub via `azure_event_hub: true` (SASL handshake v0)
- Producer `idempotent` mode (maps to `RequiredAcks=all`)
- Consumer group and isolation level configuration
- Message key from HTTP headers via `key_meta`
- Shared `cluster` config with the [`async/kafka`](/docs/async/kafka/) agent driver

## Publisher

```json
{
  "host": ["ignore"],
  "url_pattern": "/ignore",
  "disable_host_sanitize": true,
  "extra_config": {
    "backend/pubsub/publisher/kafka": {
      "success_status_code": 201,
      "writer": {
        "topic": "events",
        "key_meta": "X-Event-Key",
        "cluster": {
          "brokers": ["localhost:9092"],
          "client_id": "velonetics_publisher",
          "sasl": {
            "user": "user",
            "password": "password"
          }
        },
        "producer": {
          "idempotent": true
        }
      }
    }
  }
}
```

## Subscriber

```json
{
  "host": ["ignore"],
  "url_pattern": "/ignore",
  "disable_host_sanitize": true,
  "encoding": "json",
  "extra_config": {
    "backend/pubsub/subscriber/kafka": {
      "reader": {
        "topics": ["events"],
        "key_meta": "X-Event-Key",
        "cluster": {
          "brokers": ["localhost:9092"],
          "client_id": "velonetics_subscriber"
        },
        "group": {
          "id": "my_group_id",
          "isolation_level": "read_commited"
        }
      }
    }
  }
}
```

## Azure Event Hub

Event Hubs require SASL handshake v0. Set `azure_event_hub: true` and use PLAIN with the connection string as the password:

```json
"cluster": {
  "brokers": ["my-namespace.servicebus.windows.net:9093"],
  "client_tls": {
    "allow_insecure_connections": false
  },
  "sasl": {
    "mechanism": "PLAIN",
    "user": "$ConnectionString",
    "password": "Endpoint=sb://...",
    "azure_event_hub": true
  }
}
```

## OAUTHBEARER

```json
"sasl": {
  "mechanism": "OAUTHBEARER",
  "password": "<access-token>",
  "auth_identity": "optional-authzid"
}
```

## Offset commits

Kafka has no per-message ACK. Pucora commits partition offsets after each message is read, including when the decode step fails after a successful fetch.

## Local example

```bash
make pubsub-kafka-advanced-compose-test
```

See [examples/pubsub/kafka-advanced](https://github.com/pucora/velonetics-ce/tree/main/examples/pubsub/kafka-advanced).

## Related

- [Basic Pub/Sub](/docs/backends/pubsub/)
- [Async Kafka agent](/docs/async/kafka/)

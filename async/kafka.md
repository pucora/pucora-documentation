---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: Kafka Async Driver
title: Kafka driver for the Asynchronous Agent
description: Background Kafka consumer agents that forward events to internal webhooks
weight: 520
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/pucora/velonetics-pubsub
  namespace:
  - async/kafka
  scope:
  - async_agent
---

The **Kafka async driver** connects a background Pucora agent to Kafka topics for autonomous event consumption. Unlike endpoint Kafka subscribers, no HTTP client triggers processing.

Namespace: `async/kafka` inside `async_agent[].extra_config`.

## How it works

```
Kafka topic -> async/kafka driver -> internal proxy pipe -> backend webhook(s)
```

The topic name comes from `async_agent[].consumer.topic`. The driver uses a **latest offset** policy: only messages produced after the agent starts are consumed.

## Key capabilities

- mTLS and SASL PLAIN / OAUTHBEARER (shared `cluster` config with [Kafka Advanced Pub/Sub](/docs/backends/pubsub/kafka/))
- Azure Event Hub via `sasl.azure_event_hub: true`
- Consumer group management (`group.id` or `group.group_id`)
- Message key exposed as HTTP header via `key_meta`
- Worker pool via `consumer.workers`
- Optional rate limiting via `consumer.max_rate`
- Health pings to the gateway `__health` endpoint

## Configuration

```json
{
  "version": 3,
  "async_agent": [
    {
      "name": "events-consumer",
      "consumer": {
        "topic": "events",
        "workers": 2,
        "timeout": "2s"
      },
      "backend": [
        {
          "host": ["http://analytics:8080"],
          "url_pattern": "/ingest",
          "method": "POST"
        }
      ],
      "connection": {
        "max_retries": 3,
        "backoff_strategy": "linear",
        "health_interval": "30s"
      },
      "extra_config": {
        "async/kafka": {
          "cluster": {
            "brokers": ["localhost:9092"],
            "client_id": "velonetics_async_agent"
          },
          "group": {
            "group_id": "my_group_id",
            "isolation_level": "read_commited"
          },
          "key_meta": "Message-Id"
        }
      }
    }
  ]
}
```

## Driver fields

| Field | Description |
|-------|-------------|
| `cluster` | Broker connection (brokers, TLS, SASL) |
| `group` | Consumer group settings (`id` or `group_id`) |
| `key_meta` | HTTP header name for the Kafka message key |

### Cluster fields

| Field | Default | Description |
|-------|---------|-------------|
| `brokers` | — | Kafka broker addresses (required) |
| `client_id` | Pucora version string | Client identifier |
| `client_tls` | — | mTLS settings |
| `sasl` | — | `{ mechanism, user, password, azure_event_hub, auth_identity }` |
| `metadata_retry_max` | `3` | Reconnect attempts |
| `metadata_retry_backoff` | `250ms` | Reconnect delay |

### Consumer group fields

| Field | Default | Description |
|-------|---------|-------------|
| `group_id` / `id` | agent name | Consumer group ID |
| `isolation_level` | `read_commited` | `read_commited` or `read_uncommited` |

## Offset commits

Partition offsets are committed after each message is read, even when the backend pipeline fails. Design downstream services for idempotency.

## Local example

```bash
make pubsub-async-kafka-compose-test
```

See [examples/pubsub/async-kafka](https://github.com/pucora/velonetics-ce/tree/main/examples/pubsub/async-kafka).

## Related

- [Async agents overview](/docs/async/)
- [AMQP async driver](/docs/async/amqp/)
- [Kafka Advanced Pub/Sub](/docs/backends/pubsub/kafka/)

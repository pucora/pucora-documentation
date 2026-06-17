---
lastmod: 2024-05-11
date: 2024-05-11
linktitle: Introduction to Non-REST Connectivity
title: Non-REST Connectivity
description: Explore Velonetics's non-REST connectivity features, including RabbitMQ integration, GraphQL, Lambda functions, event-driven async agents, and AMQP support.
weight: -10
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
dark_header_image: true
images:
- /images/documentation/hero/non-rest-connectivity.png
---

Velonetics supports a range of non-REST connectivity options that open the door to integration with message brokers, event-driven systems, and alternative protocols and communication styles. By providing built-in capabilities to interact with services beyond traditional REST APIs, Velonetics enables applications to handle complex, real-time data flows and asynchronous communication patterns.

You can connect to non-REST options using a REST interface and mix multiple services at once. For instance, imagine you have a REST endpoint `/subscribe`  targeted for a mobile application that inserts payload directly in a RabbitMQ when called. You could even make the gateway react to a new event on a queue (an Async Agent), regardless of the origin, and call a webhook inside your infrastructure with its payload.

These are two examples, but through the documentation, you will find a lot more of the non-REST connectivity features:

- **RabbitMQ Consumer and Producer**: Velonetics can both consume from and produce to RabbitMQ queues, supporting bidirectional messaging. When configured as a [RabbitMQ Consumer](/docs/backends/amqp-consumer/), Velonetics retrieves messages directly from RabbitMQ queues and processes them according to your routing configuration. As a [RabbitMQ Producer](/docs/backends/amqp-producer/), Velonetics sends messages to specified RabbitMQ queues, enabling it to trigger downstream processes or distribute messages asynchronously across systems.

- **Apache Kafka and other Publisher/Subscriber Patterns**: Aside from RabbitMQ, Velonetics supports [Pub/Sub communication with Apache Kafka](/docs/backends/pubsub/), AWS SNS, SQS, Azure Service Bus, GCP PubSub, or NATS.io. This pattern allows Velonetics to broadcast events to multiple systems without tightly coupling them, making it suitable for applications that require distributed, event-driven workflows.

- **GraphQL Support**: In addition to REST, Velonetics offers support for [GraphQL](/docs/backends/graphql/), allowing API clients to request precisely the data they need. With GraphQL, Velonetics is an efficient aggregator for complex data queries, providing clients with a flexible, schema-driven approach that can reduce over-fetching and under-fetching data.

- **Lambda Function Integration**: Velonetics can [invoke AWS Lambda functions directly](/docs/backends/lambda/), allowing it to act as a bridge between event sources and serverless compute resources. This integration enables applications to process data or execute business logic asynchronously without requiring dedicated infrastructure, a common pattern in serverless and event-driven architectures. With significant volumes, using Velonetics instead of the native AWS API Gateway can remarkably reduce your bill.

- **Event-Driven Async Agents**: Velonetics’s [async agents](/docs/async/) facilitate non-blocking, event-driven processing within the gateway. These agents allow Velonetics to handle events autonomously and asynchronously, making it ideal for workflows that require decoupled, reactive responses to changes in data or state. Async agents improve throughput in distributed systems by offloading tasks without waiting for synchronous HTTP responses.

Velonetics Community Edition also ships the following connectivity options:

- **gRPC Client and Server**: Velonetics supports both gRPC client and server functionalities, enabling bidirectional gRPC communication. As a [gRPC client](/docs/backends/grpc/), Velonetics can make requests to external gRPC services. When configured as a [gRPC server](/docs/grpc/server/), Velonetics can receive and handle incoming gRPC requests on the same port as HTTP. You can transparently convert REST to gRPC and vice versa if needed. See the [gRPC overview](/docs/grpc/) for catalog setup.

- **WebSockets**: Velonetics supports [WebSocket communication](/docs/websockets/), facilitating real-time, bidirectional data exchange between clients and servers. Use direct WebSockets or multiplexing, which optimizes infrastructure when many clients share one backend. Suitable for chat applications, real-time notifications, and IoT device communication.

- **SOAP**: Velonetics can interface with [SOAP-based services](/docs/backends/soap/), enabling the gateway to connect with legacy systems that rely on the XML-based SOAP protocol. Expose legacy services as JSON REST automatically, and even convert methods to GET.

- **HTTP Streaming & SSE**: Long-lived HTTP streams and [Server-Sent Events](/docs/endpoints/streaming/) use the `no-op` proxy encoding — no separate module required. Ideal for LLM token streams, live logs, and push notifications over plain HTTP.

- **Kafka Advanced Pub/Sub**: When the basic `kafka://` driver is not enough, use [Kafka advanced](/docs/backends/pubsub/kafka/) for mTLS, SASL, Azure Event Hub, and idempotent producers.

- **Async Kafka Driver**: Background [Kafka consumers](/docs/async/kafka/) forward topic events to internal webhooks without an HTTP client trigger.

For faster setup, use the [Velonetics Configurator](/docs/configuration/configurator/) to generate `velonetics.json` from YAML profiles and presets (REST, gRPC, WebSockets, Kafka, SSE, and more).

These non-REST connectivity features in Velonetics allow applications to manage complex data flows and event-based interactions beyond the limitations of HTTP, making Velonetics a versatile option for integrating diverse services within modern and legacy architectures.

Follow to the documents under this section to find out more about each one.

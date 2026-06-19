---
date: 2023-06-08
lastmod: 2026-06-18
linktitle: Version upgrades
title: Upgrading Pucora
description: Guidelines for upgrading Pucora Community Edition and validating configuration across versions.
weight: 20
menu:
  community_current:
    parent: "999 Frequently Asked Questions"
---

Pucora Community Edition is the supported distribution documented in this site. When upgrading between Pucora releases, follow these guidelines.

## Before you upgrade

1. Run `pucora check -c pucora.json` on your current configuration.
2. Review the release notes for breaking changes in `extra_config` namespaces or schema versions.
3. Back up your configuration and any custom plugins.

## Configuration compatibility

Pucora generally loads older configuration files on newer releases. Unknown `extra_config` keys are ignored rather than crashing the gateway, but features behind those keys will not run.

Remove or replace configuration blocks that reference features you no longer use, so audits and reviews stay clear.

## Features included in Pucora CE

The following connectivity features are part of Pucora CE and documented here:

- [WebSockets](/docs/websockets/)
- [gRPC](/docs/grpc/) (client and server)
- [SOAP](/docs/backends/soap/)
- [HTTP streaming & SSE](/docs/endpoints/streaming/)
- [Kafka advanced Pub/Sub](/docs/backends/pubsub/kafka/)
- [Async Kafka agents](/docs/async/kafka/)
- [Pub/Sub](/docs/backends/pubsub/), [GraphQL](/docs/backends/graphql/), [Lambda](/docs/backends/lambda/), [Async agents](/docs/async/)

## After upgrading

```bash
pucora check -c pucora.json
pucora audit -c pucora.json
pucora run -c pucora.json
```

Use the [Pucora Configurator](/docs/configuration/configurator/) to regenerate configs from YAML profiles when migrating large setups.

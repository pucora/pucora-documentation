---
lastmod: 2026-06-18
date: 2018-06-23
aliases: ["velonetics-vs-lura"]
linktitle: Lura vs. Pucora
description: The Lura Project is the API gateway engine; Pucora CE is the ready-to-use distribution built on top of it.
notoc: true
menu:
  community_current:
    parent: "310 Design principles"
title: Lura vs. Pucora
weight: 100
---
If you look at the git repositories or this documentation, you will see two related names: the **Lura Project** and **Pucora**.

## TL;DR

- [**Lura**](https://luraproject.org) is the API gateway **engine** — a toolkit of Go libraries for building gateways. It is a [Linux Foundation](https://linuxfoundation.org/) project.
- **Pucora CE** ([`velonetics-ce`](https://github.com/pucora/velonetics-ce)) is a ready-to-use API gateway distribution built on Lura, with logging, metrics, circuit breaking, rate limiting, JWT, pub/sub, WebSockets, gRPC, SOAP, and more.

### Lura Project

The [Lura Project](https://luraproject.org) offers an extendable, simple, stateless high-performance API gateway **framework** for cloud-native and on-prem setups. You can use it as a full framework or import individual Go packages.

Lura focuses on core gateway functionality and keeps the surface clean and extensible. Pucora CE is the primary distribution built on Lura in this project.

{{< button-group >}}
{{< button url="https://luraproject.org" text="Website" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9m-9 9a9 9 0 019-9" />
</svg>{{< /button >}}
{{< button url="https://github.com/luraproject/lura" type="inversed" >}}Source code{{< /button >}}
{{< /button-group >}}

### Pucora API Gateway

[`velonetics-ce`](https://github.com/pucora/velonetics-ce) is the Community Edition gateway: a single binary with the middleware and backends the project maintains, including non-REST connectivity documented in this site.

{{< button-group >}}
{{< button url="https://github.com/pucora/velonetics-ce" type="inversed" >}}Source code{{< /button >}}
{{< /button-group >}}

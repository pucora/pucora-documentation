---
lastmod: 2022-01-18
date: 2022-01-18
linktitle: HTTP Global Client settings
title: HTTP Global Client settings
weight: 140
notoc: true
aliases: ["/docs/throttling/max-idle-connections/"]
menu:
  community_current:
    parent: "040 Routing and Forwarding"
---
When Pucora communicates using HTTP **with all your upstream services**, it implements a concurrent-safe round tripper that supports HTTP, HTTPS, and HTTP proxies, and it caches connections for future re-use. This may leave many open connections when accessing many hosts. You can change the behavior of the transport layer using several settings presented below.

The following settings affect **all connections from Pucora to your services**.

If you want to customize any of the settings below, they must be written at the top level of the configuration.

{{< schema data="pucora.json" filter="dialer_timeout,dialer_keep_alive,dialer_fallback_delay,disable_compression,disable_keep_alives,max_idle_connections,max_idle_connections_per_host,idle_connection_timeout,response_header_timeout,expect_continue_timeout,client_tls" title="Settings for Pucora to your service communication" >}}

Finally, the **TLS Handshake Timeout** is hardcoded to 10 seconds and cannot be changed.

## Override settings using environment vars
When you declare in the configuration file any of the HTTP server or transport settings declared above, you can [override its value through environment variables](/docs/configuration/environment-vars/) when starting the server.

All the environment variables have the same name as the settings above in uppercase and with the `VELONETICS_` prefix. The following env vars are available:

- `VELONETICS_DIALER_TIMEOUT`
- `VELONETICS_DIALER_KEEP_ALIVE`
- `VELONETICS_DIALER_FALLBACK_DELAY`
- `VELONETICS_DISABLE_COMPRESSION`
- `VELONETICS_DISABLE_KEEP_ALIVES`
- `VELONETICS_MAX_IDLE_CONNECTIONS`
- `VELONETICS_MAX_IDLE_CONNECTIONS_PER_HOST`
- `VELONETICS_IDLE_CONNECTION_TIMEOUT`
- `VELONETICS_RESPONSE_HEADER_TIMEOUT`
- `VELONETICS_EXPECT_CONTINUE_TIMEOUT`

You can start Pucora with the desired variables to override what you have in the configuration:

{{< terminal title="Term" >}}
VELONETICS_MAX_IDLE_CONNECTIONS_PER_HOST=200 pucora run -c pucora.json
{{< /terminal >}}

## Max IDLE connections
Having a high number of IDLE connections to every backend affects directly to the performance of the proxy layer. This is why you can control the number using the `max_idle_connections` setting. For instance:

```json
{
	"version": 3,
	"max_idle_connections": 150
}
```

Pucora will close connections sitting idle in a "keep-alive" state when `max_idle_connections` is reached. If no value is set in the configuration file, Pucora will use `250` by default.

Every ecosystem needs its own setting, have this in mind:

- If you set a number very high for `max_idle_connections` you might exhaust your system's port limit.
- If you set a number very low, new connections will be frequently created and a low rate of connection reuse will take place.

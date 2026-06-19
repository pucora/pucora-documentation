---
lastmod: 2024-10-24
date: 2022-01-18
linktitle: HTTP Server settings
title: HTTP Server Settings
description: Configure HTTP server settings in Pucora API Gateway for optimal performance, security, and compatibility with your infrastructure
weight: 50
notoc: true
menu:
  community_current:
    parent: "030 Service Settings"
meta:
  source: https://github.com/luraproject/lura
  scope:
  - service
  log_prefix:
  - "[SERVICE: HTTP Server]"
---
Pucora starts an HTTP server to offer the API Gateway server. You can personalize some of the settings used to start the service and also override the default settings of the underlying Go [standard library](https://pkg.go.dev/net/http#Server).

If you want to customize any of the settings below, they must be written at the top level of the configuration.

{{< schema data="_root.json" filter="max_shutdown_wait_time,port,cache_ttl,sequential_start,read_timeout,read_header_timeout,write_timeout,idle_timeout,use_h2c,listen_ip,max_header_bytes,disable_rest">}}

## Override settings using environment vars
When you declare in the configuration file any of the HTTP server settings declared above, you can [override its value through environment variables](/docs/configuration/environment-vars/) when starting the server.

All the environment variables have the same name are the same settings above in uppercase and with the `VELONETICS_` prefix. For instance, looking at the list of settings above, you could override:

- `VELONETICS_PORT`
- `VELONETICS_READ_TIMEOUT`
- `VELONETICS_READ_HEADER_TIMEOUT`
- `VELONETICS_WRITE_TIMEOUT`
- `VELONETICS_IDLE_TIMEOUT`
- `VELONETICS_USE_H2C`
- etc...

You can start Pucora with the desired variables to override what you have in the configuration:

{{< terminal title="Term" >}}
VELONETICS_PORT=8000 VELONETICS_READ_TIMEOUT="1s" pucora run -c pucora.json
{{< /terminal >}}

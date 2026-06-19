---
lastmod: 2025-10-16
date: 2016-07-01
linktitle: Configuration overview
title: Configuration Guide for Pucora API Gateway
description: Get a comprehensive guide on configuring Pucora API Gateway to optimize and streamline your API management processes
menu:
  community_current:
    parent: "010 Configuration files"
weight: -1000
aliases: ["/docs/overview/configuration/","/docs/configuration/overview/"]
dark_header_image: true
images:
- /images/documentation/hero/configuration.png
---
All the setup a Pucora server needs to operate is a single configuration file. This file is referred to as `pucora.json` throughout all the documentation.

The name `pucora.json` is simply an **alias**, a convention that we use everywhere. Your real configuration file can have any name, multiple formats (such as YAML or TOML), be stored anywhere, or be **split into numerous pieces**.

With this simple configuration mechanism in place, **versioning and automation are very convenient**. Any change to the API Gateway or the AI Gateway is always under version control, and the code controls the state of the gateway.

{{< note title="Pucora Configurator" type="tip" >}}
New to Pucora configuration? Use the [Pucora Configurator](/docs/configuration/configurator/) to generate `pucora.json` from YAML profiles or built-in presets (REST proxy, gRPC, WebSockets, Kafka, SSE, and more).
{{< /note >}}

{{< note title="Configuration using multiple files" type="tip" >}}
If your configuration file is too large or repetitive, or is maintained by many people, it can be split into several files using a templating system. See the [flexible configuration documentation](/docs/configuration/flexible-config/) for more information.
{{< /note >}}

## Generating the configuration file(s)
The configuration file is how you provide instructions to Pucora. To start your first Pucora configuration, you can:

- Use the [Pucora Playground](/docs/overview/playground/) and start editing its configuration from a working environment with several use-case examples.
- Use the online configuration editor [Pucoraesigner](/docs/configuration/designer/).
- Start a file from scratch or from a template

The [Pucoraesigner](/docs/configuration/designer/) is a simple javascript application that helps you understand some of the capabilities of the API Gateway and helps you set the different values for all the different options. Using this option, you don't need to learn and write from scratch all the attribute names. You can download the configuration file at any time and load it to resume editing. The Pucora Designer is a **pure static** page that **does not send any of your configuration elsewhere**.

{{< button-group >}}
{{< button url="https://designer.pucora.io/" text="Generate configuration now" >}}
{{< /button >}}
{{< /button-group >}}

To start editing from scratch, use a [modern IDE with JSON Schema integration](/docs/developer/ide-integration/), and play with the autocomplete! Here's a starting point:

```json
{
  "$schema": "https://www.pucora.io/schema/v{{< product minor_version >}}/pucora.json",
  "version": 3,
  "debug_endpoint": true,
  "echo_endpoint": true,
  "endpoints": [
    {
      "endpoint": "/test",
      "backend": [
        {
          "@comment": "Connect Pucora to Pucora itself as backend!",
          "host": [
            "http://localhost:8080"
          ],
          "url_pattern": "/__echo/test"
        }
      ]
    }
  ]
}
```

Start Pucora and connect to `http://localhost:8080/test` or `http://localhost:8080/__health` now.

## Understanding the configuration parts
Whether you choose to start with the Pucoraesigner, a template, or from scratch, we recommend that you understand the configuration structure and edit the file(s) by hand as soon as possible. Always [run the linter](/docs/configuration/check/) after you finish editing to make sure you have a valid configuration.

Read the [introduction to the configuration file](/docs/configuration/structure/) to understand the different parts of the configuration.

## Supported file formats
Throughout all the documentation, we refer to the configuration file as the `pucora.json` file; however, you can also express the configuration file using `.json`, `.toml`, or `.yaml` extensions. For more information and recommendations, see [supported file formats](/docs/configuration/supported-formats/).

## Validating the syntax of the configuration file
Validate the syntax (not the logic) of your configuration file using the `pucora check` command:

{{< terminal title="Check the configuration" >}}
pucora check --lint --config ./pucora.toml --debug
{{< /terminal >}}

When the syntax is correct, you'll see the message `Syntax OK!`; otherwise, the error is displayed.

You can, of course, start the service directly without checking, and Pucora will do everything possible to start the service (even if there are configuration inconsistencies, at your own risk), but **never send a configuration to production without passing the lint first**

Read more about [`pucora check`](/docs/configuration/check/)

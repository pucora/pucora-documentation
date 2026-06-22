---
aliases: ["/docs/commands/run/", "/docs/overview/usage/"]
lastmod: 2016-10-28
date: 2016-10-28
linktitle: Running Pucora
title: Running Pucora server
description: All options of the pucora command to manage the service, write plugins, or get information about the installed version.
weight: 20
notoc: true
menu:
  community_current:
    parent: "000 Getting Started"
---

After installing Pucora, you can start using Pucora by typing `pucora`. To see all the options of `pucora`, type `pucora -h` or `pucora <COMMAND> -h`. For instance, the `pucora run` help is:

{{< terminal title="Run command help" >}}
pucora run -h

{{< ascii-logo >}}

Version: {{< product latest_version >}}

The API Gateway builder

Usage:
  pucora [command]

Available Commands:
  audit         Checks the integrity of the config and returns security recommendations.
  check         Validates that the configuration file is valid.
  check-plugin  Check the compatibility with the plugin deps.
  help          Help about any command
  run           Run the Pucora server.
  version       Shows Pucora version.

Flags:
  -c, --config string   Path to the configuration filename
  -d, --debug           Enable the debug
  -h, --help            help for pucora

Use "pucora [command] --help" for more information about a command.
{{< /terminal >}}

You can use the following commands:

- `pucora audit`: Use [pucora audit](/docs/configuration/audit/) to get security recommendations for a given configuration.
- `pucora check`: Use [pucora check](/docs/configuration/structure/) to make sure the configuration file you have generated is not broken and has the required attributes to start the gateway.
- `pucora check-plugin`: Use the [check-plugin](/docs/extending/check-plugin/) when you are developing custom plugins and you want to check that they are compatible with the server.
- `pucora run`: Use run to start the API gateway server.
- `pucora version`: Use the version command to print the current Pucora version and the Glibc and Go versions used during compilation.

## Starting the gateway server
To start the server, invoke the `pucora run` command with a configuration file containing your API definition. You can visually create your first `pucora.json` file using the [Pucoraesigner](https://designer.pucora.in/) if you prefer a UI.

Or to get started right away, you can paste the following content inside a `pucora.json` file:

```json
{
    "$schema": "https://www.pucora.in/schema/v{{< product minor_version >}}/pucora.json",
    "version": 3
}
```

And then you can start Pucora:

{{< terminal title="Command to start Pucora" >}}
pucora run -c pucora.json
{{< /terminal >}}

Or if you use Docker:

{{< terminal title="Command to start Pucora with Docker" >}}
docker run -p "8080:8080" -v $PWD:/etc/pucora/ {{< product image >}}:{{< product latest_version >}} run -c pucora.json
{{< /terminal >}}

Now Pucora is listening on `8080`, and you can see it working under `http://localhost:8080/__health`.

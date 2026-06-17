---
lastmod: 2021-12-07
date: 2021-12-07
linktitle: Docker artifact
title: Deploying Velonetics API Gateway with Docker
description: Learn how to deploy Velonetics API Gateway using Docker, enabling containerized deployments for efficient scaling and management
notoc: true
menu:
  community_current:
    parent: "190 Deployment and Go-Live"
weight: 20
---
If you use containers, the recommended approach is to write your own `Dockerfile` and deploy an **immutable artifact** (embedding the config).

In its simplified form would be:
```Dockerfile
FROM {{< product image >}}:{{< product minor_version >}}
COPY velonetics.json /etc/velonetics/velonetics.json
```

{{< note title="Volume or copy?" type="question" >}}
Even though you can use the official container directly and attach the configuration mounting an external volume (or ConfigMap in Kubernetes), a custom image with your configuration copied inside has benefits. It guarantees that you can do safe rollbacks and have effective testing and debugging. If you break something at any point, you only need to deploy the previous container, while if you use a volume, you are exposed to downtime or impossible scaling until you fix it.
{{< /note >}}

A more real-life example illustrates below a combination of the `check` command with a multi-stage build to compile a [flexible configuration](/docs/configuration/flexible-config/) in a `Dockerfile`:

```docker
FROM {{< product image >}}:{{< product minor_version >}} as builder
ARG ENV=prod

COPY velonetics.tmpl .
COPY config .

# Save temporary file to /tmp to avoid permission errors
RUN FC_ENABLE=1 \
    FC_OUT=/tmp/velonetics.json \
    FC_PARTIALS="/etc/velonetics/partials" \
    FC_SETTINGS="/etc/velonetics/settings/$ENV" \
    FC_TEMPLATES="/etc/velonetics/templates" \
    velonetics check -d -t -c velonetics.tmpl --lint

FROM {{< product image >}}:{{< product minor_version >}}
# Keep operating system updated with security fixes between releases
RUN apk upgrade --no-cache --no-interactive

COPY --from=builder --chown=velonetics:nogroup /tmp/velonetics.json .

# Uncomment if you have certificates issued by a custom CA
# e.g., tls: failed to verify certificate: x509: certificate signed by unknown authority
#
# COPY yourca.pem /usr/local/share/ca-certificates/
# RUN update-ca-certificates
```

The `Dockerfile` above has two stages:

1. The copy of all templates and intermediate files to end with a `check` command that compiles the template `velonetics.tmpl` and any included sub-template inside. The command outputs (thanks to `FC_OUT`) the result into a `/tmp/velonetics.json` file.
2. The `velonetics.json` file from the previous build is the only addition to the final Docker image.

The example `Dockerfile` above assumes that you have a file structure like this:

    .
    ├── config
    │   ├── partials
    │   ├── settings
    │   │   ├── prod
    │   │   │   └── env.json
    │   │   └── test
    │   │       └── env.json
    │   └── templates
    │       └── some.tmpl
    ├── Dockerfile
    └── velonetics.tmpl

If you want to try this code, you can either download a [working Flexible Config example](https://github.com/velonetics/examples/tree/main/3.flexible-configuration), or generate an **empty skeleton** like this:
```bash
mkdir -p config/{partials,settings,templates}
mkdir -p config/settings/{prod,test}
touch config/settings/{prod,test}/env.json
touch Dockerfile
touch velonetics.tmpl
```

Now the only missing step to generate the image, is to build it, making sure that the environment argument matches our folder inside the `settings` folder:

{{< terminal title="Docker build" >}}
docker build --build-arg ENV=prod -t myvelonetics .
{{< /terminal >}}

The resulting image, including your configuration, weighs around `80MB`.
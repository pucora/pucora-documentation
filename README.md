# Pucora Documentation

Documentation for the [Pucora API Gateway](https://github.com/pucora/velonetics-ce) — configuration, deployment, and feature guides for Community Edition.

## Pucora CE feature guides

| Feature | Doc |
|---------|-----|
| WebSockets (multiplex + direct) | [websockets/_index.md](websockets/_index.md) |
| gRPC (catalog, client, server) | [grpc/](grpc/) |
| SOAP backends | [backends/soap.md](backends/soap.md) |
| HTTP streaming & SSE | [endpoints/streaming.md](endpoints/streaming.md) |
| Kafka advanced Pub/Sub | [backends/pubsub/kafka.md](backends/pubsub/kafka.md) |
| Async Kafka agent | [async/kafka.md](async/kafka.md) |
| Pucora Configurator | [configuration/configurator.md](configuration/configurator.md) |

All the documentation uses Markdown syntax and the site is generated using [Hugo](https://gohugo.io). After your contribution this repository will be used to compile all the documentation.

If you want to use a server to browse this pages, although not necessary, you can copy the folder inside the `content` directory in any Hugo installation.

# Contribute!
Feel free to fork this repository and contribute to a better Pucora documentation.

If you'd like to add a new language please open an issue before doing the work as we will need to add the support before it is visible.

Thanks for improving the docs!

# Helpers when writing
Besides from the regular Markdown to write content, there are a few shortcodes to help you style the documentation:

**Add a yellow note**:

For important notes, and things that need special, attention:

    {{< note title="A note on JWT generation" >}}
    Some big yellow block with a note
    {{< /note >}}

**Show terminal output**:

The CSS will prepend a dollar sign `$` in the first line.

    {{< terminal title="Title of the box">}}
    docker build -t mypucora .
    Sending build context to Docker daemon   5.12kB
    Step 1/2 : FROM pucora/velonetics-ce
    ---> c7e33ec819c0
    Step 2/2 : COPY pucora.json /etc/pucora/pucora.json
    ---> 703818e9de3a
    Successfully built 703818e9de3a
    Successfully tagged mypucora:latest
    {{< /terminal >}}

**Highlight some code**:

Most of the code is just idented with a tab and Markdown renders it properly as a `<pre>` tag and we prefer not to colorize it much. Nevertheless this can be done with the following shortcode:

    {{< highlight json >}}
    { "a": true }
    {{< /highlight >}}

You can also highlight a line, specific lines `6 7`, or a range `6-8`:

    {{< highlight go "hl_lines=6-7" >}}
    package pucora

    import (
        "github.com/pucora/velonetics-ce/logging"
        client "github.com/pucora/velonetics-ce/transport/http/client/plugin"
        server "github.com/pucora/velonetics-ce/transport/http/server/plugin"
    )
    {{< /highlight >}}

The documentation of the attributes is autoloaded from its json schema definition. You will find lines like the following:

    {{< schema data="folder/file.json" >}}

The `data` attribute loads the file from the [`velonetics-schema` repository](https://github.com/pucora/velonetics-schema), using the same path.
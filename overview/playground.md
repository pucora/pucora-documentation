---
lastmod: 2018-09-27
date: 2018-09-13
linktitle: Playground
title: Pucora API Gateway Playground
description: Explore Pucora Playground and experiment with different API configurations before deploying them
notoc: true
source: https://github.com/pucora/playground-community
menu:
  community_current:
    parent: "000 Getting Started"
weight: 50
images: ["/images/documentation/velonetics-playground-diagram.png"]

---
If you are new to Pucora, a quick way to get started is to make use of the [Pucora Playground](https://github.com/pucora/playground-community).

The Pucora Playground is a **Docker Compose** environment that puts together the necessary pieces to let you play with Pucora in a working environment.

As Pucora is an API gateway, we have also added to the environment an API (backend) to feed the gateway and a website to make use of the data. With the Pucora Playground, you can see the different pieces that take place in the API Gateway use cases.

## An environment ready to use!
The Pucora Playground can be started with `docker compose up` and then fire up your browser, `curl`, Postman, httpie or anything else you use to interact with the services.

When starting the Docker compose you will have:

- The Pucora API Gateway, on port 8080
- A client consuming data from the gateway, on port 3000
- A backend simulating API responses on port 8000
- Plus a [Jaeger](https://www.jaegertracing.io/) where traces are sent on port 16686

Have a look at the `docker-compose.yml` file to understand how these services interact together, and also at the `pucora.json` file to find out what endpoints are exposed.

If you'd like to see more examples, please let us know or open an [issue](https://github.com/pucora/playground-community/issues)!

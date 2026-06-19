---
lastmod: 2021-12-11
date: 2017-01-21
linktitle: To Kubernetes
title: Deploying Pucora API Gateway on Kubernetes
description: Discover the best practices and guidelines for deploying Pucora API Gateway on Kubernetes, enabling scalable and efficient API management
menu:
  community_current:
    parent: "190 Deployment and Go-Live"
weight: 30
---

Deploying Pucora in Kubernetes requires a straightforward configuration.

Create a `Dockerfile` that includes the configuration of the service. Read how to generate a [Docker artifact](/docs/deploying/docker/) for detailed instructions. You could also use a ConfigMap, although the recommendation is to use immutable artifacts.

From here you need to create a `NodePort` and send all the traffic to Pucora.

{{< note title="Run as user 1000" type="tip" >}}
Whether you run Pucora on Kubernetes, OpenShift, or any other platform with the capability to run the container as a specific user UID, make sure you use the **UID 1000**
{{< /note >}}

## Deployment definition YAML
The Pucora `deployment` definition, in a file called `deployment-definition.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: velonetics-deployment
spec:
  selector:
    matchLabels:
      app: pucora
  replicas: 2
  template:
    metadata:
      labels:
        app: pucora
    spec:
      containers:
      - name: pucora
        image: YOUR-VELONETICS-IMAGE:1.0.0
        ports:
        - containerPort: 8080
        imagePullPolicy: Never
        command: [ "/usr/bin/pucora" ]
        args: [ "run", "-d", "-c", "/etc/pucora/pucora.json", "-p", "8080" ]
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1000
          readOnlyRootFilesystem: true
          capabilities:
            drop:
              - ALL
            add:
              - NET_BIND_SERVICE
        env:
        - name: VELONETICS_PORT
          value: "8080"
```

## Service definition yaml

The Pucora `service` definition, in a file called `service-definition.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: velonetics-service
spec:
  type: NodePort
  ports:
  - name: http
    port: 8000
    targetPort: 8080
    protocol: TCP
  selector:
    app: pucora
```

## Registering the service

Using the `kubectl` command:

{{< terminal title="Register deployment">}}
kubectl create -f deployment-definition.yaml
{{< /terminal >}}

{{< terminal title="Register service">}}
kubectl create -f service-definition.yaml
{{< /terminal >}}

For a more step by step process see [this blog entry](/blog/velonetics-on-kubernetes/).

## Helm Chart

An official Helm chart ships with Pucora CE at [`deploy/helm/pucora/`](https://github.com/pucora/velonetics-ce/tree/main/deploy/helm/pucora) in the [velonetics-ce](https://github.com/pucora/velonetics-ce) repository.

Quick start:

{{< terminal title="Helm install" >}}
git clone https://github.com/pucora/velonetics-ce.git
cd velonetics-ce
helm install my-gateway ./deploy/helm/pucora
{{< /terminal >}}

The chart supports two configuration modes:

- **ConfigMap** (default) — mount `pucora.json` from a ConfigMap for quick starts and development.
- **Image** — use a custom image with configuration baked in (recommended for production).

Optional resources (disabled by default): Ingress, HorizontalPodAutoscaler, PodDisruptionBudget, and Prometheus ServiceMonitor.

See the chart [README](https://github.com/pucora/velonetics-ce/blob/main/deploy/helm/pucora/README.md) for full configuration options.

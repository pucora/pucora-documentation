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
  name: pucora-deployment
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
        image: YOUR-PUCORA-IMAGE:1.0.0
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
        - name: PUCORA_PORT
          value: "8080"
```

## Service definition yaml

The Pucora `service` definition, in a file called `service-definition.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: pucora-service
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

For a more step by step process see [this blog entry](/blog/pucora-on-kubernetes/).

## Helm Chart

The recommended way to deploy on Kubernetes is the **official Helm chart** shipped with Pucora CE.

It includes Deployment, Service, health probes, optional Ingress, HPA, PDB, Prometheus integration, and three configuration modes (ConfigMap, Secret, or immutable image).

{{< button-group >}}
{{< button url="/docs/deploying/helm/" text="Helm chart guide" >}}
{{< /button-group >}}

Quick install:

{{< terminal title="Helm install" >}}
git clone https://github.com/pucora/pucora-ce.git
cd pucora-ce
helm install my-gateway ./deploy/helm/pucora
kubectl port-forward svc/my-gateway-pucora 8080:8080
curl http://localhost:8080/__health
helm test my-gateway
{{< /terminal >}}

For production values, OCI installs, upgrades, and GitOps patterns, see the [Deploy Pucora with the Helm Chart](/docs/deploying/helm/) guide.

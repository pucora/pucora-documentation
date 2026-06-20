---
lastmod: 2026-06-18
date: 2026-06-18
linktitle: Helm Chart
title: Deploy Pucora with the Helm Chart
description: Install, configure, upgrade, and test the official Pucora Helm chart on Kubernetes — ConfigMap, Secret, and immutable image modes.
aliases: ["/docs/deploying/helm/"]
menu:
  community_current:
    parent: "190 Deployment and Go-Live"
weight: 35
---

The official **Pucora Helm chart** packages everything you need to run the gateway on Kubernetes: Deployment, Service, probes, optional Ingress, HPA, monitoring hooks, and configuration mounting.

**Chart location:** [`deploy/helm/pucora/`](https://github.com/pucora/pucora-ce/tree/main/deploy/helm/pucora) in the [pucora-ce](https://github.com/pucora/pucora-ce) repository.

## Prerequisites

- Kubernetes **1.23+**
- **Helm 3.8+**
- (Optional) Prometheus Operator for `ServiceMonitor` / `PodMonitor`
- (Optional) cert-manager for automatic TLS
- (Optional) Gateway API controller for `HTTPRoute`
- (Optional) KEDA for event-driven autoscaling

## Quick start

Clone the repository and install the chart with default values (ConfigMap mode, 2 replicas, built-in sample config):

{{< terminal title="Install from the repository" >}}
git clone https://github.com/pucora/pucora-ce.git
cd pucora-ce
helm install my-gateway ./deploy/helm/pucora
{{< /terminal >}}

Verify the release:

{{< terminal title="Port-forward and health check" >}}
kubectl port-forward svc/my-gateway-pucora 8080:8080
curl http://localhost:8080/__health
helm test my-gateway
{{< /terminal >}}

The chart runs Pucora as **UID 1000** with a read-only root filesystem — the same security baseline described in the [Kubernetes guide](/docs/deploying/kubernetes/).

## Install from OCI registry

On each release, the chart is published to GitHub Container Registry:

{{< terminal title="Install from OCI" >}}
helm install my-gateway oci://ghcr.io/pucora/charts/pucora --version 2.2.0
{{< /terminal >}}

Replace `2.2.0` with the [latest release](https://github.com/pucora/pucora-ce/releases) tag.

## Configuration modes

The chart supports three ways to supply `pucora.json`:

| Mode | `config.mode` | Best for |
|------|---------------|----------|
| ConfigMap | `configmap` (default) | Development, quick starts |
| Secret | `secret` | Configs with embedded credentials |
| Immutable image | `image` | **Production** — config baked into the container |

### ConfigMap with your own file

{{< terminal title="Install with a custom pucora.json" >}}
helm install my-gateway ./deploy/helm/pucora \
  --set-file config.pucoraJson=./pucora.json
{{< /terminal >}}

### Secret mode

{{< terminal title="Mount config from a Secret" >}}
helm install my-gateway ./deploy/helm/pucora \
  --set config.mode=secret \
  --set-file config.pucoraJson=./pucora.json
{{< /terminal >}}

### Immutable image (production)

Build a Docker image that copies your config, then point the chart at it:

```dockerfile
FROM niteesh20/pucora:2.2.0
COPY pucora.json /etc/pucora/pucora.json
```

{{< terminal title="Install with config.mode=image" >}}
helm install my-gateway ./deploy/helm/pucora \
  --set config.mode=image \
  --set image.repository=myregistry/pucora-gateway \
  --set image.tag=v1.0.0
{{< /terminal >}}

{{< note title="Validate before deploy" type="tip" >}}
Run `pucora check --lint` in CI before upgrading the release:

{{< terminal title="Validate configuration" >}}
docker run --rm -v $(pwd)/pucora.json:/etc/pucora/pucora.json \
  niteesh20/pucora:2.2.0 check --lint -c /etc/pucora/pucora.json
{{< /terminal >}}
{{< /note >}}

## Values files and production profiles

Use a dedicated values file instead of long `--set` chains:

{{< terminal title="Install with a values file" >}}
helm install my-gateway ./deploy/helm/pucora -f my-values.yaml
{{< /terminal >}}

Example production overrides:

```yaml
replicaCount: 2

config:
  mode: image

usageReporting:
  disable: true

resources:
  requests:
    cpu: 100m
    memory: 64Mi
  limits:
    cpu: 500m
    memory: 256Mi

probes:
  startup:
    enabled: true

lifecycle:
  preStop:
    enabled: true
    sleepSeconds: 10

podAntiAffinity:
  enabled: true
  type: soft

podDisruptionBudget:
  enabled: true
  minAvailable: 1

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

serviceMonitor:
  enabled: true
```

The repository ships ready-made examples:

| File | Use case |
|------|----------|
| [`ci/values-prod.yaml`](https://github.com/pucora/pucora-ce/blob/main/deploy/helm/pucora/ci/values-prod.yaml) | Production: image config, HPA, PDB, monitoring |
| [`ci/values-aws-nlb.yaml`](https://github.com/pucora/pucora-ce/blob/main/deploy/helm/pucora/ci/values-aws-nlb.yaml) | AWS Network Load Balancer |
| [`ci/values-istio.yaml`](https://github.com/pucora/pucora-ce/blob/main/deploy/helm/pucora/ci/values-istio.yaml) | Istio sidecar with startup probe |

{{< terminal title="Production install example" >}}
helm install my-gateway ./deploy/helm/pucora \
  -f deploy/helm/pucora/ci/values-prod.yaml
{{< /terminal >}}

## Common configuration tasks

### Expose with Ingress and TLS

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix

certificate:
  enabled: true
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

Requires [cert-manager](https://cert-manager.io/) and an Ingress controller.

### AWS Network Load Balancer

```yaml
service:
  nlb:
    enabled: true
    externalTrafficPolicy: Local
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
      service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
```

### Istio sidecar injection

```yaml
sidecarInjection:
  enabled: true
  istio:
    enabled: true
    holdApplicationUntilProxyStarts: true

probes:
  startup:
    enabled: true
```

Enable the **startup probe** when a mesh sidecar starts before the gateway is ready.

### OpenTelemetry and Prometheus

```yaml
opentelemetry:
  enabled: true
  endpoint: http://otel-collector:4317

serviceMonitor:
  enabled: true

prometheusRule:
  enabled: true
  rules:
    - alert: PucoraDown
      expr: up{job=~".*pucora.*"} == 0
      for: 5m
      labels:
        severity: critical
```

See [Telemetry](/docs/telemetry/) for exporter configuration inside `pucora.json`.

### GitOps (Argo CD)

```yaml
gitops:
  argocd:
    enabled: true
    syncWave: "2"
```

Pair with a [CI/CD pipeline](/docs/deploying/ci-cd/) that runs `pucora check` before sync.

## Upgrade, test, and rollback

{{< terminal title="Lifecycle commands" >}}
helm upgrade my-gateway ./deploy/helm/pucora -f my-values.yaml
helm test my-gateway
helm rollback my-gateway
helm history my-gateway
{{< /terminal >}}

When `tests.configCheck` is enabled (default), `helm test` runs `pucora check` against the mounted configuration.

## Blue/green deployments

Pucora is **stateless**. For zero-downtime config changes with `config.mode=image`:

1. Build a new image with the updated `pucora.json`.
2. Deploy a second Helm release (e.g. `my-gateway-green`).
3. Switch the Service selector or Ingress backend to the new release.
4. Remove the old release.

Alternatively use Argo Rollouts or Flagger with the Istio options above.

## Key values reference

| Parameter | Default | Description |
|-----------|---------|-------------|
| `replicaCount` | `2` | Gateway pod replicas |
| `config.mode` | `configmap` | `configmap`, `secret`, or `image` |
| `image.repository` | `niteesh20/pucora` | Container image |
| `image.tag` | chart `appVersion` | Image tag |
| `usageReporting.disable` | `true` | Sets `USAGE_DISABLE=1` |
| `probes.startup.enabled` | `false` | Startup probe (enable with mesh) |
| `lifecycle.preStop.enabled` | `true` | Graceful drain before termination |
| `autoscaling.enabled` | `false` | Horizontal Pod Autoscaler |
| `keda.enabled` | `false` | KEDA ScaledObject (mutually exclusive with HPA) |
| `ingress.enabled` | `false` | Kubernetes Ingress |
| `serviceMonitor.enabled` | `false` | Prometheus Operator scrape config |
| `tests.configCheck` | `true` | Run `pucora check` on `helm test` |

See [`values.yaml`](https://github.com/pucora/pucora-ce/blob/main/deploy/helm/pucora/values.yaml) and [`values.schema.json`](https://github.com/pucora/pucora-ce/blob/main/deploy/helm/pucora/values.schema.json) for the complete list.

## Uninstall

{{< terminal title="Remove the release" >}}
helm uninstall my-gateway
{{< /terminal >}}

## Related documentation

- [Deploying on Kubernetes](/docs/deploying/kubernetes/) — manual manifests and cluster basics
- [Docker artifacts](/docs/deploying/docker/) — building immutable images
- [CI/CD pipelines](/docs/deploying/ci-cd/) — validate config before deploy
- [Chart README](https://github.com/pucora/pucora-ce/blob/main/deploy/helm/pucora/README.md) — full maintainer reference

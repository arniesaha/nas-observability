# observability

Kubernetes observability stack for k3s clusters. Mobile-friendly Grafana dashboards with Prometheus metrics collection for cluster health and application monitoring.

## Features

- **Prometheus** — Metrics collection with 15-day retention
- **Grafana** — Mobile-friendly dashboards and alerting
- **Node Exporter** — Host-level metrics (CPU, memory, disk, network)
- **Kube State Metrics** — Kubernetes object metrics (pods, deployments, etc.)
- **AlertManager** — Alert routing and notifications

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Grafana                                 │
│                  (Visualization + Alerts)                       │
│                  grafana.arnabsaha.com                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Prometheus                               │
│                   (Metrics Storage)                             │
└─────────────────────────────────────────────────────────────────┘
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ Node Exporter │    │  Kube State   │    │   App Metrics │
│  (host stats) │    │   Metrics     │    │ (ServiceMon)  │
└───────────────┘    └───────────────┘    └───────────────┘
```

## Quick Start

### Prerequisites

- k3s cluster
- Helm 3.x
- Traefik ingress controller (included in k3s)

### Installation

1. Add Prometheus community Helm repo:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

2. Create monitoring namespace and install:

```bash
kubectl create namespace monitoring

helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  -f helm/values.yaml \
  --wait
```

3. Apply Traefik IngressRoute:

```bash
kubectl apply -f helm/grafana-ingressroute.yaml
```

4. Access Grafana:
   - URL: http://grafana.arnabsaha.com
   - Username: `admin`
   - Password: `observability123` (change this!)

## Default Dashboards

The kube-prometheus-stack includes pre-built dashboards:

| Dashboard | Description |
|-----------|-------------|
| Kubernetes / Compute Resources / Cluster | Cluster-wide resource usage |
| Kubernetes / Compute Resources / Namespace (Pods) | Per-namespace breakdown |
| Node Exporter / Nodes | Host-level metrics |
| CoreDNS | DNS query metrics |
| Prometheus | Prometheus self-monitoring |

## Adding Application Metrics

To expose metrics from your applications, create a ServiceMonitor:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: my-app
  namespaceSelector:
    matchNames:
      - apps
  endpoints:
    - port: metrics
      interval: 30s
```

## Key Metrics

### Cluster Health
- `node_cpu_seconds_total` — CPU usage
- `node_memory_MemAvailable_bytes` — Available memory
- `node_filesystem_avail_bytes` — Disk space
- `kube_pod_status_phase` — Pod states

### Network
- `node_network_receive_bytes_total` — Network ingress
- `node_network_transmit_bytes_total` — Network egress

### Application
- `container_cpu_usage_seconds_total` — Container CPU
- `container_memory_usage_bytes` — Container memory

## Storage

- Prometheus data: 10Gi (15-day retention)
- Grafana data: 2Gi (dashboards, settings)
- AlertManager: 1Gi (silences, notifications)

## Resource Usage

Approximate resource requirements:

| Component | Memory | CPU |
|-----------|--------|-----|
| Prometheus | 512Mi-2Gi | 200m-1000m |
| Grafana | 256Mi | 100m |
| AlertManager | 64Mi-256Mi | 50m-200m |
| Node Exporter | 32Mi-64Mi | 50m-100m |
| Kube State Metrics | 64Mi-128Mi | 50m-100m |

## Upgrading

```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  -f helm/values.yaml
```

## Uninstalling

```bash
helm uninstall kube-prometheus-stack --namespace monitoring
kubectl delete namespace monitoring
```

## License

MIT

## Author

[Arnab Saha](https://github.com/arniesaha)

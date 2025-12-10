# Flask Application Helm Charts

This repository contains Helm charts for deploying Flask applications (app-1, app2, app3) to Kubernetes.

## Chart Structure

Each application has its own Helm chart with the following components:

- **Deployment**: Manages the application pods with configurable replicas
- **Service**: ClusterIP service exposing the application
- **Ingress**: NGINX ingress with TLS support
- **ServiceAccount**: Dedicated service account for the application
- **ConfigMap**: Application configuration
- **HorizontalPodAutoscaler**: Optional autoscaling based on CPU/Memory

## Prerequisites

- Kubernetes cluster (1.19+)
- Helm 3.x installed
- NGINX Ingress Controller
- cert-manager (for TLS certificates)

## Installation

### Deploy a single application

```bash
# Install app-1
helm install app-1 ./charts/app-1 -n production --create-namespace

# Install app2
helm install app2 ./charts/app2 -n production

# Install app3
helm install app3 ./charts/app3 -n production
```

### Deploy with custom values

```bash
helm install app-1 ./charts/app-1 \
  --set image.tag=v1.0.0 \
  --set replicaCount=3 \
  --set ingress.hosts[0].host=app-1.yourdomain.com \
  -n production
```

### Upgrade an application
```bash
helm upgrade --install app-1 ./charts/app-1 \
  -n dev \
  --set image.tag=... \
  --set image.repository=.../app-1 \
  -f charts/app-1/values.yaml \
  --wait --timeout 10m

helm upgrade --install app-1 ./charts/app-1 \
  -n prod \
  --set image.tag=... \
  --set image.repository=.../app-1 \
  -f charts/app-1/values.yaml \
  --wait --timeout 10m
```

Deploy with custom values:
```bash
helm install app-1 ./charts/app-1 -f custom-values.yaml -n production
```

## Rollback

```bash
# View release history
helm history app-1 -n production

# Rollback to previous version
helm rollback app-1 -n production

# Rollback to specific revision
helm rollback app-1 2 -n production
```

## Uninstall

```bash
helm uninstall app-1 -n production
helm uninstall app2 -n production
helm uninstall app3 -n production
```

## Health Checks

Each application includes:
- **Liveness probe**: `/health` endpoint (checks if app is running)
- **Readiness probe**: `/ready` endpoint (checks if app can serve traffic)

### Check pod status
```bash
kubectl get pods -n production -l app.kubernetes.io/name=app-1
```

### View logs
```bash
kubectl logs -n production -l app.kubernetes.io/name=app-1 --tail=100
```

### Debug deployment
```bash
helm get values app-1 -n production
helm get manifest app-1 -n production
kubectl describe deployment -n production app-1
```

### Test locally
```bash
# Dry run
helm install app-1 ./charts/app-1 --dry-run --debug

# Template rendering
helm template app-1 ./charts/app-1

# Lint chart
helm lint ./charts/app-1
```
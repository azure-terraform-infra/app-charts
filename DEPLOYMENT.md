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
helm upgrade app-1 ./charts/app-1 \
  --set image.tag=v1.1.0 \
  -n production
```

## Configuration

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Container image repository | `ghcr.io/dolvladzio/app-*` |
| `image.tag` | Container image tag | `latest` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `5000` |
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.className` | Ingress class | `nginx` |
| `ingress.hosts[0].host` | Hostname | `app-*.example.com` |
| `resources.requests.cpu` | CPU request | `250m` |
| `resources.requests.memory` | Memory request | `256Mi` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `autoscaling.enabled` | Enable HPA | `false` |

### Example values.yaml override

```yaml
# custom-values.yaml
replicaCount: 3

image:
  tag: "v1.2.3"

ingress:
  hosts:
    - host: app-1.mycompany.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: app-1-tls
      hosts:
        - app-1.mycompany.com

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

env:
  - name: FLASK_ENV
    value: "production"
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: database-url
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

Ensure your Flask application implements these endpoints:

```python
@app.route('/health')
def health():
    return {'status': 'healthy'}, 200

@app.route('/ready')
def ready():
    # Check database connection, etc.
    return {'status': 'ready'}, 200
```

## Security

The charts include security best practices:
- Non-root user execution
- Read-only root filesystem option
- Dropped capabilities
- Resource limits
- Network policies ready

## Monitoring

To enable metrics collection, add Prometheus annotations:

```yaml
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "5000"
  prometheus.io/path: "/metrics"
```

## Troubleshooting

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

## CI/CD Integration

The charts are designed to work with GitHub Actions. See `.github/workflows/charts.yaml` for automated chart releases.

## License

MIT

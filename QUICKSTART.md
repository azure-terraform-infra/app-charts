# Quick Deployment Guide

## Deploy all applications

```bash
# Deploy to dev namespace
helm install app-1 ./charts/app-1 -n dev --create-namespace
helm install app2 ./charts/app2 -n dev
helm install app3 ./charts/app3 -n dev

# Deploy to production with custom values
helm install app-1 ./charts/app-1 -n prod --create-namespace \
  --set image.tag=v1.0.0 \
  --set replicaCount=3 \
  --set ingress.hosts[0].host=app-1.yourdomain.com

helm install app2 ./charts/app2 -n prod \
  --set image.tag=v1.0.0 \
  --set replicaCount=3 \
  --set ingress.hosts[0].host=app2.yourdomain.com

helm install app3 ./charts/app3 -n prod \
  --set image.tag=v1.0.0 \
  --set replicaCount=3 \
  --set ingress.hosts[0].host=app3.yourdomain.com
```

## Update application version

```bash
# Upgrade to new version
helm upgrade app-1 ./charts/app-1 -n prod --set image.tag=v1.1.0

# Rollback if needed
helm rollback app-1 -n prod
```

## Verify deployment

```bash
# Check Helm releases
helm list -n prod

# Check pods
kubectl get pods -n prod

# Check services
kubectl get svc -n prod

# Check ingress
kubectl get ingress -n prod
```

## Test the application

```bash
# Port-forward for local testing
kubectl port-forward -n prod svc/app-1 8080:80

# Access via curl
curl http://localhost:8080/health
```

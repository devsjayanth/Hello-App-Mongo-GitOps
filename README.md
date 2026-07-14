# Kubernetes Manifests

All resources deploy to the `prod-space` namespace.

## Application

| File | Resource | Description |
|------|----------|-------------|
| `namespace.yml` | Namespace | `prod-space` — target namespace for all app resources |
| `hello-app-deployment.yml` | Deployment | Main app (`devsjayanth/hello-app`), port 8000, with liveness/readiness probes |
| `hello-app-service.yml` | Service (ClusterIP) | Exposes hello-app on port 8000 internally |
| `hello-app-configmap.yml` | ConfigMap | App env vars: `PORT`, `LOG_LEVEL`, `DEBUG`, `FORWARDED_ALLOW_IPS` |
| `hello-app-hpa.yml` | HPA | Autoscales 2–5 replicas at 70% CPU |
| `hello-app-mongo-ingress.yml` | Ingress | Routes `/` → hello-app, `/mongo` → mongo-express (host replaced at deploy time) |

## Database

| File | Resource | Description |
|------|----------|-------------|
| `mongo.yml` | Deployment + Service | MongoDB on port 27017, credentials from Secret |
| `mongo-secret.yml` | Secret | MongoDB root username/password (base64-encoded) |
| `mongo-express.yml` | Deployment + Service | Mongo Express UI on port 8081, connects to `mongodb-service` |

## Monitoring

| File | Resource | Description |
|------|----------|-------------|
| `grafana-dashboard.yml` | ConfigMap | Grafana dashboard (request rate, error rate, latency, heatmap) — auto-discovered by sidecar |
| `servicemonitor.yml` | ServiceMonitor | Prometheus scrapes `/metrics` every 15s from hello-app |

## Ingress Routing

```
HAProxy (:80/:443) → nginx-ingress (NodePort 30080) → Ingress rules:

  /       → hello-app:8000
  /mongo  → mongo-express-service:8081
```

The `host` field in `hello-app-mongo-ingress.yml` is a placeholder (`PLACEHOLDER`) — the Jenkins pipeline replaces it with the HAProxy EIP at deploy time.

## Apply Order

```bash
kubectl apply -f namespace.yml
kubectl apply -f mongo-secret.yml
kubectl apply -f mongo.yml
kubectl apply -f hello-app-configmap.yml
kubectl apply -f hello-app-deployment.yml
kubectl apply -f hello-app-service.yml
kubectl apply -f mongo-express.yml
kubectl apply -f hello-app-mongo-ingress.yml
kubectl apply -f hello-app-hpa.yml
kubectl apply -f grafana-dashboard.yml
kubectl apply -f servicemonitor.yml
```

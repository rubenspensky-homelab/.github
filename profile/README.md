# Rubenspensky HomeLab

Production-style Kubernetes homelab focused on Platform Engineering, GitOps, Observability, and CI/CD.

```text
Developer
   |
   v
GitHub Actions / ARC
   |
   v
GHCR
   |
   v
GitOps repositories
   |
   v
Argo CD
   |
   v
Kubernetes
   |
   +--> Demo API
   |
   +--> Prometheus --> Grafana
   |
   +--> Loki <-- Alloy
   |
   +--> MetalLB
          |
          v
      Envoy Gateway
          |
          v
   Cloudflare Tunnel
```

## Current Platform

- Debian single-node Kubernetes cluster
- kubeadm
- containerd
- Argo CD
- GitHub Actions Runner Controller
- Cloudflare Tunnel
- Envoy Gateway / Gateway API
- MetalLB
- Prometheus
- Grafana
- Loki
- Alloy
- Sample TypeScript backend API

## Repositories

- `homelab-infra`: GitOps repository for platform infrastructure.
- `homelab-apps`: GitOps repository for application deployments.
- `ts-backend-demo`: sample backend API used to test CI/CD, observability, and public routing.

## CI/CD Flow

1. Developer pushes code.
2. GitHub Actions runs.
3. ARC runner executes jobs inside Kubernetes.
4. Docker image is built and pushed to GHCR.
5. GitOps repository is updated.
6. Argo CD syncs the change.
7. The app is deployed to Kubernetes.

## Observability

- Metrics are collected with Prometheus.
- Dashboards are managed with Grafana.
- Logs are collected with Alloy and stored in Loki.

## Networking

- MetalLB provides local `LoadBalancer` IPs.
- Envoy Gateway handles routing inside Kubernetes using Gateway API.
- Cloudflare Tunnel exposes selected public services.

## Roadmap

- Improve demo API production readiness
- Add API metrics dashboard
- Add OpenTelemetry and Tempo
- Add External Secrets Operator
- Add alerting
- Add backup and restore strategy
- Add incident demo documentation

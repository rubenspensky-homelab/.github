# Rubenspensky Homelab

Production-style Kubernetes homelab focused on platform engineering, GitOps, observability, routing, and CI/CD automation.

This organization contains the source repositories for a self-managed homelab platform. The goal is to run a small but realistic Kubernetes environment with the same operating model used in production systems: declarative infrastructure, automated delivery, centralized routing, telemetry, and clear ownership boundaries between platform and application configuration.

The current environment is a Debian single-node Kubernetes cluster installed with `kubeadm` and `containerd`. Argo CD continuously reconciles both platform infrastructure and application workloads from dedicated GitOps repositories.

## Architecture

![Homelab Kubernetes architecture](https://github.com/rubenspensky-homelab/homelab-infra/raw/main/arch.jpg)

## What This Platform Demonstrates

- Kubernetes cluster operations on a self-managed Debian node.
- GitOps bootstrapping with Argo CD and an app-of-apps model.
- Separation between platform infrastructure and application deployment repositories.
- Gateway API based HTTP routing with Envoy Gateway.
- LAN `LoadBalancer` addressing with MetalLB.
- Public service exposure through Cloudflare Tunnel.
- Cluster observability with metrics, logs, traces, and dashboards.
- Kubernetes-hosted GitHub Actions runners through Actions Runner Controller.
- Container image publishing to GHCR and GitOps-driven application rollout.

## Current Platform

| Area | Components |
| --- | --- |
| Kubernetes | Debian, `kubeadm`, `containerd` |
| GitOps | Argo CD app-of-apps, infrastructure repository, application repository |
| Networking | MetalLB, Envoy Gateway, Gateway API, Cloudflare Tunnel |
| Observability | Prometheus, Grafana, Loki, Tempo, Alloy |
| CI/CD | GitHub Actions, Actions Runner Controller, runner scale set, BuildKit, GHCR |
| Application | TypeScript Backend Demo REST API |

## Repositories

| Repository | Purpose |
| --- | --- |
| [`homelab-infra`](https://github.com/rubenspensky-homelab/homelab-infra) | GitOps source of truth for cluster infrastructure, platform services, routing, and observability. |
| [`homelab-apps`](https://github.com/rubenspensky-homelab/homelab-apps) | GitOps source of truth for application Kubernetes manifests managed by Argo CD. |
| [`ts-backend-demo`](https://github.com/rubenspensky-homelab/ts-backend-demo) | Sample TypeScript Express API used to exercise CI/CD, GitOps deployment, routing, metrics, logs, and tracing. |

## Repository Responsibilities

`homelab-infra` owns platform-level resources:

- Argo CD bootstrap and infrastructure applications.
- MetalLB address pool and L2 advertisement.
- Envoy Gateway and shared Gateway API resources.
- Cloudflare Tunnel deployment.
- Local storage, metrics-server, and observability stack.
- GitHub Actions Runner Controller, ARC runner scale set, and BuildKit.

`homelab-apps` owns application-level resources:

- Argo CD application definitions for workloads.
- Kubernetes manifests for the `ts-backend-demo` service.
- Deployment, Service, ConfigMap, HTTPRoute, and ServiceMonitor resources.

`ts-backend-demo` owns application code:

- TypeScript Node.js backend using Express.
- Health endpoint at `/health`.
- Prometheus-compatible metrics endpoint at `/metrics`.
- OpenTelemetry tracing export over OTLP HTTP.
- CI workflow for test, build, image publish, and GitOps update.

## GitOps Model

The platform uses Argo CD as the deployment control plane. The infrastructure repository bootstraps the full platform and delegates application workloads to the application repository.

1. `homelab-infra/bootstrap/root-app.yaml` registers the root Argo CD application.
2. The root app points Argo CD at `homelab-infra/applications`.
3. `homelab-infra/applications/homelab-apps.yaml` delegates application workloads to `homelab-apps`.
4. `homelab-apps/applications/` registers application deployments.
5. Argo CD reconciles infrastructure and application desired state into Kubernetes with automated prune and self-heal enabled.

This split keeps platform ownership and application ownership separate while still allowing the whole cluster to be operated through Git.

## CI/CD Flow

The demo application validates the end-to-end delivery path from source code to Kubernetes rollout.

1. A developer pushes code to GitHub.
2. GitHub Actions runs tests with `npm test` on `arc-runner-set`.
3. The build job compiles the TypeScript application with `npm run build`.
4. The Docker job uses remote BuildKit at `buildkit.buildkit.svc.cluster.local:1234`.
5. The container image is pushed to GHCR with `latest` and commit SHA tags.
6. The workflow updates `homelab-apps/infrastructure/ts-backend-demo/deployment.yaml` with the commit SHA image.
7. Argo CD reconciles the GitOps change into the cluster.

The deployed application currently runs with two replicas, `/health` readiness and liveness probes, and a `ServiceMonitor` scraping `/metrics` every 30 seconds.

## Routing And Exposure

The cluster uses Envoy Gateway and Gateway API for HTTP routing. MetalLB provides LAN `LoadBalancer` addresses from `192.168.0.240-192.168.0.250`.

The shared Gateway is `homelab-gateway` in the `routing` namespace. It accepts HTTP traffic for:

- `*.home.lab`
- `*.rubenspensky.com`

Current internal routes include:

- `argocd.home.lab`
- `grafana.home.lab`
- `ts-backend-demo.home.lab`

Public exposure is handled through Cloudflare Tunnel for selected services, including `ts-backend-demo.rubenspensky.com`.

## Observability

The observability stack is deployed as platform infrastructure and is available to workloads running in the cluster.

| Signal | Implementation |
| --- | --- |
| Metrics | Prometheus and ServiceMonitor resources |
| Dashboards | Grafana |
| Logs | Alloy collection with Loki storage |
| Traces | OpenTelemetry export through Alloy with Tempo storage |

The TypeScript demo API exposes Prometheus-compatible metrics at `/metrics` and supports OpenTelemetry trace export over OTLP HTTP.

Application metrics include default Node.js/process metrics and HTTP request counters and histograms labelled by method, route, and status code.

## Demo Application

`ts-backend-demo` is intentionally small so the platform behavior is easy to observe. It is an Express-based TypeScript API that provides:

- `GET /health` for readiness and liveness checks.
- `GET /metrics` for Prometheus scraping.
- Structured service configuration through environment variables.
- OpenTelemetry tracing with `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` configured to send traces to Alloy inside the cluster.

The application is routed through the same Gateway API path as platform services, which makes it useful for validating CI/CD, routing, observability, and GitOps reconciliation together.

## Operating Principles

- Git is the source of truth for cluster state.
- Platform infrastructure and application manifests are versioned separately.
- Application delivery updates GitOps manifests instead of mutating cluster state directly.
- Public exposure is limited and routed through Cloudflare Tunnel.
- Secrets should not be committed to Git; External Secrets Operator is planned for secret synchronization.
- Documentation should follow the implemented manifests and workflows rather than describe aspirational components as current state.

## Roadmap

- Add External Secrets Operator with AWS Secrets Manager.
- Add Velero with S3-backed backup and restore workflows.
- Add component-specific documentation and troubleshooting runbooks.
- Add application dashboards and alerting rules as services mature.

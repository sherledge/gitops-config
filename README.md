# GitOps Configuration Repository

This repository manages declarative Kubernetes configurations using **Kustomize** and **ArgoCD** for multi-environment deployments (`dev`, `staging`, `prod`).

---

## Repository Structure

```
gitops-config/
├── base/
│   ├── deployment.yaml      # Workload spec with Layer 1 resilience
│   ├── service.yaml         # Named service with cluster IP
│   ├── hpa.yaml             # Autoscaling v2 (CPU 60% + Mem 70%)
│   ├── pdb.yaml             # PodDisruptionBudget (minAvailable: 1)
│   └── kustomization.yaml   # Base resource manifest
├── overlays/
│   ├── dev/                 # Development environment overlay
│   ├── staging/             # Staging environment overlay
│   └── prod/                # Production environment overlay
└── argocd-app.yaml          # ArgoCD Application manifest
```

---

## Workload Resilience (Layer 1)

All base deployments incorporate production resilience standards:
- **Zero-Downtime Rolling Updates:** `maxUnavailable: 0` and `maxSurge: 1`.
- **Startup Protection:** `startupProbe` with a 30s budget (10 × 3s) buffering slow cold starts.
- **Immediate Ingress Protection:** `readinessProbe` unregisters unhealthy pods within 5 seconds.
- **Graceful Termination:** `terminationGracePeriodSeconds: 60` to complete in-flight HTTP requests.
- **Node Drain Safeguard:** `minAvailable: 1` in `pdb.yaml` guarantees cluster maintenance never evicts all pods simultaneously.
- **Horizontal Autoscaling:** `hpa.yaml` (`autoscaling/v2`) scales 2–5 pods based on CPU and memory utilization, with a 5-minute scale-down stabilization window.

---

## Deployment & Verification

```bash
# Build & validate overlay locally
kubectl kustomize overlays/dev

# Apply changes to cluster directly (or sync via ArgoCD)
kubectl apply -k overlays/dev

# Check rollout status
kubectl -n development rollout status deploy/dev-platform-app
```

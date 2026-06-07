# Findings - MarketLens GitOps Refinement

## Repository Structure
- Root contains `k8s-manifests/`, `Makefile`, `README.md`.
- `k8s-manifests/apps/` contains deployment manifests.
- `k8s-manifests/infrastructure/` contains secrets and minio.

## ArgoCD Configuration
- Repo URL: `https://github.com/MarketLens-AI-Platform/marketlens-gitops`
- Path: `k8s-manifests`

## Manifests Details
- `marketlens-secrets` is the secret name used for `envFrom`.

## GitIgnore Requirements
- Ignore `*.yaml` but keep `*.example.yaml`. (Need to check if `kustomization.yaml` should be kept as well, usually yes).

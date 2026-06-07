# MarketLens GitOps & Infrastructure

This repository contains the infrastructure-as-code and orchestration manifests for the MarketLens AI Platform. It serves as the single source of truth for the platform's Kubernetes environment.

## Directory Structure

- `k8s-manifests/`: Contains Kubernetes YAML manifests and Kustomize configuration.
  - `infrastructure/`: Base infrastructure like MinIO and Secrets templates.
  - `apps/`: Application deployments (Frontend, MCP Server, Ingestion).
  - `kubeflow-fix/`: Specific fixes and configurations for Kubeflow Pipelines.
- `.github/workflows/`: CI/CD pipelines for manifest validation.
- `Makefile`: Central automation script for cluster management and KFP deployment.

## Infrastructure Setup

### Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Make](https://www.gnu.org/software/make/)

### Deployment Workflow

1.  **Start Kubernetes Cluster:**
    ```bash
    make k8s-start
    ```
    This initializes a Minikube cluster with 4 CPUs and 8GB of RAM.

2.  **Install Kubeflow Pipelines (KFP):**
    ```bash
    make kfp-install
    ```
    Deploys KFP v2.5.0 and applies necessary fixes located in `k8s-manifests/kubeflow-fix/`.

3.  **Deploy MarketLens Stack:**
    First, create your secrets:
    ```bash
    cp k8s-manifests/infrastructure/marketlens-secrets.example.yaml k8s-manifests/infrastructure/marketlens-secrets.yaml
    # Edit k8s-manifests/infrastructure/marketlens-secrets.yaml with your API keys
    ```
    Then apply the kustomization:
    ```bash
    kubectl apply -k k8s-manifests/
    ```

4.  **Access the Dashboards:**
    - KFP UI: `make kfp-ui` (forwarded to `http://localhost:8080`)
    - MarketLens Frontend: `kubectl port-forward svc/marketlens-frontend 5000:5000 -n marketlens`

## CI/CD Validation

The repository includes a GitHub Action (`kustomize-lint.yml`) that automatically validates the Kustomize configuration on every push and pull request to `main` or `develop`.

To run validation locally:
```bash
kubectl kustomize k8s-manifests/
```

## Makefile Targets

| Target | Description |
|--------|-------------|
| `k8s-start` | Starts Minikube and enables storage addons. |
| `kfp-install` | Installs KFP cluster-scoped and platform-agnostic resources. |
| `kfp-ui` | Port-forwards the KFP UI for local access. |
| `k8s-clean` | Deletes the Minikube cluster. |
| `k8s-status` | Checks the status of pods and PVCs in the `kubeflow` namespace. |

## GitOps Principles

This repository follows GitOps best practices:
- All infrastructure state is declared in version-controlled manifests.
- Automation is driven by the `Makefile` and CI pipelines to ensure environment consistency.
- Any changes to the cluster should be reflected in the manifests here first.

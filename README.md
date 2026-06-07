# MarketLens GitOps

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/argocd-%23ef7b4d.svg?logo=argo&logoColor=white)
![Kustomize](https://img.shields.io/badge/kustomize-%23326ce5.svg?logo=kubernetes&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

Centralized GitOps repository for the MarketLens AI Platform. This repository manages the Kubernetes state for the entire microservices ecosystem, ensuring declarative and reproducible deployments.

## Architecture Overview

The MarketLens AI Platform is composed of four decoupled layers, each residing in its own repository and integrated via this GitOps layer:

1.  **[marketlens-ingestion](https://github.com/MarketLens-AI-Platform/marketlens-ingestion)**: Autonomous A2A scraping agents for Shopify and WooCommerce. Uses Playwright and Pydantic for robust data extraction.
2.  **[marketlens-llm-mcp](https://github.com/MarketLens-AI-Platform/marketlens-llm-mcp)**: Semantic enrichment pipeline using DeepSeek LLM and LangChain, exposed via a Model Context Protocol (MCP) server.
3.  **[marketlens-mlops](https://github.com/MarketLens-AI-Platform/marketlens-mlops)**: Scalable MLOps layer using Kubeflow. Features XGBoost classification, K-Means clustering, PCA, and Apriori association rules.
4.  **[marketlens-frontend](https://github.com/MarketLens-AI-Platform/marketlens-frontend)**: Real-time BI dashboard built with Flask and Plotly, featuring a LangChain-powered conversational AI assistant.

## ArgoCD Integration

This platform follows a GitOps delivery model using **ArgoCD**. ArgoCD monitors the `k8s-manifests/` directory in this repository and automatically synchronizes the cluster state.

### Bootstrap with ArgoCD

Apply the application manifest to your cluster:

```bash
kubectl apply -f argocd-app.yaml
```

This will create the `marketlens-ai-platform` application in the `argocd` namespace, pointing to the `k8s-manifests` path of this repository.

## Repository Structure

- `k8s-manifests/`: Base Kubernetes manifests organized by category.
  - `apps/`: Deployments and Services for frontend, MCP server, and ingestion cronjobs.
  - `infrastructure/`: Shared resources like MinIO and Secret templates.
  - `kustomization.yaml`: Kustomize aggregation file.
- `argocd-app.yaml`: ArgoCD Application definition.
- `Makefile`: Automation for local cluster setup and deployment.

## Local Development & Deployment

The included `Makefile` provides targets for local Kubernetes management using Minikube:

- `make k8s-start`: Start Minikube with optimized resources.
- `make kfp-install`: Install Kubeflow Pipelines via Kustomize.
- `make deploy-local`: Apply all application manifests to the local cluster.
- `make k8s-status`: Check the status of pods in both `kubeflow` and `marketlens` namespaces.
- `make kfp-ui`: Port-forward the Kubeflow UI to `localhost:8080`.
- `make k8s-clean`: Delete the local Minikube cluster.

## Security

Secrets are managed via `marketlens-secrets`. An example template is provided in `k8s-manifests/infrastructure/marketlens-secrets.example.yaml`. **Never commit actual secrets to this repository.**

---
**Author:** Yassine Kamouss — FST Tanger, LSI 2, 2025/2026

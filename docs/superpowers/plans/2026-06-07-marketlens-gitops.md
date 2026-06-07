# MarketLens GitOps Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a centralized GitOps repository for MarketLens AI Platform, containing Kubernetes manifests, infrastructure configurations, and CI validation.

**Architecture:** Standard Kubernetes manifests organized into `infrastructure/` and `apps/`, aggregated via `Kustomize`. CI pipeline via GitHub Actions to validate Kustomize configurations.

**Tech Stack:** Kubernetes, Kustomize, GitHub Actions, Makefile.

---

### Task 1: Base Setup & Infrastructure Manifests

**Files:**
- Create: `k8s-manifests/infrastructure/minio.yaml`
- Create: `k8s-manifests/infrastructure/marketlens-secrets.example.yaml`
- Modify: `.gitignore`

- [ ] **Step 1: Create MinIO manifest**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: minio
  namespace: marketlens
spec:
  serviceName: "minio"
  replicas: 1
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
      - name: minio
        image: minio/minio:latest
        args:
        - server
        - /data
        - --console-address
        - ":9001"
        ports:
        - containerPort: 9000
          name: api
        - containerPort: 9001
          name: console
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: marketlens
spec:
  ports:
  - port: 9000
    targetPort: 9000
    name: api
  - port: 9001
    targetPort: 9001
    name: console
  selector:
    app: minio
```

- [ ] **Step 2: Create Secrets example manifest**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: marketlens-secrets
  namespace: marketlens
type: Opaque
stringData:
  DEEPSEEK_API_KEY: "your-api-key-here"
  MINIO_ACCESS_KEY: "minioadmin"
  MINIO_SECRET_KEY: "minioadmin"
```

- [ ] **Step 3: Update .gitignore**
```bash
echo "*.yaml" >> .gitignore
echo "!*.example.yaml" >> .gitignore
echo "!kustomization.yaml" >> .gitignore
```

- [ ] **Step 4: Commit**
```bash
git add k8s-manifests/infrastructure/ .gitignore
git commit -m "infra: add minio and secrets template"
```

### Task 2: App Manifests

**Files:**
- Create: `k8s-manifests/apps/frontend-deployment.yaml`
- Create: `k8s-manifests/apps/mcp-server-deployment.yaml`
- Create: `k8s-manifests/apps/scraping-cronjob.yaml`

- [ ] **Step 1: Create Frontend Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: marketlens-frontend
  namespace: marketlens
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ghcr.io/marketlens-ai-platform/marketlens-frontend:latest
        ports:
        - containerPort: 5000
        envFrom:
        - secretRef:
            name: marketlens-secrets
---
apiVersion: v1
kind: Service
metadata:
  name: marketlens-frontend
  namespace: marketlens
spec:
  type: ClusterIP
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app: frontend
```

- [ ] **Step 2: Create MCP Server Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: marketlens-llm-mcp
  namespace: marketlens
spec:
  replicas: 1
  selector:
    matchLabels:
      app: llm-mcp
  template:
    metadata:
      labels:
        app: llm-mcp
    spec:
      containers:
      - name: llm-mcp
        image: ghcr.io/marketlens-ai-platform/marketlens-llm-mcp:latest
        ports:
        - containerPort: 8000
        envFrom:
        - secretRef:
            name: marketlens-secrets
---
apiVersion: v1
kind: Service
metadata:
  name: marketlens-llm-mcp
  namespace: marketlens
spec:
  type: ClusterIP
  ports:
  - port: 8000
    targetPort: 8000
  selector:
    app: llm-mcp
```

- [ ] **Step 3: Create Scraping CronJob**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: marketlens-ingestion
  namespace: marketlens
spec:
  schedule: "0 */6 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: ingestion
            image: ghcr.io/marketlens-ai-platform/marketlens-ingestion:latest
            envFrom:
            - secretRef:
                name: marketlens-secrets
          restartPolicy: OnFailure
```

- [ ] **Step 4: Commit**
```bash
git add k8s-manifests/apps/
git commit -m "apps: add frontend, mcp-server and ingestion cronjob"
```

### Task 3: Kustomize Aggregation & CI

**Files:**
- Create: `k8s-manifests/kustomization.yaml`
- Create: `.github/workflows/kustomize-lint.yml`

- [ ] **Step 1: Create Kustomization file**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: marketlens

resources:
  - infrastructure/minio.yaml
  - apps/frontend-deployment.yaml
  - apps/mcp-server-deployment.yaml
  - apps/scraping-cronjob.yaml
```

- [ ] **Step 2: Create CI Workflow**
```yaml
name: Kustomize Lint

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Validate Kustomize
      run: |
        kubectl kustomize k8s-manifests/
```

- [ ] **Step 3: Commit**
```bash
git add k8s-manifests/kustomization.yaml .github/workflows/kustomize-lint.yml
git commit -m "ci: add kustomization and lint workflow"
```

### Task 4: Documentation & Final Polish

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Rewrite README.md**
(Content per user request)

- [ ] **Step 2: Final Verification**
Run `kubectl kustomize k8s-manifests/` locally to ensure no errors.

- [ ] **Step 3: Commit**
```bash
git add README.md
git commit -m "docs: finalize marketlens-gitops readme"
```

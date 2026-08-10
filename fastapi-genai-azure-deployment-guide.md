# Deploying a FastAPI GenAI App to Azure with CI/CD — All Options

This guide covers every practical way to run a FastAPI-based GenAI solution on Azure, with CI/CD steps for each. Pick the option that matches your team's operational maturity and the app's scaling/GPU needs.

---

## Which option should you pick?

| Option                                | Best for                                                  | Complexity        | Scales to zero      | GPU support                  |
| ------------------------------------- | --------------------------------------------------------- | ----------------- | ------------------- | ---------------------------- |
| **Azure App Service**                 | Simple APIs, fastest to set up                            | Low               | No (always-on)      | No (CPU only)                |
| **Azure Container Apps (ACA)**        | Most GenAI APIs — containerized, event/HTTP-driven        | Medium            | Yes                 | Yes (GPU workload profiles)  |
| **Azure Kubernetes Service (AKS)**    | Complex microservices, full control, multi-team platforms | High              | No (unless KEDA)    | Yes                          |
| **Azure Container Instances (ACI)**   | One-off jobs, batch inference, quick tests                | Low               | N/A (per-container) | Yes                          |
| **Azure Functions**                   | Lightweight/event-driven endpoints, low steady traffic    | Low               | Yes                 | No (Premium plan has limits) |
| **Azure VM (IaaS)**                   | Full OS control, legacy constraints, custom drivers       | High (manual ops) | No                  | Yes                          |
| **Azure ML Managed Online Endpoints** | Model-serving specifically (not general FastAPI)          | Medium            | No                  | Yes                          |

For most GenAI + FastAPI workloads (calling OpenAI/Azure OpenAI, LangChain pipelines, RAG, agents), **Azure Container Apps** is the sweet spot: containerized, autoscaling (including scale-to-zero), supports GPU workload profiles for local model inference, and has first-class GitHub Actions support.

---

## Prerequisites (all options)

1. An Azure subscription (`az login` to verify access).
2. Azure CLI installed: `az --version`.
3. Your FastAPI app in a Git repo (GitHub or Azure Repos).
4. A `requirements.txt` (or `pyproject.toml`) with pinned dependencies.
5. Your app exposes a health endpoint, e.g.:

```python
# main.py
from fastapi import FastAPI
app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}
```

6. A `Dockerfile` (needed for ACA, AKS, ACI, App Service-for-containers, VM):

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Option 1: Azure App Service (code-based, no Docker)

Fastest path if you don't need containers. App Service builds your app for you (Oryx build system).

### Steps

1. **Create resources:**
   
   ```bash
   az group create --name genai-rg --location eastus
   
   ```

az appservice plan create \
  --name genai-plan --resource-group genai-rg \
  --sku B1 --is-linux

az webapp create \
  --name genai-fastapi-app --resource-group genai-rg \
  --plan genai-plan --runtime "PYTHON:3.12"

```

2. **Set the startup command** (App Service needs to know how to run FastAPI/uvicorn):
```bash
az webapp config set \
  --resource-group genai-rg --name genai-fastapi-app \
  --startup-file "uvicorn main:app --host 0.0.0.0 --port 8000"
```

3. **Set environment variables/secrets** (e.g., Azure OpenAI key):
   
   ```bash
   az webapp config appsettings set \
   --resource-group genai-rg --name genai-fastapi-app \
   --settings AZURE_OPENAI_KEY="<value>" AZURE_OPENAI_ENDPOINT="<value>"
   ```

4. **Set up CI/CD with GitHub Actions:**
   
   - In Azure Portal → your App Service → **Deployment Center** → choose **GitHub** → authorize → select repo/branch → this auto-generates a workflow file under `.github/workflows/`.
   - Or manually add `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Azure App Service

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest || true   # remove `|| true` once you have real tests

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v3
        with:
          app-name: genai-fastapi-app
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

5. **Get the publish profile and store it as a GitHub secret:**
   
   ```bash
   az webapp deployment list-publishing-profiles \
   --name genai-fastapi-app --resource-group genai-rg --xml
   ```
   
   Copy the XML output → GitHub repo → **Settings → Secrets and variables → Actions → New repository secret** → name it `AZURE_WEBAPP_PUBLISH_PROFILE`.

6. Push to `main` → GitHub Actions builds and deploys automatically. App is live at `https://genai-fastapi-app.azurewebsites.net`.

---

## Option 2: Azure App Service for Containers (Docker on App Service)

Same App Service platform, but you deploy a container image instead of raw code — useful if your GenAI app needs system-level dependencies (e.g., `poppler`, `ffmpeg`, custom CUDA libs not in the default image).

### Steps

1. **Create an Azure Container Registry (ACR):**
   
   ```bash
   az acr create --resource-group genai-rg --name genaiacr --sku Basic
   ```

2. **Create App Service configured for a container:**
   
   ```bash
   az appservice plan create \
   --name genai-plan --resource-group genai-rg --sku B1 --is-linux
   
   ```

az webapp create \
  --resource-group genai-rg --plan genai-plan \
  --name genai-fastapi-app --deployment-container-image-name genaiacr.azurecr.io/fastapi-genai:latest

```

3. **Enable managed identity so App Service can pull from ACR without embedded credentials:**
```bash
az webapp identity assign --resource-group genai-rg --name genai-fastapi-app
az acr update -n genaiacr --anonymous-pull-enabled false
az role assignment create \
  --assignee <principalId-from-above> \
  --scope $(az acr show -n genaiacr --query id -o tsv) \
  --role AcrPull
```

4. **Create a service principal for GitHub Actions (federated OIDC recommended over secrets):**
   
   ```bash
   az ad sp create-for-rbac --name "genai-github-actions" \
   --role contributor \
   --scopes /subscriptions/<sub-id>/resourceGroups/genai-rg \
   --sdk-auth
   ```
   
   Save the JSON output as a GitHub secret named `AZURE_CREDENTIALS`.

5. **GitHub Actions workflow (`.github/workflows/deploy.yml`):**

```yaml
name: Build and Deploy Container to App Service

on:
  push:
    branches: [main]

env:
  ACR_NAME: genaiacr
  IMAGE_NAME: fastapi-genai
  WEBAPP_NAME: genai-fastapi-app

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Build and push image to ACR
        run: |
          az acr build --registry $ACR_NAME \
            --image $IMAGE_NAME:${{ github.sha }} \
            --image $IMAGE_NAME:latest .

      - name: Deploy to App Service
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.WEBAPP_NAME }}
          images: ${{ env.ACR_NAME }}.azurecr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

> `az acr build` builds the image **inside ACR** (no local Docker daemon needed in the runner) — simpler and avoids Docker-in-Docker issues in CI.

---

## Option 3: Azure Container Apps (recommended for most GenAI FastAPI apps)

Serverless containers with autoscaling (including scale-to-zero), built-in ingress/TLS, revisions for blue-green deploys, and GPU workload profiles if you're running local inference (e.g., open-weight LLMs, embeddings models).

### Steps

1. **Install the Container Apps CLI extension and register providers (one-time):**
   
   ```bash
   az extension add --name containerapp --upgrade
   az provider register --namespace Microsoft.App
   az provider register --namespace Microsoft.OperationalInsights
   ```

2. **Create the environment and ACR:**
   
   ```bash
   az group create --name genai-rg --location eastus
   
   ```

az acr create --resource-group genai-rg --name genaiacr --sku Basic

az containerapp env create \
  --name genai-env --resource-group genai-rg --location eastus

```

3. **Build and push the initial image:**
```bash
az acr build --registry genaiacr --image fastapi-genai:latest .
```

4. **Create the Container App:**
   
   ```bash
   az containerapp create \
   --name genai-fastapi-app --resource-group genai-rg \
   --environment genai-env \
   --image genaiacr.azurecr.io/fastapi-genai:latest \
   --target-port 8000 --ingress external \
   --registry-server genaiacr.azurecr.io \
   --min-replicas 0 --max-replicas 5 \
   --env-vars AZURE_OPENAI_ENDPOINT="<value>" AZURE_OPENAI_KEY=secretref:openai-key \
   --secrets openai-key="<actual-key-value>"
   ```

5. **(Optional) GPU workload profile** — for self-hosted model inference:
   
   ```bash
   az containerapp env workload-profile add \
   --name genai-env --resource-group genai-rg \
   --workload-profile-name gpu-profile \
   --workload-profile-type Consumption-GPU-NC24-A100
   
   ```

az containerapp update \
  --name genai-fastapi-app --resource-group genai-rg \
  --workload-profile-name gpu-profile

```

6. **CI/CD with GitHub Actions (`.github/workflows/deploy.yml`):**

```yaml
name: Deploy to Azure Container Apps

on:
  push:
    branches: [main]

env:
  ACR_NAME: genaiacr
  IMAGE_NAME: fastapi-genai
  RESOURCE_GROUP: genai-rg
  CONTAINERAPP_NAME: genai-fastapi-app

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Build and push image
        run: |
          az acr build --registry $ACR_NAME \
            --image $IMAGE_NAME:${{ github.sha }} .

      - name: Deploy new revision
        uses: azure/container-apps-deploy-action@v2
        with:
          containerAppName: ${{ env.CONTAINERAPP_NAME }}
          resourceGroup: ${{ env.RESOURCE_GROUP }}
          imageToDeploy: ${{ env.ACR_NAME }}.azurecr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

> **OIDC (federated credentials)** avoids storing long-lived secrets in GitHub. Set it up via `az ad app federated-credential create` — see Microsoft's ["Use GitHub Actions to connect to Azure" docs](https://learn.microsoft.com/azure/developer/github/connect-from-azure) for exact steps, since Azure updates the recommended flow periodically.

7. Push to `main` → new revision is built, pushed to ACR, and deployed. Container Apps handles traffic splitting between revisions automatically if you use `--revision-mode multiple`.

---

## Option 4: Azure Kubernetes Service (AKS)

Use this if you already run Kubernetes, need fine-grained control (custom autoscalers, service mesh, multi-tenant namespaces), or are deploying many interdependent GenAI microservices (API + vector DB + workers + queue).

### Steps (high level)

1. **Create the cluster:**
   
   ```bash
   az aks create \
   --resource-group genai-rg --name genai-aks \
   --node-count 2 --generate-ssh-keys \
   --attach-acr genaiacr
   ```

2. **Get credentials locally:**
   
   ```bash
   az aks get-credentials --resource-group genai-rg --name genai-aks
   ```

3. **Write Kubernetes manifests** (`deployment.yaml`, `service.yaml`, `ingress.yaml`) or a Helm chart. Minimal deployment example:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-genai
spec:
  replicas: 2
  selector:
    matchLabels: { app: fastapi-genai }
  template:
    metadata:
      labels: { app: fastapi-genai }
    spec:
      containers:
        - name: fastapi-genai
          image: genaiacr.azurecr.io/fastapi-genai:latest
          ports: [{ containerPort: 8000 }]
          env:
            - name: AZURE_OPENAI_KEY
              valueFrom:
                secretKeyRef: { name: openai-secret, key: key }
---
apiVersion: v1
kind: Service
metadata:
  name: fastapi-genai-svc
spec:
  type: LoadBalancer
  selector: { app: fastapi-genai }
  ports: [{ port: 80, targetPort: 8000 }]
```

4. **CI/CD with GitHub Actions:**

```yaml
name: Deploy to AKS

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Build and push image
        run: az acr build --registry genaiacr --image fastapi-genai:${{ github.sha }} .

      - name: Set AKS context
        uses: azure/aks-set-context@v4
        with:
          resource-group: genai-rg
          cluster-name: genai-aks

      - name: Deploy manifests
        uses: azure/k8s-deploy@v5
        with:
          manifests: |
            k8s/deployment.yaml
            k8s/service.yaml
          images: genaiacr.azurecr.io/fastapi-genai:${{ github.sha }}
```

5. For GPU nodes (self-hosted model inference), add a GPU node pool:
   
   ```bash
   az aks nodepool add \
   --resource-group genai-rg --cluster-name genai-aks \
   --name gpunodes --node-vm-size Standard_NC6s_v3 \
   --node-count 1 --node-taints sku=gpu:NoSchedule
   ```

> AKS is the highest-effort option — only choose it if you need orchestration features App Service/ACA don't give you.

---

## Option 5: Azure Container Instances (ACI)

Good for quick demos, batch jobs, or short-lived inference tasks — not ideal for a persistent public API (no built-in autoscaling or load balancing across instances).

```bash
az container create \
  --resource-group genai-rg --name genai-fastapi-aci \
  --image genaiacr.azurecr.io/fastapi-genai:latest \
  --cpu 2 --memory 4 \
  --ports 8000 --ip-address Public \
  --registry-login-server genaiacr.azurecr.io \
  --registry-username <acr-username> --registry-password <acr-password> \
  --environment-variables AZURE_OPENAI_KEY="<value>"
```

CI/CD: same pattern as Option 2/3 — GitHub Actions runs `az acr build` then `az container create` (or `az container restart` after updating the image tag) on every push. Treat this as a stopgap, not a production serving pattern.

---

## Option 6: Azure Functions (with FastAPI via ASGI adapter)

Works if your GenAI endpoints are lightweight and traffic is spiky/low-volume — you pay per-execution and it scales to zero.

1. **Wrap FastAPI for Functions** using `azure-functions` + an ASGI adapter:

```python
# function_app.py
import azure.functions as func
from main import app as fastapi_app

app = func.AsgiFunctionApp(app=fastapi_app, http_auth_level=func.AuthLevel.ANONYMOUS)
```

2. **Deploy with Azure Functions Core Tools:**
   
   ```bash
   func azure functionapp publish genai-func-app
   ```

3. **CI/CD with GitHub Actions:**
   
   ```yaml
   name: Deploy to Azure Functions
   
   ```

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12' }
      - run: pip install -r requirements.txt --target=".python_packages/lib/site-packages"
      - uses: Azure/functions-action@v1
        with:
          app-name: genai-func-app
          package: '.'
          publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}

```

> Caveats for GenAI workloads: cold starts hurt latency for LLM-calling endpoints, and long-running streaming responses (SSE/WebSocket for token streaming) are awkward on the Consumption plan — use Premium plan if you need this route.

---

## Option 7: Azure VM (IaaS)

Full control — useful if you need custom GPU drivers, non-standard networking, or specific OS-level packages not supported in managed platforms.

1. **Create VM:**
```bash
az vm create \
  --resource-group genai-rg --name genai-vm \
  --image Ubuntu2404 --size Standard_NC6s_v3 \
  --admin-username azureuser --generate-ssh-keys
```

2. **Open port 8000 (or put behind Nginx on 443):**
   
   ```bash
   az vm open-port --resource-group genai-rg --name genai-vm --port 8000
   ```

3. **Install Docker, run `docker run` or `docker compose up -d` for the app + Nginx reverse proxy.**

4. **CI/CD with GitHub Actions (deploy over SSH):**
   
   ```yaml
   name: Deploy to Azure VM
   
   ```

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy over SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VM_HOST }}
          username: azureuser
          key: ${{ secrets.VM_SSH_KEY }}
          script: |
            cd /app
            git pull origin main
            docker compose down
            docker compose up -d --build

```

> This is the most manual option — you own patching, scaling, TLS certs, and failover. Only pick this if a managed option genuinely can't meet a requirement.

---

## Azure DevOps Pipelines (alternative to GitHub Actions)

If your org uses Azure Repos/Azure DevOps instead of GitHub, the equivalent pipeline (example targeting Container Apps) is:

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-latest'

variables:
  acrName: 'genaiacr'
  imageName: 'fastapi-genai'
  containerAppName: 'genai-fastapi-app'
  resourceGroup: 'genai-rg'

steps:
  - task: AzureCLI@2
    inputs:
      azureSubscription: 'genai-service-connection'   # set up under Project Settings > Service connections
      scriptType: bash
      scriptLocation: inlineScript
      inlineScript: |
        az acr build --registry $(acrName) --image $(imageName):$(Build.BuildId) .
        az containerapp update \
          --name $(containerAppName) --resource-group $(resourceGroup) \
          --image $(acrName).azurecr.io/$(imageName):$(Build.BuildId)
```

Set up the service connection: **Project Settings → Service connections → New → Azure Resource Manager → Workload identity federation (recommended)**.

---

## Common CI/CD add-ons (recommended regardless of platform)

```yaml
# Add to any workflow before the deploy step:

- name: Lint
  run: |
    pip install ruff
    ruff check .

- name: Run tests
  run: |
    pip install pytest
    pytest

- name: Security scan (container image)
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'genaiacr.azurecr.io/fastapi-genai:${{ github.sha }}'
    severity: 'HIGH,CRITICAL'
```

**Secrets to never hardcode** — store these in Azure Key Vault and reference them from your compute platform (App Service/ACA support Key Vault references natively) rather than as plain environment variables where possible:

- `AZURE_OPENAI_KEY` / `OPENAI_API_KEY`
- Database connection strings
- Any vector DB / third-party API keys

---

## Summary Decision Tree

```
Do you need Kubernetes-level orchestration or already run K8s?
 ├─ Yes → AKS (Option 4)
 └─ No
     Do you need GPU-backed self-hosted inference or scale-to-zero containers?
      ├─ Yes → Azure Container Apps (Option 3)
      └─ No
          Is traffic spiky/low and endpoints lightweight (no long streaming)?
           ├─ Yes → Azure Functions (Option 6)
           └─ No
               Do you just need the simplest possible deploy, no Docker?
                ├─ Yes → App Service, code-based (Option 1)
                └─ No → App Service for Containers (Option 2) or ACI for quick tests (Option 5)
```

For a production GenAI FastAPI app today, **Azure Container Apps + GitHub Actions + ACR + OIDC federated auth** is the most commonly recommended default — it balances low ops overhead with the flexibility (including GPU profiles) that GenAI workloads often need.

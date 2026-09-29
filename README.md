# Road Events GitOps

ArgoCD GitOps declarative application definitions for the Road Events & Smart Bulb infrastructure on MicroK8s.

## Directory Structure
```text
road-events-gitops/
├── projects/
│   └── road-events-project.yaml # ArgoCD AppProject definition (road-events)
├── applicationset.yaml          # ArgoCD ApplicationSet (1-click deploy for all components)
├── apps/
│   ├── road-events-stack.yaml   # Umbrella chart application (PostGIS + API)
│   ├── postgres.yaml            # Standalone PostGIS database application
│   └── api.yaml                 # Standalone FastAPI backend application
└── README.md
```

---

## 🚀 One-Command Deployment (Using ApplicationSet)

With the **ApplicationSet**, you do **not** need to manually apply multiple YAML files. It automatically generates and orchestrates the applications with ordered sync waves (`postgres` wave 1 ➡️ `api` wave 2).

```bash
# 1. Create the custom ArgoCD Project
microk8s kubectl apply -f projects/road-events-project.yaml -n argocd

# 2. Apply the ApplicationSet (automatically deploys both Postgres and API)
microk8s kubectl apply -f applicationset.yaml -n argocd
```

### Or combine them into a single command:
```bash
microk8s kubectl apply -f projects/road-events-project.yaml -f applicationset.yaml -n argocd
```

ArgoCD will immediately generate:
* **`road-events-postgres`** (Sync Wave 1: PostGIS persistent database)
* **`road-events-api`** (Sync Wave 2: FastAPI backend & Swagger UI)

---

## 🔍 Verify Deployment

```bash
# Check generated applications in ArgoCD
microk8s kubectl get applications -n argocd

# Check running workloads
microk8s kubectl get pods -n road-events
```

---

## 📖 Accessing & Using Swagger UI (REST APIs)

FastAPI provides an interactive OpenAPI / Swagger UI at `/docs`.

### 1. Connect to Swagger UI
Forward the API service port to your local machine:
```bash
microk8s kubectl port-forward svc/road-events-api 8000:8000 -n road-events
```
Open in your browser:
👉 **[http://localhost:8000/docs](http://localhost:8000/docs)** (or simply **[http://localhost:8000/](http://localhost:8000/)** which redirects automatically).

*(If using ngrok tunnel)*:
👉 **`https://unmolded-runway-shelve.ngrok-free.dev/docs`**

### 2. Authenticating in Swagger UI
1. Click the green **Authorize 🔓** button in the top right corner of Swagger UI.
2. In the **Value** box, enter your device token:
   ```text
   secret-device-token-12345
   ```
3. Click **Authorize**, then click **Close**.
4. All endpoints will now automatically send `Authorization: Bearer secret-device-token-12345`.

### 3. Testing Key Endpoints
* **`GET /healthz`**: Check system and database connectivity (no auth required).
* **`PUT /api/v1/rides/{ride_id}/status`**: Send telemetry / trigger emergency alerts.
* **`POST /api/v1/events`**: Upload hazard photo and metadata.
* **`GET /api/v1/events/nearby`**: Spatial radius search (ST_DWithin).
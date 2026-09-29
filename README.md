# Road Events GitOps

ArgoCD GitOps declarative application definitions for the Road Events & Smart Bulb infrastructure on MicroK8s.

## Directory Structure
```text
road-events-gitops/
├── projects/
│   └── road-events-project.yaml # ArgoCD AppProject definition (road-events)
├── apps/
│   ├── road-events-stack.yaml   # Umbrella chart application (PostGIS + API)
│   ├── postgres.yaml            # Dedicated PostGIS database application
│   └── api.yaml                 # Dedicated FastAPI backend application
└── README.md
```

## 🚀 Deploying with ArgoCD

### Step 1: Create the ArgoCD Project
Apply the custom AppProject to ArgoCD:
```bash
microk8s kubectl apply -f projects/road-events-project.yaml -n argocd
```

### Step 2: Deploy the Road-Events Stack
Deploy the umbrella application managed under the `road-events` project:
```bash
microk8s kubectl apply -f apps/road-events-stack.yaml -n argocd
```

*(Alternatively, to deploy only the API component separately)*:
```bash
microk8s kubectl apply -f apps/api.yaml -n argocd
```

### Step 3: Verify ArgoCD Sync
```bash
# Check ArgoCD applications status
microk8s kubectl get applications -n argocd

# Check running pods in road-events namespace
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
👉 **`https://roadevents.ngrok-free.app/docs`**

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
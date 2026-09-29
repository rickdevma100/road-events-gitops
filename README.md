# Road Events GitOps

ArgoCD GitOps declarative application definitions for the Road Events & Smart Bulb infrastructure on MicroK8s.

## Directory Structure
```text
apps/
├── road-events-stack.yaml  # Complete stack application (umbrella chart)
├── postgres.yaml           # Dedicated PostGIS database application
└── api.yaml                # Dedicated FastAPI backend application
```

## Applying via ArgoCD
```bash
# Apply stack application to ArgoCD
microk8s kubectl apply -f apps/road-events-stack.yaml -n argocd
```
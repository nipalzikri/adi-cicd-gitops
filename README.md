# 03 — GitOps: ArgoCD + Kustomize & Envoy Gateway API

Repositori ini mengelola seluruh manifest Kubernetes secara **deklaratif** berbasis prinsip **GitOps**. Setiap perubahan yang di-commit ke repositori ini akan disinkronisasikan secara otomatis ke cluster KinD oleh ArgoCD.

---

## 🏗️ Struktur Repositori

```
03-gitops/
├── bootstrap/
│   ├── install/                     # Manifest instalasi resmi ArgoCD
│   ├── root-application.yaml        # App-of-Apps root application
│   ├── project.yaml                 # ArgoCD AppProject "course"
│   └── apps/                        # Definisi Child Applications
│       ├── infrastructure.yaml      # Parent untuk komponen infra (Envoy Gateway)
│       ├── backend-go-dev.yaml      # Application untuk backend-go dev overlay
│       └── frontend-dev.yaml        # Application untuk UI frontend dev overlay
├── infrastructure/
│   ├── kustomization.yaml           # Entrypoint agregasi infrastruktur
│   └── gateway-api/                 # Kubernetes Gateway API manifests
│       └── gateway.yaml             # EnvoyProxy (KinD hostPort:80) + GatewayClass + Gateway
└── applications/
    ├── common/                      # Shared base patch & manifest template (DRY)
    └── development/
        ├── backend-go/              # Overlay environment development
        └── frontend/                # Overlay environment development UI (Nginx)
            ├── kustomization.yaml   # Target agregasi Kustomize
            ├── deployment.yaml      # Workload Pod definition Nginx
            ├── service.yaml         # ClusterIP service
            ├── patch/
            │   └── deployment.yaml  # Target update tag otomatis dari CI Jenkins
            └── network/
                └── http-route.yaml  # HTTPRoute Gateway API (expose localhost:80)
```

---

## 🔄 Pola Alur GitOps (App-of-Apps Pattern)

1. **Root Application (`root-application.yaml`)**:
   ArgoCD memantau folder `bootstrap/apps/` dan secara otomatis membuat child applications (`infrastructure` dan `frontend-dev`).
2. **Automated Promotion**:
   Pipeline CI di Jenkins mengubah tag image pada `applications/development/frontend/patch/deployment.yaml` ➔ ArgoCD mendeteksi perubahan commit ➔ Melakukan auto-sync ke cluster KinD.

---

## 🌐 Ingress & API Gateway (Envoy Gateway)

- **Gateway Controller**: Envoy Gateway di-install via Helm di namespace `envoy-gateway-system`.
- **Port Binding KinD**: Resource `EnvoyProxy` di [`gateway.yaml`](infrastructure/gateway-api/gateway.yaml) mem-bind pod Envoy ke node `control-plane` via `hostPort: 80`.
- **Akses Langsung**:
  Aplikasi UI Frontend dapat diakses langsung dari browser dan terminal:
  ```bash
  # Akses halaman UI di browser
  http://localhost

  # Uji health check & CI metadata
  curl -i http://localhost/healthz
  ```

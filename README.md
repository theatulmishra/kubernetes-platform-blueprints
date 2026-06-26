# kubernetes-platform-blueprints

[![Kubernetes Lint and Security Scan](https://img.shields.io/github/actions/workflow/status/theatulmishra/kubernetes-platform-blueprints/kube-lint.yml?branch=main&style=flat-square&logo=github-actions&logoColor=white)](https://github.com/theatulmishra/kubernetes-platform-blueprints/actions)
[![Kustomize Ready](https://img.shields.io/badge/Kustomize-v5+-blue?style=flat-square&logo=kubernetes)](https://kustomize.io/)
[![Security Scan](https://img.shields.io/badge/Security-Trivy--Scanned-brightgreen?style=flat-square&logo=security)](https://trivy.dev/)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

Production-ready, cloud-native platform architecture patterns for Kubernetes. This repository contains enterprise blueprints for managing core cluster services, workload isolation, security compliance, scaling, and observability using **Kustomize**.

---

## 🏛️ Platform Architecture Patterns

The repository is structured with a `base/` directory containing generic blueprints (modules) and an `overlays/` directory that adapts those configurations for specific environments (`dev` and `prod`).

### Planned & Implemented Modules

1.  **🚪 Ingress (networking.k8s.io/v1 & cert-manager.io/v1)**
    *   Secure NGINX Ingress rules featuring custom security annotations (SSL redirection, HSTS headers, DDOS shielding).
    *   Cert-Manager `ClusterIssuer` utilizing Let's Encrypt for automated TLS validation and certificate creation.
2.  **📈 Autoscaling (autoscaling/v2 & keda.sh/v1alpha1)**
    *   Standard HorizontalPodAutoscaler (HPA) using target metrics for both CPU and Memory.
    *   KEDA (Kubernetes Event-driven Autoscaling) `ScaledObject` example to scale pods dynamically based on Prometheus query triggers.
3.  **🔑 RBAC (rbac.authorization.k8s.io/v1)**
    *   `platform-developer-role`: Least-privilege role permitting devs to manage standard workloads while restricting infrastructure changes.
    *   `cicd-deployer`: Dedicated ServiceAccount, namespace-scoped Role, and RoleBinding configuration for continuous delivery.
4.  **🛡️ Network Policies (networking.k8s.io/v1)**
    *   `default-deny-all`: Standard Zero-Trust network security rule blocking all namespace traffic by default.
    *   `allow-dns-egress`: Policies explicitly opening egress for CoreDNS resolution on port 53 (TCP/UDP).
    *   `app-segmentation`: Restricts application pods to only receive ingress traffic coming from the Ingress Controller namespace.
5.  **📊 Monitoring (monitoring.coreos.com/v1)**
    *   `ServiceMonitor` to dynamically register target pods for Prometheus discovery.
    *   `PrometheusRule` containing platform alerts for pod crash loops and latency spikes.
6.  **📝 Logging (fluent-bit ConfigMap)**
    *   Fluent Bit log parsing configs, custom Kubernetes metadata filters, and streaming outputs to Elasticsearch/OpenSearch.
7.  **🔒 Secure Deployments (apps/v1 & policy/v1)**
    *   Deployment templates implementing **Pod Security Standards (Restricted Profile)**:
        *   Root filesystem read-only (`readOnlyRootFilesystem: true`).
        *   Non-root execution (`runAsNonRoot: true`, user UID `10001`).
        *   Dropping all Linux capabilities.
        *   Strict CPU/Memory request and limit boundaries.
    *   `PodDisruptionBudget` for production guaranteeing HA availability during maintenance windows.

---

## 📂 Repository Folder Structure

```
kubernetes-platform-blueprints/
├── .github/workflows/
│   └── kube-lint.yml           # CI validation & security scan
├── base/                       # Base modules
│   ├── ingress/                # TLS & Cert-manager routing
│   ├── autoscaling/            # HPA & KEDA configs
│   ├── rbac/                   # User and service security accounts
│   ├── network-policies/       # Workload network isolation
│   ├── monitoring/             # ServiceMonitors & Alert rules
│   ├── logging/                # Fluent-Bit Log Configurations
│   ├── secure-deployments/     # Workloads (PSS restricted)
│   └── kustomization.yaml      # Master base aggregator
└── overlays/                   # Environment overlays
    ├── dev/                    # Development configurations
    └── prod/                   # Production HA configurations
```

---

## 🛠️ How to Use & Deploy

### Prerequisites
*   Kubernetes Cluster (`v1.24+`)
*   `kubectl` client configured with cluster access.

### 1. View Rendered Manifests
Use `kubectl kustomize` to inspect the fully-evaluated YAML structure:

*   **Development Overlay:**
    ```bash
    kubectl kustomize overlays/dev
    ```
*   **Production Overlay:**
    ```bash
    kubectl kustomize overlays/prod
    ```

### 2. Deploy to Your Cluster
Apply the environment configs directly using the Kustomize flag `-k`:

```bash
# Deploy Development stack
kubectl apply -k overlays/dev

# Deploy Production stack
kubectl apply -k overlays/prod
```

---

## 🤖 Code Quality & Validation

### Local Linting
Validate your YAML configurations before committing:
```bash
# Validate format of all manifests
kubectl kustomize overlays/dev | kubeconform -summary
```

### GitHub Actions CI/CD
Every pull request is automatically validated by our pipeline:
1.  **Validation Check (`Kubeconform`):** Verifies all generated manifests conform to target Kubernetes API schemas.
2.  **Security Scans (`Trivy`):** Audits manifests against Pod Security Standards, reporting warning configurations (e.g. running as root, missing limits) and breaking on critical threats.

---

## 📄 License
This repository is licensed under the MIT License. See [LICENSE](LICENSE) for details.

# k8s-lab

<p align="center">
  <strong>GitOps-powered Kubernetes Lab with Flux CD</strong>
  <br>
  A production-ready local Kubernetes environment for learning and experimentation
</p>

<p align="center">
  <a href="https://fluxcd.io/"><img src="https://img.shields.io/badge/GitOps-Flux%20v2.7.4-blue" alt="Flux CD"></a>
  <a href="https://kind.sigs.k8s.io/"><img src="https://img.shields.io/badge/Kind-v1.34.0-326CE5" alt="Kubernetes in Docker"></a>
  <a href="https://istio.io/"><img src="https://img.shields.io/badge/Service%20Mesh-Istio-466BB0" alt="Istio"></a>
  <a href="https://prometheus.io/"><img src="https://img.shields.io/badge/Monitoring-Prometheus-E6522C" alt="Prometheus"></a>
  <br>
  <a href="https://github.com/mozilla/sops"><img src="https://img.shields.io/badge/Secrets-SOPS%20Encrypted-green" alt="SOPS"></a>
  <a href="https://grafana.com/"><img src="https://img.shields.io/badge/Dashboards-Grafana-F46800" alt="Grafana"></a>
</p>

<h3 align="center">
  A fully automated, GitOps-driven Kubernetes lab environment<br>
  featuring monitoring, service mesh, and encrypted secrets management.
</h3>

<p align="center">
  <strong>🚀 GitOps</strong> • <strong>🔐 Encrypted Secrets</strong> • <strong>📊 Observability</strong> • <strong>🌐 Service Mesh</strong>
</p>

## ✨ Why k8s-lab?

- **🚀 GitOps Native**: Everything managed through Git - push changes and watch Flux deploy automatically
- **🔐 Production-Grade Security**: SOPS encryption for secrets with Age keys - safe to commit to Git
- **📊 Full Observability**: Prometheus + Grafana stack with pre-configured dashboards
- **🌐 Modern Networking**: Istio service mesh with Gateway API for advanced traffic management
- **🎯 Best Practices**: Kustomize overlays for multi-environment support (dev/staging/prod)
- **⚡ Fast Setup**: Complete stack deployment in minutes with single cluster creation command
- **🔄 Self-Healing**: Flux continuously reconciles desired state from Git

## 🏗️ Architecture

### Repository Structure

```
k8s-lab/
├── clusters/                          # Per-cluster configurations
│   └── dev/                           # Dev environment
│       └── flux-system/               # Flux bootstrap configs
│           ├── gotk-components.yaml   # Flux controllers
│           ├── gotk-sync.yaml         # Git source & root Kustomization
│           ├── kustomization.yaml     # Bootstrap resource list
│           └── infra-monitoring-kustomization.yaml  # Monitoring stack reference
│
├── infrastructure/                    # Shared infrastructure components
│   └── monitoring/                    # Observability stack
│       ├── base/                      # Base configurations (environment-agnostic)
│       │   ├── namespace.yaml         # Monitoring namespace
│       │   ├── helmrepository-monitoring.yaml  # Prometheus Helm repo
│       │   ├── helmrelease-monitoring.yaml     # kube-prometheus-stack
│       │   └── kustomization.yaml
│       └── overlays/                  # Environment-specific overlays
│           └── dev/                   # Dev environment customizations
│               ├── kustomization.yaml
│               ├── helmrelease-patch.yaml      # Add valuesFrom
│               ├── secret-monitoring-values.yaml  # 🔒 SOPS encrypted
│               └── gateway-grafana.yaml        # Istio Gateway for Grafana
│
├── base/                              # Reusable base resources (currently unused)
├── cluster.yml                        # Kind cluster configuration
├── .sops.yaml                         # SOPS encryption configuration
└── README.md
```

### Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Cluster Runtime** | [Kind](https://kind.sigs.k8s.io/) v1.34.0 | Local Kubernetes cluster (3 nodes: 1 control-plane, 2 workers) |
| **GitOps Engine** | [Flux CD](https://fluxcd.io/) v2.7.4 | Continuous delivery from Git to Kubernetes |
| **Secret Management** | [SOPS](https://github.com/mozilla/sops) + [Age](https://age-encryption.org/) | Encrypted secrets in Git repository |
| **Service Mesh** | [Istio](https://istio.io/) | Traffic management, observability, security |
| **Monitoring** | [kube-prometheus-stack](https://github.com/prometheus-operator/kube-prometheus) v79.7.1 | Prometheus + Grafana + Alertmanager |
| **Configuration** | [Kustomize](https://kustomize.io/) | Multi-environment configuration management |

### GitOps Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  Developer                                                       │
│                                                                  │
│  1. Edit YAML files                                             │
│  2. git commit && git push                                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  GitHub Repository (Source of Truth)                            │
│                                                                  │
│  main branch: clusters/ + infrastructure/                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼ (Flux polls every 1m)
┌─────────────────────────────────────────────────────────────────┐
│  Flux CD Controllers (in Kubernetes)                            │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │ source-controller│  │ kustomize-       │                   │
│  │                  │─▶│ controller       │                   │
│  │ • Pulls Git repo │  │                  │                   │
│  │ • Detects changes│  │ • Builds manifests│                  │
│  │                  │  │ • Decrypts SOPS  │                   │
│  └──────────────────┘  │ • Applies to K8s │                   │
│                        └──────────────────┘                    │
│                               │                                 │
│                               ▼                                 │
│                        ┌──────────────────┐                    │
│                        │ helm-controller  │                    │
│                        │                  │                    │
│                        │ • Installs charts│                    │
│                        │ • Manages releases                    │
│                        └──────────────────┘                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Kubernetes Cluster                                             │
│                                                                  │
│  monitoring namespace:                                          │
│  • Prometheus (metrics collection)                              │
│  • Grafana (dashboards)                                         │
│  • Alertmanager (alerting)                                      │
│                                                                  │
│  istio-system namespace:                                        │
│  • istiod (control plane)                                       │
│  • istio-ingressgateway (traffic ingress)                       │
└─────────────────────────────────────────────────────────────────┘
```

## 📋 Features

### GitOps & Automation
- **Automated Deployment**: Push to Git → Flux deploys within 1 minute
- **Declarative Configuration**: Everything defined as code in YAML
- **Multi-Environment Support**: Base + overlays pattern for dev/staging/prod
- **Drift Detection**: Flux continuously reconciles cluster state with Git

### Security
- **Encrypted Secrets**: SOPS with Age encryption - commit secrets safely to Git
- **Namespace Isolation**: Workloads separated into dedicated namespaces
- **RBAC Ready**: Service mesh provides mTLS and authorization policies

### Observability
- **Metrics Collection**: Prometheus scrapes metrics from all components
- **Visualization**: Grafana dashboards for cluster and application monitoring
- **Alerting**: Alertmanager for notification routing
- **Service Mesh Telemetry**: Istio provides distributed tracing capability

### Networking
- **Istio Gateway**: Modern Gateway API for ingress traffic
- **Traffic Management**: VirtualServices for advanced routing
- **Service Discovery**: Automatic sidecar injection for mesh services

## ⚡ Quick Start

### Prerequisites

Install required tools on your Mac:

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install kind kubectl flux sops age

# Verify installations
kind version
kubectl version --client
flux --version
sops --version
age --version
```

### 1. Clone Repository

```bash
git clone https://github.com/junminhong/k8s-lab.git
cd k8s-lab
```

### 2. Create Kind Cluster

```bash
kind create cluster --config cluster.yml
```

This creates a 3-node cluster:
- 1 control-plane node
- 2 worker nodes
- Port mappings: HTTP (8080) and HTTPS (8443) to NodePort 30080/30443

Verify the cluster:

```bash
kubectl cluster-info
kubectl get nodes
```

### 3. Bootstrap Flux CD

**Important**: You need to fork this repository first and update the Git URL.

```bash
# Set your GitHub username
export GITHUB_USER=<your-username>

# Bootstrap Flux (this will create a deploy key in GitHub)
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=k8s-lab \
  --branch=main \
  --path=./clusters/dev \
  --personal
```

Flux will:
1. Install Flux controllers
2. Create a deploy key in your GitHub repo
3. Start monitoring the repository
4. Deploy all manifests in `clusters/dev`

### 4. Setup SOPS Encryption

Generate an Age key for encrypting secrets:

```bash
# Generate Age key
mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt

# Display public key (you'll need this)
grep "public key:" ~/.config/sops/age/keys.txt
```

Update `.sops.yaml` with your public key:

```yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    age: age1xxx...  # Replace with your public key
```

Create Kubernetes Secret with your private key:

```bash
cat ~/.config/sops/age/keys.txt | \
  kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin
```

### 5. Deploy Infrastructure

Flux automatically deploys everything defined in Git. Watch the deployment:

```bash
# Watch Flux Kustomizations
watch flux get kustomizations

# Watch HelmReleases
watch flux get helmreleases -n monitoring

# Watch Pods in monitoring namespace
watch kubectl get pods -n monitoring
```

Wait for all pods to be `Running` (takes ~2-3 minutes).

### 6. Access Grafana

**Option A: Port Forward (Quick Test)**

```bash
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
```

Add to `/etc/hosts`:

```bash
echo "127.0.0.1 grafana.local" | sudo tee -a /etc/hosts
```

Open browser: http://grafana.local:8080

**Option B: Direct Access (After Cluster Recreation with Port Mappings)**

Open browser: http://grafana.local:8080

**Login Credentials:**
- Username: `admin`
- Password: `jasper` (configured in `infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml`)

## 🔐 Managing Secrets

### Encrypting a New Secret

```bash
# Create a secret file
cat <<EOF > my-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: monitoring
stringData:
  password: "supersecret"
EOF

# Encrypt with SOPS
sops --encrypt --in-place my-secret.yaml
```

### Editing Encrypted Secrets

```bash
# SOPS will decrypt, open editor, and re-encrypt on save
sops infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml
```

### Viewing Encrypted Secrets

```bash
# Decrypt to stdout (don't save)
sops --decrypt infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml
```

## 🛠️ Common Operations

### Force Flux to Sync Now

```bash
# Reconcile specific Kustomization
flux reconcile kustomization infra-monitoring

# Reconcile GitRepository (pull latest from Git)
flux reconcile source git flux-system
```

### Check Flux Status

```bash
# Overall status
flux check

# List all resources
flux get all

# Get Kustomizations
flux get kustomizations

# Get HelmReleases
flux get helmreleases -A

# View logs
flux logs --level=error
```

### Update Monitoring Stack

```bash
# Edit values
sops infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml

# Commit and push
git add infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml
git commit -m "Update Grafana admin password"
git push

# Flux will auto-deploy within 1 minute
# Or force sync:
flux reconcile kustomization infra-monitoring
```

### Add a New Environment (e.g., Staging)

```bash
# 1. Create overlay
cp -r infrastructure/monitoring/overlays/dev infrastructure/monitoring/overlays/staging

# 2. Update values
sops infrastructure/monitoring/overlays/staging/secret-monitoring-values.yaml

# 3. Create cluster Kustomization
cat <<EOF > clusters/staging/flux-system/infra-monitoring-kustomization.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infra-monitoring
  namespace: flux-system
spec:
  interval: 5m0s
  path: ../../infrastructure/monitoring/overlays/staging
  prune: true
  targetNamespace: monitoring
  sourceRef:
    kind: GitRepository
    name: flux-system
  decryption:
    provider: sops
    secretRef:
      name: sops-age
EOF

# 4. Commit and push
git add .
git commit -m "Add staging environment"
git push
```

## 🐛 Troubleshooting

### Flux Not Syncing

```bash
# Check GitRepository status
kubectl get gitrepository -n flux-system

# Check reconciliation errors
flux logs --level=error

# Force reconciliation
flux reconcile source git flux-system
```

### SOPS Decryption Fails

```bash
# Verify sops-age secret exists
kubectl get secret sops-age -n flux-system

# Check Kustomization events
kubectl describe kustomization infra-monitoring -n flux-system

# Recreate the secret
kubectl delete secret sops-age -n flux-system
cat ~/.config/sops/age/keys.txt | \
  kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin
```

### HelmRelease Stuck

```bash
# Check HelmRelease status
flux get helmreleases -n monitoring

# Describe for events
kubectl describe helmrelease kube-prometheus-stack -n monitoring

# Check helm-controller logs
kubectl logs -n flux-system deploy/helm-controller

# Force reconciliation
flux reconcile helmrelease kube-prometheus-stack -n monitoring
```

### Pods Not Starting

```bash
# Check pod status
kubectl get pods -n monitoring

# Describe pod for events
kubectl describe pod <pod-name> -n monitoring

# Check logs
kubectl logs <pod-name> -n monitoring

# Common issues:
# - ImagePullBackOff: Network issues pulling images
# - CrashLoopBackOff: Application errors, check logs
# - Pending: Resource constraints or scheduling issues
```

### Cannot Access Grafana

```bash
# 1. Verify Istio Ingress Gateway is running
kubectl get pods -n istio-system

# 2. Check Gateway and VirtualService
kubectl get gateway,virtualservice -n monitoring

# 3. Verify Grafana service exists
kubectl get svc -n monitoring | grep grafana

# 4. Test with port-forward
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80

# 5. Check /etc/hosts entry
cat /etc/hosts | grep grafana.local

# 6. If using Kind, verify port mappings
docker ps | grep k8s-lab-control-plane
```

### Cluster Port Mapping Issues

If you can't access services from your Mac:

```bash
# 1. Check Kind cluster configuration
kind get clusters
kind export kubeconfig --name k8s-lab

# 2. Verify port mappings on control-plane node
docker ps --filter name=k8s-lab-control-plane --format "{{.Ports}}"

# 3. If ports not mapped, recreate cluster with updated cluster.yml
kind delete cluster --name k8s-lab
kind create cluster --config cluster.yml

# 4. Re-bootstrap Flux
flux bootstrap github --owner=$GITHUB_USER --repository=k8s-lab --branch=main --path=./clusters/dev --personal
```

## 📚 Learning Resources

### Flux CD
- [Official Documentation](https://fluxcd.io/docs/)
- [GitOps Toolkit Components](https://fluxcd.io/docs/components/)
- [Flux Bootstrap Guide](https://fluxcd.io/docs/installation/)

### SOPS & Secrets Management
- [SOPS GitHub](https://github.com/mozilla/sops)
- [Age Encryption](https://age-encryption.org/)
- [Flux SOPS Guide](https://fluxcd.io/docs/guides/mozilla-sops/)

### Kubernetes & Kind
- [Kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)

### Istio Service Mesh
- [Istio Documentation](https://istio.io/latest/docs/)
- [Gateway API](https://gateway-api.sigs.k8s.io/)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

This project is licensed under the MIT License.

## 🙏 Acknowledgments

- [Flux CD](https://fluxcd.io/) - GitOps toolkit for Kubernetes
- [Kind](https://kind.sigs.k8s.io/) - Kubernetes in Docker
- [Prometheus Operator](https://prometheus-operator.dev/) - Kubernetes Prometheus operator
- [Istio](https://istio.io/) - Service mesh platform
- [SOPS](https://github.com/mozilla/sops) - Secrets management

---

<p align="center">
  Made with ❤️ for learning Kubernetes and GitOps
</p>

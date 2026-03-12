# k8s-lab

<p align="center">
  <strong>基於 GitOps 的 Kubernetes 實驗環境</strong>
  <br>
  使用 Flux CD 打造生產級本地 Kubernetes 學習與實驗平台
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

<p align="center">
  <a href="README.md"><img src="https://img.shields.io/badge/English-available-lightgrey" alt="English"></a>
  <a href="#"><img src="https://img.shields.io/badge/繁體中文-selected-blue" alt="繁體中文"></a>
</p>

<h3 align="center">
  完全自動化的 GitOps 驅動 Kubernetes 實驗環境<br>
  整合監控、服務網格與加密的機密管理功能
</h3>

<p align="center">
  <strong>🚀 GitOps</strong> • <strong>🔐 加密機密</strong> • <strong>📊 可觀測性</strong> • <strong>🌐 服務網格</strong>
</p>

## ✨ 為什麼選擇 k8s-lab？

- **🚀 原生 GitOps**: 所有配置透過 Git 管理 - 推送變更後 Flux 自動部署
- **🔐 生產級安全性**: 使用 Age 金鑰的 SOPS 加密 - 機密資料可安全提交到 Git
- **📊 完整可觀測性**: Prometheus + Grafana 堆疊與預配置儀表板
- **🌐 現代化網路**: Istio 服務網格搭配 Gateway API 進行進階流量管理
- **🎯 最佳實踐**: Kustomize overlays 模式支援多環境（dev/staging/prod）
- **⚡ 快速部署**: 單一命令即可在數分鐘內完成整個堆疊部署
- **🔄 自我修復**: Flux 持續調和 Git 中定義的期望狀態

## 🏗️ 架構

### 專案結構

```
k8s-lab/
├── clusters/                          # 各集群配置
│   └── dev/                           # 開發環境
│       └── flux-system/               # Flux 啟動配置
│           ├── gotk-components.yaml   # Flux 控制器
│           ├── gotk-sync.yaml         # Git 來源與根 Kustomization
│           ├── kustomization.yaml     # 啟動資源清單
│           └── infra-monitoring-kustomization.yaml  # 監控堆疊參照
│
├── infrastructure/                    # 共享基礎設施元件
│   └── monitoring/                    # 可觀測性堆疊
│       ├── base/                      # 基礎配置（環境無關）
│       │   ├── namespace.yaml         # Monitoring 命名空間
│       │   ├── helmrepository-monitoring.yaml  # Prometheus Helm 倉庫
│       │   ├── helmrelease-monitoring.yaml     # kube-prometheus-stack
│       │   └── kustomization.yaml
│       └── overlays/                  # 環境特定覆蓋
│           └── dev/                   # 開發環境客製化
│               ├── kustomization.yaml
│               ├── helmrelease-patch.yaml      # 添加 valuesFrom
│               ├── secret-monitoring-values.yaml  # 🔒 SOPS 加密
│               └── gateway-grafana.yaml        # Grafana 的 Istio Gateway
│
├── base/                              # 可重用基礎資源（目前未使用）
├── cluster.yml                        # Kind 集群配置
├── .sops.yaml                         # SOPS 加密配置
└── README.md
```

### 技術堆疊

| 元件 | 技術 | 用途 |
|------|------|------|
| **集群執行環境** | [Kind](https://kind.sigs.k8s.io/) v1.34.0 | 本地 Kubernetes 集群（3 節點：1 control-plane、2 workers）|
| **GitOps 引擎** | [Flux CD](https://fluxcd.io/) v2.7.4 | 從 Git 到 Kubernetes 的持續交付 |
| **機密管理** | [SOPS](https://github.com/mozilla/sops) + [Age](https://age-encryption.org/) | Git 倉庫中的加密機密 |
| **服務網格** | [Istio](https://istio.io/) | 流量管理、可觀測性、安全性 |
| **監控** | [kube-prometheus-stack](https://github.com/prometheus-operator/kube-prometheus) v79.7.1 | Prometheus + Grafana + Alertmanager |
| **配置管理** | [Kustomize](https://kustomize.io/) | 多環境配置管理 |

### GitOps 工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│  開發者                                                          │
│                                                                  │
│  1. 編輯 YAML 檔案                                               │
│  2. git commit && git push                                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  GitHub 倉庫（單一事實來源）                                      │
│                                                                  │
│  main 分支：clusters/ + infrastructure/                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼ (Flux 每分鐘輪詢)
┌─────────────────────────────────────────────────────────────────┐
│  Flux CD 控制器（在 Kubernetes 中）                              │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │ source-controller│  │ kustomize-       │                   │
│  │                  │─▶│ controller       │                   │
│  │ • 拉取 Git 倉庫  │  │                  │                   │
│  │ • 偵測變更       │  │ • 建構 manifests │                   │
│  │                  │  │ • 解密 SOPS      │                   │
│  └──────────────────┘  │ • 應用到 K8s     │                   │
│                        └──────────────────┘                    │
│                               │                                 │
│                               ▼                                 │
│                        ┌──────────────────┐                    │
│                        │ helm-controller  │                    │
│                        │                  │                    │
│                        │ • 安裝 charts    │                    │
│                        │ • 管理 releases  │                    │
│                        └──────────────────┘                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Kubernetes 集群                                                 │
│                                                                  │
│  monitoring 命名空間：                                           │
│  • Prometheus（指標收集）                                        │
│  • Grafana（儀表板）                                             │
│  • Alertmanager（告警）                                          │
│                                                                  │
│  istio-system 命名空間：                                         │
│  • istiod（控制平面）                                            │
│  • istio-ingressgateway（流量入口）                              │
└─────────────────────────────────────────────────────────────────┘
```

## 📋 功能特性

### GitOps 與自動化
- **自動部署**：推送到 Git → Flux 在 1 分鐘內部署
- **聲明式配置**：所有內容都以 YAML 程式碼定義
- **多環境支援**：Base + overlays 模式支援 dev/staging/prod
- **漂移偵測**：Flux 持續調和集群狀態與 Git 的一致性

### 安全性
- **加密機密**：SOPS 搭配 Age 加密 - 安全地將機密提交到 Git
- **命名空間隔離**：工作負載分離到專用命名空間
- **RBAC 就緒**：服務網格提供 mTLS 和授權策略

### 可觀測性
- **指標收集**：Prometheus 從所有元件抓取指標
- **視覺化**：Grafana 儀表板用於集群和應用程式監控
- **告警**：Alertmanager 進行通知路由
- **服務網格遙測**：Istio 提供分散式追蹤能力

### 網路
- **Istio Gateway**：現代化 Gateway API 用於入口流量
- **流量管理**：VirtualServices 進行進階路由
- **服務發現**：自動為網格服務注入 sidecar

## ⚡ 快速開始

### 前置需求

在你的 Mac 上安裝必要工具：

```bash
# 安裝 Homebrew（如果尚未安裝）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安裝相依套件
brew install kind kubectl flux sops age

# 驗證安裝
kind version
kubectl version --client
flux --version
sops --version
age --version
```

### 1. 克隆專案

```bash
git clone https://github.com/junminhong/k8s-lab.git
cd k8s-lab
```

### 2. 創建 Kind 集群

```bash
kind create cluster --config cluster.yml
```

這會創建一個 3 節點集群：
- 1 個 control-plane 節點
- 2 個 worker 節點
- 端口映射：HTTP (8080) 和 HTTPS (8443) 對應到 NodePort 30080/30443

驗證集群：

```bash
kubectl cluster-info
kubectl get nodes
```

### 3. 啟動 Flux CD

**重要**：你需要先 fork 這個專案並更新 Git URL。

```bash
# 設定你的 GitHub 使用者名稱
export GITHUB_USER=<your-username>

# 啟動 Flux（這會在 GitHub 創建部署金鑰）
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=k8s-lab \
  --branch=main \
  --path=./clusters/dev \
  --personal
```

Flux 會：
1. 安裝 Flux 控制器
2. 在你的 GitHub 倉庫創建部署金鑰
3. 開始監控倉庫
4. 部署 `clusters/dev` 中的所有 manifests

### 4. 設定 SOPS 加密

產生用於加密機密的 Age 金鑰：

```bash
# 產生 Age 金鑰
mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt

# 顯示公鑰（你會需要它）
grep "public key:" ~/.config/sops/age/keys.txt
```

使用你的公鑰更新 `.sops.yaml`：

```yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    age: age1xxx...  # 替換成你的公鑰
```

使用你的私鑰創建 Kubernetes Secret：

```bash
cat ~/.config/sops/age/keys.txt | \
  kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin
```

### 5. 部署基礎設施

Flux 會自動部署 Git 中定義的所有內容。觀察部署過程：

```bash
# 觀察 Flux Kustomizations
watch flux get kustomizations

# 觀察 HelmReleases
watch flux get helmreleases -n monitoring

# 觀察 monitoring 命名空間中的 Pods
watch kubectl get pods -n monitoring
```

等待所有 pods 變成 `Running` 狀態（約需 2-3 分鐘）。

### 6. 訪問 Grafana

**方案 A：Port Forward（快速測試）**

```bash
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
```

添加到 `/etc/hosts`：

```bash
echo "127.0.0.1 grafana.local" | sudo tee -a /etc/hosts
```

在瀏覽器開啟：http://grafana.local:8080

**方案 B：直接訪問（使用端口映射重建集群後）**

在瀏覽器開啟：http://grafana.local:8080

**登入憑證：**
- 使用者名稱：`admin`
- 密碼：`jasper`（在 `infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml` 中配置）

## 🔐 管理機密

### 加密新機密

```bash
# 創建機密檔案
cat <<EOF > my-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: monitoring
stringData:
  password: "supersecret"
EOF

# 使用 SOPS 加密
sops --encrypt --in-place my-secret.yaml
```

### 編輯加密的機密

```bash
# SOPS 會解密、開啟編輯器，並在儲存時重新加密
sops infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml
```

### 查看加密的機密

```bash
# 解密到標準輸出（不儲存）
sops --decrypt infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml
```

## 🛠️ 常用操作

### 強制 Flux 立即同步

```bash
# 調和特定 Kustomization
flux reconcile kustomization infra-monitoring

# 調和 GitRepository（從 Git 拉取最新內容）
flux reconcile source git flux-system
```

### 檢查 Flux 狀態

```bash
# 整體狀態
flux check

# 列出所有資源
flux get all

# 取得 Kustomizations
flux get kustomizations

# 取得 HelmReleases
flux get helmreleases -A

# 查看日誌
flux logs --level=error
```

### 更新監控堆疊

```bash
# 編輯 values
sops infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml

# 提交並推送
git add infrastructure/monitoring/overlays/dev/secret-monitoring-values.yaml
git commit -m "更新 Grafana 管理員密碼"
git push

# Flux 會在 1 分鐘內自動部署
# 或強制同步：
flux reconcile kustomization infra-monitoring
```

### 添加新環境（例如 Staging）

```bash
# 1. 創建 overlay
cp -r infrastructure/monitoring/overlays/dev infrastructure/monitoring/overlays/staging

# 2. 更新 values
sops infrastructure/monitoring/overlays/staging/secret-monitoring-values.yaml

# 3. 創建集群 Kustomization
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

# 4. 提交並推送
git add .
git commit -m "添加 staging 環境"
git push
```

## 🐛 故障排除

### Flux 未同步

```bash
# 檢查 GitRepository 狀態
kubectl get gitrepository -n flux-system

# 檢查調和錯誤
flux logs --level=error

# 強制調和
flux reconcile source git flux-system
```

### SOPS 解密失敗

```bash
# 驗證 sops-age secret 存在
kubectl get secret sops-age -n flux-system

# 檢查 Kustomization 事件
kubectl describe kustomization infra-monitoring -n flux-system

# 重新創建 secret
kubectl delete secret sops-age -n flux-system
cat ~/.config/sops/age/keys.txt | \
  kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin
```

### HelmRelease 卡住

```bash
# 檢查 HelmRelease 狀態
flux get helmreleases -n monitoring

# 描述以查看事件
kubectl describe helmrelease kube-prometheus-stack -n monitoring

# 檢查 helm-controller 日誌
kubectl logs -n flux-system deploy/helm-controller

# 強制調和
flux reconcile helmrelease kube-prometheus-stack -n monitoring
```

### Pods 無法啟動

```bash
# 檢查 pod 狀態
kubectl get pods -n monitoring

# 描述 pod 以查看事件
kubectl describe pod <pod-name> -n monitoring

# 檢查日誌
kubectl logs <pod-name> -n monitoring

# 常見問題：
# - ImagePullBackOff：拉取映像時的網路問題
# - CrashLoopBackOff：應用程式錯誤，檢查日誌
# - Pending：資源限制或調度問題
```

### 無法訪問 Grafana

```bash
# 1. 驗證 Istio Ingress Gateway 正在運行
kubectl get pods -n istio-system

# 2. 檢查 Gateway 和 VirtualService
kubectl get gateway,virtualservice -n monitoring

# 3. 驗證 Grafana service 存在
kubectl get svc -n monitoring | grep grafana

# 4. 使用 port-forward 測試
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80

# 5. 檢查 /etc/hosts 條目
cat /etc/hosts | grep grafana.local

# 6. 如果使用 Kind，驗證端口映射
docker ps | grep k8s-lab-control-plane
```

### 集群端口映射問題

如果無法從 Mac 訪問服務：

```bash
# 1. 檢查 Kind 集群配置
kind get clusters
kind export kubeconfig --name k8s-lab

# 2. 驗證 control-plane 節點的端口映射
docker ps --filter name=k8s-lab-control-plane --format "{{.Ports}}"

# 3. 如果端口未映射，使用更新的 cluster.yml 重建集群
kind delete cluster --name k8s-lab
kind create cluster --config cluster.yml

# 4. 重新啟動 Flux
flux bootstrap github --owner=$GITHUB_USER --repository=k8s-lab --branch=main --path=./clusters/dev --personal
```

## 📚 學習資源

### Flux CD
- [官方文件](https://fluxcd.io/docs/)
- [GitOps Toolkit 元件](https://fluxcd.io/docs/components/)
- [Flux Bootstrap 指南](https://fluxcd.io/docs/installation/)

### SOPS 與機密管理
- [SOPS GitHub](https://github.com/mozilla/sops)
- [Age 加密](https://age-encryption.org/)
- [Flux SOPS 指南](https://fluxcd.io/docs/guides/mozilla-sops/)

### Kubernetes 與 Kind
- [Kind 快速開始](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Kubernetes 文件](https://kubernetes.io/docs/home/)

### Istio 服務網格
- [Istio 文件](https://istio.io/latest/docs/)
- [Gateway API](https://gateway-api.sigs.k8s.io/)

## 🤝 貢獻

歡迎貢獻！請隨時提交 Pull Request。

## 📝 授權

本專案採用 MIT 授權。

## 🙏 致謝

- [Flux CD](https://fluxcd.io/) - Kubernetes GitOps 工具組
- [Kind](https://kind.sigs.k8s.io/) - Kubernetes in Docker
- [Prometheus Operator](https://prometheus-operator.dev/) - Kubernetes Prometheus operator
- [Istio](https://istio.io/) - 服務網格平台
- [SOPS](https://github.com/mozilla/sops) - 機密管理

---

<p align="center">
  用 ❤️ 打造，專注於學習 Kubernetes 與 GitOps
</p>

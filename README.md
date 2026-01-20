# GitOps & Progressive Delivery with ArgoCD + Argo Rollouts 🚀

![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-orange?style=for-the-badge&logo=argo)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-blue?style=for-the-badge&logo=kubernetes)
![Canary](https://img.shields.io/badge/Strategy-Canary_Deployment-yellow?style=for-the-badge)

This project is a hands-on infrastructure example that demonstrates
**GitOps principles** and **Progressive Delivery strategies** on
**Kubernetes** using **ArgoCD** and **Argo Rollouts**.

Traditional `kubectl apply` workflows are intentionally avoided.
Instead, **Git is the Single Source of Truth**, and the entire
deployment lifecycle is automatically managed through GitOps. To ensure
production stability, a **Canary Deployment** strategy is implemented.

## 🏗 Architecture & Flow

1.  **GitOps (ArgoCD):** Continuously monitors the Git repository and
    synchronizes the Kubernetes cluster to match the desired state.
    Self-healing is enabled.
2.  **Progressive Delivery (Argo Rollouts):** When a new version (image
    tag) is introduced, traffic is gradually shifted (starting at
    **20%**) instead of an immediate 100% cutover.
3.  **Manual Promotion:** After validating the new version, traffic is
    manually promoted to 100%.

------------------------------------------------------------------------

## 📂 Project Structure

``` bash
.
├── app/
│   ├── rollout.yaml    # Argo Rollouts definition (Canary strategy)
│   └── service.yaml    # Kubernetes Service
├── application.yaml    # ArgoCD Application (entry point for GitOps)
└── README.md           # Documentation
```

------------------------------------------------------------------------

## 🚀 Installation

### Prerequisites

-   Kubernetes Cluster (Minikube, Kind, or Cloud)
-   kubectl CLI
-   ArgoCD and Argo Rollouts controllers installed in the cluster

### 1. Controller Installation

If your cluster is empty, install required components:

``` bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

### 2. Bootstrap GitOps

Tell ArgoCD to start watching this repository:

``` bash
kubectl apply -f application.yaml
```

ArgoCD will automatically deploy the manifests under the `app/`
directory.

------------------------------------------------------------------------

## 🚦 Scenario: How Canary Deployment Works

This project uses **Canary Deployment** instead of Blue-Green. Traffic
flow:

`20% (New Version) → PAUSE (Manual Approval) → 100% (Full Rollout)`

### Step 1: Trigger a New Version

Update the image tag in `app/rollout.yaml` and push to Git (e.g. `blue`
→ `yellow`).

``` yaml
spec:
  containers:
  - name: demo-app
    image: argoproj/rollouts-demo:yellow
```

``` bash
git add .
git commit -m "feat: upgrade app to yellow version"
git push
```

### Step 2: Observe

ArgoCD detects the change. Argo Rollouts shifts only 20% of traffic to
the new version.

``` bash
kubectl argo rollouts get rollout demo-app --watch
```

Access via browser after port-forward: `http://localhost:8081`

### Step 3: Promote

After validating the new version:

``` bash
kubectl argo rollouts promote demo-app
```

All traffic is shifted to the new version and the old ReplicaSet is
removed.

------------------------------------------------------------------------
## 🛠 Useful Commands

| Task | Command|
|-------|--------|
| UI Access (ArgoCD) | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| UI Access (Demo App) | `kubectl port-forward svc/demo-app-svc 8081:80` |
| Rollout Status | `kubectl argo rollouts get rollout demo-app` |
| Emergency (Abort) | `kubectl argo rollouts abort demo-app` |
| Admin Password | `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"` |

------------------------------------------------------------------------


## 💡 Why It Matters

This project demonstrates real-world **production deployment patterns**
used by platform teams.

It proves hands-on experience with:

-   **GitOps mindset** and declarative infrastructure
-   **Progressive delivery** for reducing deployment risk
-   **ArgoCD & Argo Rollouts** in real scenarios
-   Operational safety via **manual promotion & rollback**
-   Kubernetes-native release management

------------------------------------------------------------------------

Developed by **Bilal Yılmaz**

------------------------------------------------------------------------

------------------------------------------------------------------------

# GitOps & Progressive Delivery (Türkçe)

Bu proje, **Kubernetes** üzerinde **GitOps** prensiplerini ve
**Progressive Delivery (Kademeli Dağıtım)** stratejilerini uygulayan
örnek bir altyapı çalışmasıdır.

Geleneksel `kubectl apply` kullanımı bilinçli olarak devre dışı
bırakılmıştır. Tüm deployment süreci **Git** üzerinden yönetilir ve Git
**Single Source of Truth** olarak kabul edilir. Canlı ortam
stabilitesini korumak için **Canary Deployment** uygulanmıştır.

## 🏗 Mimari ve Akış

1.  **GitOps (ArgoCD):** GitHub reposunu izler ve cluster durumunu
    otomatik olarak senkronize eder (Self-Healing aktiftir).
2.  **Progressive Delivery (Argo Rollouts):** Yeni versiyon geldiğinde
    trafik %20 oranında yeni versiyona yönlendirilir.
3.  **Manual Promotion:** Sağlık kontrolü sonrası manuel onay ile trafik
    %100'e çıkarılır.

------------------------------------------------------------------------

## 📂 Proje Yapısı

``` bash
.
├── app/
│   ├── rollout.yaml
│   └── service.yaml
├── application.yaml
└── README.md
```

------------------------------------------------------------------------

## 🚀 Kurulum

### Ön Gereksinimler

-   Kubernetes Cluster (Minikube, Kind veya Cloud)
-   kubectl
-   ArgoCD ve Argo Rollouts Controller

### 1. Controller Kurulumları

``` bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

### 2. GitOps Başlatma

``` bash
kubectl apply -f application.yaml
```

------------------------------------------------------------------------

## 🚦 Canary Deployment Senaryosu

Trafik akışı: `%20 Yeni Versiyon → Pause → %100 Geçiş`

Yeni versiyon için `rollout.yaml` dosyasında image tag değiştirilir ve
Git'e pushlanır.

Durum izleme:

``` bash
kubectl argo rollouts get rollout demo-app --watch
```

Onaylama:

``` bash
kubectl argo rollouts promote demo-app
```

------------------------------------------------------------------------

## 🛠 Faydalı Komutlar

| Görev | Komut |
|-------|--------|
| UI Erişimi (ArgoCD) | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| UI Erişimi (Demo App) | `kubectl port-forward svc/demo-app-svc 8081:80` |
| Rollout Durumu | `kubectl argo rollouts get rollout demo-app` |
| Acil Durum (Abort) | `kubectl argo rollouts abort demo-app` |
| Admin Şifresi | `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"` |

------------------------------------------------------------------------

## 💡 Neden Önemli?

Bu proje, gerçek hayatta platform ekiplerinin kullandığı:

-   **GitOps**
-   **Kademeli dağıtım**
-   **Risk azaltma stratejileri**
-   **Argo ekosistemi**

konularında pratik deneyim sunar.

------------------------------------------------------------------------

Geliştiren: **Bilal Yılmaz**

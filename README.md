# GitOps & Progressive Delivery with ArgoCD + Argo Rollouts 🚀

![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-orange?style=for-the-badge&logo=argo)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-blue?style=for-the-badge&logo=kubernetes)
![Canary](https://img.shields.io/badge/Strategy-Canary_Deployment-yellow?style=for-the-badge)

Bu proje, **Kubernetes** üzerinde **GitOps** prensiplerini ve **Progressive Delivery (Kademeli Dağıtım)** stratejilerini uygulayan örnek bir altyapı çalışmasıdır.

Geleneksel `kubectl apply` komutları yasaklanmış olup, tüm deployment süreci **Git** üzerinden otomatik olarak yönetilmektedir (Single Source of Truth). Ayrıca, canlı ortamın stabilitesini korumak için **Canary Deployment** stratejisi uygulanmıştır.

## 🏗 Mimari ve Akış

1.  **GitOps (ArgoCD):** GitHub reposundaki değişiklikleri izler ve Kubernetes cluster'ı ile senkronize eder. "Self-Healing" özelliği aktiftir.
2.  **Progressive Delivery (Argo Rollouts):** Yeni bir versiyon (Image Tag) geldiğinde trafiği anında %100 çevirmek yerine, önce **%20** oranında yeni versiyona yönlendirir.
3.  **Manual Promotion:** Yeni versiyonun sağlıklı çalıştığı doğrulandıktan sonra manuel onay ile trafik %100'e tamamlanır.

---

## 📂 Proje Yapısı

```bash
.
├── app/
│   ├── rollout.yaml    # Argo Rollouts tanımı (Canary stratejisi burada)
│   └── service.yaml    # Kubernetes Service tanımı
├── application.yaml    # ArgoCD Uygulama konfigürasyonu (Cluster'a ilk giriş noktası)
└── README.md           # Proje dokümantasyonu
```

---

## 🚀 Kurulum (Installation)
Ön Gereksinimler
- Kubernetes Cluster (Minikube, Kind veya Cloud)

- kubectl CLI

- ArgoCD ve Argo Rollouts Controller'larının cluster'da kurulu olması.

**1. Controller Kurulumları**
Eğer cluster boş ise aşağıdaki komutlarla gerekli araçları kurun:
```bash
# ArgoCD Kurulumu
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

# Argo Rollouts Kurulumu
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f [https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml](https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml)
```

**2. GitOps Başlatma**
ArgoCD'ye repoyu izlemesi emrini verin:
```bash
kubectl apply -f application.yaml
```
Bu işlemden sonra ArgoCD, `app/` klasöründeki manifestleri cluster'a deploy edecektir.

## 🚦 Senaryo: Canary Deployment Nasıl Çalışır?
Bu projede Blue-Green yerine Canary stratejisi kullanılmıştır. Trafik geçişi şu şekildedir: `%20 (Yeni Versiyon) -> PAUSE (Onay Bekler) -> %100 (Tam Geçiş)`

**Adım 1: Yeni Versiyonu Tetikleme**
`app/rollout.yaml` dosyasında image tag'ini değiştirin ve Git'e pushlayın (Örn: `blue` -> `yellow`).

```bash
spec:
      containers:
      - name: demo-app
        image: argoproj/rollouts-demo:yellow # Değişiklik burada
```
```bash
git add .
git commit -m "feat: upgrade app to yellow version"
git push
```

**Adım 2: Gözlemleme**
ArgoCD değişikliği algılar. Argo Rollouts, trafiğin sadece %20'sini yeni versiyona yönlendirir. Durumu izlemek için:
```bash
kubectl argo rollouts get rollout demo-app --watch
```
Tarayıcıdan test etmek için: `http://localhost:8081` (Port-forward sonrası)

**Adım 3: Onaylama (Promote)**
Yeni versiyonun (Sarı kutucuklar) hatasız çalıştığını gördükten sonra dağıtımı tamamlayın:
```bash
kubectl argo rollouts promote demo-app
```
Tüm podlar yeni versiyona geçer ve eski versiyon (ReplicaSet) silinir.

---

## 🛠 Faydalı Komutlar

| Görev | Komut |
|-------|--------|
| UI Erişimi (ArgoCD) | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| UI Erişimi (Demo App) | `kubectl port-forward svc/demo-app-svc 8081:80` |
| Rollout Durumu | `kubectl argo rollouts get rollout demo-app` |
| Acil Durum (Abort) | `kubectl argo rollouts abort demo-app` |
| Admin Şifresi | `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"` |

---

## 👨‍💻 Yazar
Bu proje Bilal Yılmaz tarafından GitOps pratiklerini pekiştirmek amacıyla oluşturulmuştur.

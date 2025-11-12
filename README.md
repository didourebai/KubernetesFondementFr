# 🚀 Formation Kubernetes pour Débutants

Ce guide a pour objectif de vous accompagner pas à pas dans la découverte de **Kubernetes**, en partant des bases jusqu’à des déploiements complets avec monitoring.

---

## 🧭 Objectifs

- Comprendre la structure d’un cluster Kubernetes (Cluster, Nœuds, Pods)
- Créer un cluster local avec Docker Desktop
- Déployer et exposer des applications
- Gérer les configurations et le stockage
- Superviser et automatiser les déploiements avec Helm

---

## ⚙️ Activer Kubernetes dans Docker Desktop

### 🔹 Étape 1 : Accéder aux paramètres

1. Ouvre **Docker Desktop**
2. Va dans **Settings → Kubernetes**
3. Active l’option **Enable Kubernetes**

---

## 🏗️ Choisir la méthode de provisionnement du cluster

Quand tu actives Kubernetes, Docker Desktop te propose deux méthodes :

### 🧩 **kubeadm**

- Crée un **cluster single-node** (un seul nœud maître/worker).
- Simule un environnement **proche de la production**.
- Meilleure compatibilité avec les **composants système** (DaemonSets, drivers...).
- Plus lent à créer et à mettre à jour.

### 🧱 **kind** (Kubernetes in Docker)

- Chaque nœud du cluster est un **conteneur Docker**.
- Léger, rapide à démarrer, facile à réinitialiser.
- Parfait pour les **tests, ateliers et formations**.
- Multi-nœuds possible très facilement via un fichier `kind.yaml`.

---

### 💡 Recommandation

| Cas d’usage | Choix recommandé |
|--------------|------------------|
| Formation, atelier, démo | ✅ **kind** |
| Simulation proche de la production | 🔧 **kubeadm** |

> ⚠️ **Attention** : changer la version de Kubernetes dans Docker Desktop **réinitialise complètement le cluster** (tous les Pods, Deployments, Services, etc. sont supprimés).

---

## 🧰 Création d’un cluster Kind (exemple multi-nœuds)

Créer un cluster à 3 nœuds avec un seul fichier :

```bash
kind create cluster --name k8s-lab --config - <<'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
networking:
  apiServerAddress: "127.0.0.1"
EOF
```
Vérifie que ton cluster est prêt :
```bash
kubectl get nodes
```
# 🧪 Ateliers Kubernetes

## 🧱 Atelier 1: Premier Pod

Créer un fichier `nginx-pod.yaml` :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
Commandes à exécuter :

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl port-forward pod/nginx-pod 8080:80
```

➡️ Accéder à l’application via : http://localhost:8080

## ⚙️ Atelier 2 : Déployer et exposer une application

Créer un fichier `web-deploy.yaml` :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
Commandes à exécuter :
```bash
kubectl apply -f web-deploy.yaml
kubectl apply -f web-service.yaml
kubectl get all
```

➡️ Application accessible via: http://localhost:30080

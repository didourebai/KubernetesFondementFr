
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

### 🎯 Objectif  
Déployer une application web Nginx avec un `Deployment` Kubernetes et l’exposer via un `Service` de type **NodePort** pour y accéder depuis le navigateur.

---

### 🧩 Étape 1 : Créer le fichier `web-deploy.yaml`

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

### 🧩 Étape 2 : Créer le fichier `web-service.yaml`

Ce fichier définit un **Service Kubernetes** de type `NodePort` qui permet d’exposer ton application Nginx en dehors du cluster.

Crée un fichier nommé **web-service.yaml** et ajoute le contenu suivant :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```
Commandes à exécuter :

➡️ Application accessible via: http://localhost:30080

### Étape 3 : Appliquer les manifests

```bash
kubectl apply -f web-deploy.yaml
kubectl apply -f web-service.yaml
```
Vérifie que tout est bien créé :
```bash
kubectl get all
```
Tu devrais voir :

- Un Deployment nommé web-deploy

- Un ReplicaSet gérant 3 Pods

- 3 Pods en statut Running

- Un Service web-service exposant le port 30080
### Étape 4 : Vérification et débogage
## A) Pods et Service

```bash
kubectl get pods -l app=web -o wide
kubectl get svc web-service -o wide
kubectl get endpoints web-service
```
- Les Pods doivent être Running et Ready (1/1).

- ENDPOINTS doit lister 3 adresses (si replicas: 3).

- Si ENDPOINTS est vide, corriger les labels/selector puis :

```bash
kubectl rollout restart deployment/web-deploy
kubectl get endpoints web-service
```
### Étape 5 : Accès au navigateur (si OK sinon port-forward)
Ouvre : http://localhost:30080
ou : http://kubernetes.docker.internal:30080
Si l’accès NodePort ne répond pas sur ta machine (firewall/routage), utilise *port-forward* :
```bash
kubectl port-forward svc/web-service 30080:80
```
Puis ouvre : http://localhost:30080

### 💡 Pourquoi ça marche toujours ?

`kubectl port-forward` crée un **tunnel direct** entre ta machine et le **Service** via le **kube-apiserver**, sans dépendre du réseau, du `NodePort` ou du routage Docker.

Cette commande établit une connexion sécurisée entre ton poste local et le cluster Kubernetes.  
Ainsi, le trafic envoyé à `localhost` est redirigé directement vers le Pod ou le Service ciblé à l’intérieur du cluster.

C’est la méthode la plus fiable pour **tester ou déboguer localement** une application, que tu sois sur :
- 🐳 **Docker Desktop**
- 🔹 **kind**
- ☸️ **minikube**
- 🌩️ ou même un **cluster distant**

> ⚙️ En résumé : `port-forward` contourne les problèmes de réseau et de routage  
> il t’assure un accès direct et immédiat à ton application dans le cluster Kubernetes.

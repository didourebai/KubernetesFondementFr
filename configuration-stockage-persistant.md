
# 🧩 Configuration et stockage persistant
## 🎯 Objectifs pédagogiques

À la fin de ce module, les participants sauront :

- Gérer la configuration et les secrets des applications (ConfigMap & Secret)

- Comprendre et utiliser les volumes et le stockage persistant (PV, PVC, StorageClass)

- Déployer une application avec base de données et données persistantes

- Manipuler la configuration dynamique d’une application sans reconstruire d’image

## Concepts clés
Kubernetes gère les configurations et les données de façon déclarative :
- **ConfigMap** : stocke des variables non sensibles (URL, paramètres)
- **Secret** : stocke des informations sensibles (mots de passe, tokens)
- **Volumes** : espace de stockage temporaire ou persistant
- **PersistentVolume (PV)** : ressource de stockage fournie par un administrateur
- **PersistentVolumeClaim (PVC)** : demande de stockage faite par un utilisateur
- **StorageClass** : définit la manière dont les volumes sont provisionnés dynamiquement
### ConfigMap – Configuration non sensible
#### 🔍 Concept
Un ConfigMap stocke des paires clé-valeur (ou fichiers) injectées dans les pods via variables d’environnement ou fichiers montés.
# Atelier Kubernetes — ConfigMap, Secret, Volumes

## 1. Atelier 1 – Manipuler un ConfigMap

Créez `configmap.yaml` (voir exemple dans le cours) puis appliquez-le :

```bash
kubectl apply -f configmap.yaml
kubectl get configmap app-config -o yaml
```

Créez un pod pour utiliser ce ConfigMap :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-config
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["sleep", "3600"]
    envFrom:
    - configMapRef:
        name: app-config
```

Vérifiez les variables d'environnement dans le pod :

```bash
kubectl exec -it demo-config -- env | grep APP_
```

Résultat attendu : les variables d’environnement proviennent du ConfigMap sans rebuild d’image.

---

## 2. Secret – Données sensibles

### Concept (Secrets)

Les Secrets stockent des valeurs confidentielles (mots de passe, clés, tokens). Les valeurs sont encodées en Base64.

### Exemple YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: cm9vdHBhc3M=  # "rootpass" encodé en Base64
```

### Utilisation dans un pod (Secret)

```yaml
env:
- name: MYSQL_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: password
```

### Atelier 2 – Tester un Secret

Créez le secret :

```bash
kubectl create secret generic mysql-secret --from-literal=password=rootpass
```

Créez un pod qui utilise ce secret :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["sleep", "3600"]
    env:
    - name: MYSQL_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: password
```

Vérifiez dans le conteneur :

```bash
kubectl exec -it secret-demo -- env | grep MYSQL_PASSWORD
```

Résultat attendu : la variable `MYSQL_PASSWORD` contient le mot de passe décodé.

---

## 3. Volumes, PV et PVC – Stockage persistant

### Concept (Volumes)

- Un PersistentVolume (PV) décrit un stockage physique.
- Un PersistentVolumeClaim (PVC) est la demande d’un pod pour ce stockage.
- Un Volume monte ce PVC dans le conteneur.

### Exemple PV + PVC

Fichier `pv.yaml` :

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Fichier `pvc.yaml` :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

### Utilisation dans un pod (PVC)

```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: pvc-demo
volumeMounts:
- mountPath: "/data"
  name: data
```

### Atelier 3 – Créer un volume persistant

Créez `pv.yaml` et `pvc.yaml`, puis appliquez :

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl get pv,pvc
```

Créez un pod (nginx ou alpine) qui utilise le PVC. Créez un fichier dans `/data`, supprimez le pod, recréez-le et vérifiez : le fichier doit toujours être présent.

Résultat attendu : les données persistent même après suppression du pod.

---

## 4. Cas pratique – Application + Base de données persistante

### Objectif

Déployer une application web + base MySQL persistante, avec Secret et PVC.

#### Étape 1 – Secret MySQL

```bash
kubectl create secret generic mysql-secret --from-literal=password=rootpass
```

#### Étape 2 – Déploiement MySQL

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: pvc-demo
```

#### Étape 3 – Déploiement web (ex : Nginx)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx
        ports:
        - containerPort: 80
```

#### Étape 4 – Service d’exposition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
  - port: 80
    nodePort: 30080
```

Résultat attendu :

- Base MySQL persistante via PVC
- Secret MySQL sécurisé
- Webapp accessible sur [http://localhost:30080](http://localhost:30080)

---

## 5. Bonnes pratiques

- Ne jamais stocker de mots de passe en clair dans un manifest.
- Isoler les ressources par namespace (dev, test, prod).
- Utiliser des labels clairs (ex. `app=mysql`, `tier=db`).
- Sur cloud : employer une StorageClass dynamique.
- Pour les données critiques : prévoir sauvegarde/restauration (Velero, Restic).

---

## 6. Synthèse du module

| Élément   | Description           | Exemple                 |
|-----------|-----------------------|-------------------------|
| ConfigMap | Config non sensible   | Variables d’environnement |
| Secret    | Données sensibles     | Mot de passe MySQL      |
| PV        | Stockage physique     | /mnt/data               |
| PVC       | Demande de volume     | 500 MiB                 |
| Volume    | Montage dans le pod   | /var/lib/mysql          |

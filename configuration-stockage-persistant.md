
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
#### 🧪 Atelier 1 – Manipuler un ConfigMap
 1. Créer le fichier `configmap.yaml`
2. Appliquer le ConfigMap
```bash
kubectl apply -f configmap.yaml
kubectl get configmap app-config -o yaml
```
3. Créer un Pod pour l’utiliser
```bash
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
4. Vérifier les variables d’environnement
```bash
kubectl exec -it demo-config -- env | grep APP_
```
Résultat attendu :
Les variables d’environnement proviennent du ConfigMap sans rebuild de l’image.

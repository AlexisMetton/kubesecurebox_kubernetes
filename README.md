# 🚀 KubeSecureBox - Déploiement Kubernetes

Ce répertoire contient tous les manifests Kubernetes nécessaires pour déployer KubeSecureBox dans un cluster Kubernetes.

## ⚠️ Configuration initiale (IMPORTANT)

Avant de déployer, vous devez créer vos fichiers de configuration à partir des templates `.example` :

```bash
# 1. Copier les fichiers templates
cp k8s.env.example k8s.env
cp secret.yaml.example secret.yaml
cp configmap.yaml.example configmap.yaml
cp vpn-config.yaml.example vpn-config.yaml   # Optionnel, si vous utilisez le VPN

# 2. Éditer les fichiers avec vos valeurs réelles
# - k8s.env : Adresse de votre registre Docker, mots de passe
# - secret.yaml : Secrets encodés en base64
# - configmap.yaml : URLs et configuration des services
# - vpn-config.yaml : Votre configuration OpenVPN
```

> **🔒 Sécurité** : Les fichiers `k8s.env`, `secret.yaml`, `configmap.yaml` et `vpn-config.yaml` contiennent des données sensibles et sont exclus du dépôt Git via `.gitignore`. Ne les commitez jamais !

## 📋 Prérequis

- **Kubernetes Cluster** : Version 1.20+ (Docker Desktop, Minikube, ou cluster cloud)
- **kubectl** : Outil de ligne de commande Kubernetes
- **envsubst** : Outil de substitution de variables (package `gettext`)
- **Ingress Controller** : Nginx Ingress Controller installé
- **Images Docker** : Toutes les images des services doivent être construites et disponibles
- **Registry Docker** : Registry local ou distant accessible

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Nginx         │    │   Ingress       │
│   (React)       │◄───┤   (Reverse      │◄───┤   Controller    │
│   Port: 3000    │    │    Proxy)       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Gateway   │    │   Redis         │    │   PostgreSQL    │
│   Port: 8000    │◄───┤   Port: 6379    │    │   Port: 5432    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Microservices                                │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│  │ Nmap        │ │ Hydra       │ │ SQLMap      │ │ John        │ │
│  │ Port: 8001  │ │ Port: 8002  │ │ Port: 8003  │ │ Port: 8004  │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│  │ Aircrack    │ │ Gobuster    │ │ Nikto       │ │ WPScan      │ │
│  │ Port: 8005  │ │ Port: 8006  │ │ Port: 8007  │ │ Port: 8008  │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
│  ┌─────────────┐ ┌─────────────┐                                 │
│  │ Enum4linux  │ │ CrackMapExec│                                 │
│  │ Port: 8009  │ │ Port: 8010  │                                 │
│  └─────────────┘ └─────────────┘                                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────┐    ┌─────────────────┐
│   VPN Service   │    │   RabbitMQ      │
│   Port: 8080    │    │   Port: 5672    │
└─────────────────┘    └─────────────────┘
```

## 📁 Structure des fichiers

```
kubesecurebox/
├── namespace.yaml              # Namespace principal
├── storage.yaml                # PersistentVolumeClaims
├── infrastructure.yaml         # Redis, PostgreSQL, RabbitMQ
├── microservices.yaml          # Tous les services de sécurité (utilise ${DOCKER_REGISTRY})
├── frontend.yaml               # Frontend React et Nginx (utilise ${DOCKER_REGISTRY})
├── deploy.sh                   # Script de déploiement
├── README.md                   # Ce fichier
├── .gitignore                  # Fichiers à exclure de Git
│
├── # Fichiers templates (à personnaliser)
├── k8s.env.example             # Template des variables d'environnement
├── secret.yaml.example         # Template des secrets Kubernetes
├── configmap.yaml.example      # Template des ConfigMaps
├── vpn-config.yaml.example     # Template de la configuration VPN
│
└── # Fichiers sensibles (créés localement, NON commités)
    ├── k8s.env                 # Vos variables d'environnement
    ├── secret.yaml             # Vos secrets
    ├── configmap.yaml          # Votre configuration
    └── vpn-config.yaml         # Votre config VPN
```

## ⚙️ Configuration

### Variables d'environnement

Le fichier `k8s.env` contient les variables d'environnement (créez-le à partir de `k8s.env.example`) :

```bash
# Configuration du registry Docker
DOCKER_REGISTRY=your-registry.example.com:5000

# Configuration PostgreSQL
POSTGRES_DB=kubesecurebox
POSTGRES_USER=your_postgres_user
POSTGRES_PASSWORD=your_secure_password

# Configuration RabbitMQ
RABBITMQ_USER=your_rabbitmq_user
RABBITMQ_PASSWORD=your_secure_password
```

### Secrets

Configurez les secrets dans `secret.yaml` :

```bash
# Encoder vos valeurs en base64
echo -n "votre_mot_de_passe" | base64
echo -n "votre_utilisateur" | base64
```

## 🚀 Déploiement

### 1. Préparation des images Docker

Avant de déployer, assurez-vous que toutes les images Docker sont construites :

```bash
# Dans le répertoire docker/kubesecurebox/
docker-compose build
```

### 2. Déploiement automatique

```bash
# Rendre le script exécutable
chmod +x deploy.sh

# Déploiement dans le namespace par défaut (kubesecurebox)
./deploy.sh

# Ou spécifier un namespace personnalisé
./deploy.sh mon-namespace
```

### 3. Déploiement manuel

```bash
# 1. Charger les variables d'environnement
export $(cat k8s.env | grep -v '^#' | xargs)

# 2. Créer le namespace
kubectl apply -f namespace.yaml

# 3. Appliquer les secrets et configs
kubectl apply -f secret.yaml
kubectl apply -f configmap.yaml
kubectl apply -f vpn-config.yaml

# 4. Appliquer le stockage
kubectl apply -f storage.yaml

# 5. Déployer l'infrastructure (avec substitution de variables)
envsubst < infrastructure.yaml | kubectl apply -f -

# 6. Attendre que l'infrastructure soit prête
kubectl wait --for=condition=available --timeout=300s deployment/redis -n kubesecurebox
kubectl wait --for=condition=available --timeout=300s deployment/postgres -n kubesecurebox
kubectl wait --for=condition=available --timeout=300s deployment/rabbitmq -n kubesecurebox

# 7. Déployer les microservices (avec substitution de variables)
envsubst < microservices.yaml | kubectl apply -f -

# 8. Déployer le frontend (avec substitution de variables)
envsubst < frontend.yaml | kubectl apply -f -

# Configurer l'ingress
kubectl apply -f ingress.yaml
```

## 🔧 Configuration

### Variables d'environnement

Les variables d'environnement sont configurées dans `configmap.yaml` :

- **REDIS_URL** : Connexion à Redis
- **DATABASE_URL** : Connexion à PostgreSQL
- **RABBITMQ_URL** : Connexion à RabbitMQ
- **Ports des services** : Configuration des ports de chaque service

### Secrets

Les secrets sensibles sont dans `secret.yaml` :

- **postgres-password** : Mot de passe PostgreSQL
- **postgres-user** : Utilisateur PostgreSQL
- **postgres-db** : Nom de la base de données
- **vpn-username** : Nom d'utilisateur VPN
- **vpn-password** : Mot de passe VPN

### Configuration VPN

Modifiez `vpn-config.yaml` avec votre configuration ProtonVPN réelle.

## 🌐 Accès à l'application

### Via Ingress (recommandé)

1. Ajoutez l'entrée dans `/etc/hosts` :
   ```
   127.0.0.1 kubesecurebox.local
   ```

2. Accédez à l'application :
   - **Frontend** : http://kubesecurebox.local
   - **API** : http://kubesecurebox.local/api

### Via Port-Forward (développement)

```bash
# Frontend
kubectl port-forward service/frontend-service 3000:3000 -n kubesecurebox

# API Gateway
kubectl port-forward service/api-gateway-service 8000:8000 -n kubesecurebox

# Services individuels
kubectl port-forward service/nmap-service 8001:8001 -n kubesecurebox
```

## 📊 Monitoring et logs

### Vérifier le statut

```bash
# Statut des déploiements
kubectl get deployments -n kubesecurebox

# Statut des services
kubectl get services -n kubesecurebox

# Statut des pods
kubectl get pods -n kubesecurebox

# Statut de l'ingress
kubectl get ingress -n kubesecurebox
```

### Consulter les logs

```bash
# Logs du frontend
kubectl logs -f deployment/frontend -n kubesecurebox

# Logs de l'API Gateway
kubectl logs -f deployment/api-gateway -n kubesecurebox

# Logs d'un service spécifique
kubectl logs -f deployment/nmap-service -n kubesecurebox
```

### Accès aux pods

```bash
# Accès bash à un pod
kubectl exec -it deployment/nmap-service -n kubesecurebox -- bash

# Accès aux logs en temps réel
kubectl logs -f deployment/nmap-service -n kubesecurebox
```

## 🧹 Nettoyage

### Nettoyage automatique

```bash
# Rendre le script exécutable
chmod +x cleanup.sh

# Nettoyage complet
./cleanup.sh
```

### Nettoyage manuel

```bash
# Supprimer le namespace (supprime tout)
kubectl delete namespace kubesecurebox

# Ou supprimer ressources par ressources
kubectl delete -f ingress.yaml
kubectl delete -f frontend.yaml
kubectl delete -f microservices.yaml
kubectl delete -f infrastructure.yaml
kubectl delete -f storage.yaml
kubectl delete -f configmap.yaml
kubectl delete -f secret.yaml
kubectl delete -f vpn-config.yaml
kubectl delete -f namespace.yaml
```

## 🔍 Dépannage

### Problèmes courants

1. **Images non trouvées** :
   ```bash
   # Vérifier que les images sont construites
   docker images | grep kubesecurebox
   ```

2. **Ports déjà utilisés** :
   ```bash
   # Vérifier les ports utilisés
   netstat -tulpn | grep :8080
   ```

3. **Services non accessibles** :
   ```bash
   # Vérifier la connectivité réseau
   kubectl exec -it deployment/nmap-service -n kubesecurebox -- ping redis-service
   ```

4. **Problèmes de stockage** :
   ```bash
   # Vérifier les PVCs
   kubectl get pvc -n kubesecurebox
   kubectl describe pvc -n kubesecurebox
   ```

### Vérification de la santé

```bash
# Vérifier la santé des services
kubectl get endpoints -n kubesecurebox

# Vérifier les événements
kubectl get events -n kubesecurebox --sort-by='.lastTimestamp'

# Vérifier la configuration des services
kubectl describe service api-gateway-service -n kubesecurebox
```

## 📚 Ressources supplémentaires

- [Documentation Kubernetes](https://kubernetes.io/docs/)
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## 🤝 Support

Pour toute question ou problème :

1. Vérifiez les logs des services
2. Consultez la documentation Kubernetes
3. Vérifiez la configuration des manifests
4. Assurez-vous que toutes les dépendances sont installées

---

**Note** : Ce déploiement est conçu pour un environnement de développement/test. Pour la production, considérez :
- L'activation de TLS/HTTPS
- La configuration de la haute disponibilité
- La mise en place de monitoring avancé
- La configuration de sauvegarde des données

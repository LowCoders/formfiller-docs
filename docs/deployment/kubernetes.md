# Kubernetes Telepítés

Ez az útmutató a FormFiller Kubernetes telepítéséhez nyújt segítséget.

## Előfeltételek

- Kubernetes cluster (1.24+)
- Helm 3.x
- kubectl
- Ingress controller (nginx-ingress ajánlott)

## Gyors Indítás (Minikube)

### Minikube Indítása

```bash
# Minikube indítása
minikube start --memory 4096 --cpus 2

# Ingress addon
minikube addons enable ingress

# Registry addon (image-ekhez)
minikube addons enable registry
```

### Telepítés

```bash
cd formfiller-deployment

# Image-ek építése
./scripts/build-images.sh minikube

# Telepítés
./scripts/deploy.sh minikube
```

### Elérés

```bash
# hosts fájl beállítása
echo "$(minikube ip) formfiller.local" | sudo tee -a /etc/hosts

# Alkalmazás megnyitása
open http://formfiller.local
```

## Helm Chart

### Chart Struktúra

```
helm-chart/
├── Chart.yaml
├── values.yaml
├── values-minikube.yaml
├── values-staging.yaml
├── values-production.yaml
└── templates/
    ├── backend-deployment.yaml
    ├── backend-service.yaml
    ├── frontend-deployment.yaml
    ├── frontend-service.yaml
    ├── mongodb-statefulset.yaml
    ├── ingress.yaml
    ├── configmap.yaml
    └── secret.yaml
```

### Telepítés Helm-mel

```bash
# Namespace létrehozása
kubectl create namespace formfiller

# Titkos kulcsok létrehozása
kubectl create secret generic formfiller-secrets \
  --from-literal=jwt-secret=$(openssl rand -base64 32) \
  --from-literal=session-secret=$(openssl rand -base64 32) \
  --from-literal=mongodb-password=$(openssl rand -base64 16) \
  -n formfiller

# Helm telepítés
helm install formfiller ./helm-chart \
  -n formfiller \
  -f ./helm-chart/values-production.yaml
```

### Helm Frissítés

```bash
helm upgrade formfiller ./helm-chart \
  -n formfiller \
  -f ./helm-chart/values-production.yaml
```

## values.yaml Konfiguráció

### Alapértelmezett Értékek

```yaml
# Backend konfiguráció
backend:
  replicaCount: 2
  image:
    repository: formfiller-backend
    tag: latest
    pullPolicy: IfNotPresent
  resources:
    requests:
      memory: "256Mi"
      cpu: "100m"
    limits:
      memory: "512Mi"
      cpu: "500m"

# Frontend konfiguráció
frontend:
  replicaCount: 2
  image:
    repository: formfiller-frontend
    tag: latest
    pullPolicy: IfNotPresent
  resources:
    requests:
      memory: "128Mi"
      cpu: "50m"
    limits:
      memory: "256Mi"
      cpu: "200m"

# MongoDB konfiguráció
mongodb:
  enabled: true
  replicaCount: 1
  persistence:
    enabled: true
    size: 10Gi
    storageClass: standard

# Redis konfiguráció
redis:
  enabled: true
  persistence:
    enabled: true
    size: 1Gi

# Ingress konfiguráció
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: formfiller.local
      paths:
        - path: /
          pathType: Prefix
  tls: []
```

### Production Értékek

```yaml
# values-production.yaml
backend:
  replicaCount: 3
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "1000m"

frontend:
  replicaCount: 3

mongodb:
  replicaCount: 3
  persistence:
    size: 50Gi
    storageClass: ssd

redis:
  enabled: true
  cluster:
    enabled: true

ingress:
  tls:
    - secretName: formfiller-tls
      hosts:
        - formfiller.yourdomain.com
```

## Multisite Telepítés

### Deployment Modellek

A FormFiller három multisite módot támogat:

#### 1. Shared Backend - Shared DB

```yaml
# values.yaml
multisite:
  enabled: true
  model: shared-backend-shared-db
```

Egyetlen backend és adatbázis, tenant szűréssel.

#### 2. Shared Backend - Isolated DB

```yaml
multisite:
  enabled: true
  model: shared-backend-isolated-db
```

Egyetlen backend, tenant-enként külön adatbázis.

#### 3. Isolated Backend

```yaml
multisite:
  enabled: true
  model: isolated-backend
  autoProvision: true
```

Tenant-enként külön backend és adatbázis pod.

### Provisioning Service

Az Isolated Backend módhoz szükséges a provisioning service:

```bash
# Provisioning service telepítése
helm install formfiller-provisioning ./provisioning-service/helm-chart \
  -n formfiller
```

## Monitoring és Logging

### Prometheus Monitoring

```yaml
# values.yaml
monitoring:
  prometheus:
    enabled: true
    serviceMonitor:
      enabled: true
```

### Logok

```bash
# Backend logok
kubectl logs -f deployment/formfiller-backend -n formfiller

# Összes pod logja
kubectl logs -f -l app=formfiller -n formfiller
```

## Skálázás

### Horizontális Pod Autoscaler

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: formfiller-backend
  namespace: formfiller
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: formfiller-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### Manuális Skálázás

```bash
kubectl scale deployment formfiller-backend --replicas=5 -n formfiller
```

## Backup és Visszaállítás

### MongoDB Backup (CronJob)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mongodb-backup
  namespace: formfiller
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: mongo:7
              command:
                - /bin/sh
                - -c
                - mongodump --uri="$MONGODB_URI" --out=/backup/$(date +%Y%m%d)
              volumeMounts:
                - name: backup
                  mountPath: /backup
          volumes:
            - name: backup
              persistentVolumeClaim:
                claimName: backup-pvc
          restartPolicy: OnFailure
```

## Rollback

### Helm Rollback

```bash
# Előzmények megtekintése
helm history formfiller -n formfiller

# Visszaállítás előző verzióra
helm rollback formfiller -n formfiller

# Visszaállítás adott verzióra
helm rollback formfiller 3 -n formfiller
```

### Deployment Rollback

```bash
kubectl rollout undo deployment/formfiller-backend -n formfiller
```

## Hibaelhárítás

### Pod-ok Ellenőrzése

```bash
# Pod-ok listázása
kubectl get pods -n formfiller

# Pod részletek
kubectl describe pod <pod-name> -n formfiller

# Pod logok
kubectl logs <pod-name> -n formfiller
```

### Események

```bash
kubectl get events -n formfiller --sort-by='.lastTimestamp'
```

### Debug Konténer

```bash
kubectl run debug --rm -it --image=busybox -n formfiller -- sh
```

### Port Forward (Teszteléshez)

```bash
# Backend
kubectl port-forward service/formfiller-backend 3001:3001 -n formfiller

# MongoDB
kubectl port-forward service/mongodb 27017:27017 -n formfiller
```

## CI/CD Integráció

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Setup Helm
        uses: azure/setup-helm@v3
      
      - name: Configure kubectl
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
      
      - name: Deploy
        run: |
          ./scripts/deploy.sh production \
            --skip-build \
            --version ${{ github.ref_name }}
```

## Biztonsági Megfontolások

### Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: formfiller-network-policy
  namespace: formfiller
spec:
  podSelector:
    matchLabels:
      app: formfiller
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 3001
        - protocol: TCP
          port: 80
```

### Pod Security Standards

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: formfiller
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
```


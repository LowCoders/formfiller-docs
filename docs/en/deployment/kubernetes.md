# Kubernetes Installation

This guide provides assistance for FormFiller Kubernetes installation.

## Prerequisites

- Kubernetes cluster (1.24+)
- Helm 3.x
- kubectl
- Ingress controller (nginx-ingress recommended)

## Quick Start (Minikube)

### Start Minikube

```bash
# Start Minikube
minikube start --memory 4096 --cpus 2

# Ingress addon
minikube addons enable ingress

# Registry addon (for images)
minikube addons enable registry
```

### Installation

```bash
cd formfiller-deployment

# Build images
./scripts/build-images.sh minikube

# Deploy
./scripts/deploy.sh minikube
```

### Access

```bash
# Set hosts file
echo "$(minikube ip) formfiller.local" | sudo tee -a /etc/hosts

# Open application
open http://formfiller.local
```

## Helm Chart

### Chart Structure

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

### Install with Helm

```bash
# Create namespace
kubectl create namespace formfiller

# Create secrets
kubectl create secret generic formfiller-secrets \
  --from-literal=jwt-secret=$(openssl rand -base64 32) \
  --from-literal=session-secret=$(openssl rand -base64 32) \
  --from-literal=mongodb-password=$(openssl rand -base64 16) \
  -n formfiller

# Helm install
helm install formfiller ./helm-chart \
  -n formfiller \
  -f ./helm-chart/values-production.yaml
```

### Helm Upgrade

```bash
helm upgrade formfiller ./helm-chart \
  -n formfiller \
  -f ./helm-chart/values-production.yaml
```

## values.yaml Configuration

### Default Values

```yaml
# Backend configuration
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

# Frontend configuration
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

# MongoDB configuration
mongodb:
  enabled: true
  replicaCount: 1
  persistence:
    enabled: true
    size: 10Gi
    storageClass: standard

# Redis configuration
redis:
  enabled: true
  persistence:
    enabled: true
    size: 1Gi

# Ingress configuration
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

### Production Values

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

## Multisite Deployment

### Deployment Models

FormFiller supports three multisite modes:

#### 1. Shared Backend - Shared DB

```yaml
# values.yaml
multisite:
  enabled: true
  model: shared-backend-shared-db
```

Single backend and database with tenant filtering.

#### 2. Shared Backend - Isolated DB

```yaml
multisite:
  enabled: true
  model: shared-backend-isolated-db
```

Single backend, separate database per tenant.

#### 3. Isolated Backend

```yaml
multisite:
  enabled: true
  model: isolated-backend
  autoProvision: true
```

Separate backend and database pod per tenant.

## Monitoring and Logging

### Prometheus Monitoring

```yaml
# values.yaml
monitoring:
  prometheus:
    enabled: true
    serviceMonitor:
      enabled: true
```

### Logs

```bash
# Backend logs
kubectl logs -f deployment/formfiller-backend -n formfiller

# All pod logs
kubectl logs -f -l app=formfiller -n formfiller
```

## Scaling

### Horizontal Pod Autoscaler

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

### Manual Scaling

```bash
kubectl scale deployment formfiller-backend --replicas=5 -n formfiller
```

## Backup and Restore

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
# View history
helm history formfiller -n formfiller

# Rollback to previous version
helm rollback formfiller -n formfiller

# Rollback to specific version
helm rollback formfiller 3 -n formfiller
```

### Deployment Rollback

```bash
kubectl rollout undo deployment/formfiller-backend -n formfiller
```

## Troubleshooting

### Check Pods

```bash
# List pods
kubectl get pods -n formfiller

# Pod details
kubectl describe pod <pod-name> -n formfiller

# Pod logs
kubectl logs <pod-name> -n formfiller
```

### Events

```bash
kubectl get events -n formfiller --sort-by='.lastTimestamp'
```

### Debug Container

```bash
kubectl run debug --rm -it --image=busybox -n formfiller -- sh
```

### Port Forward (For Testing)

```bash
# Backend
kubectl port-forward service/formfiller-backend 3001:3001 -n formfiller

# MongoDB
kubectl port-forward service/mongodb 27017:27017 -n formfiller
```

## CI/CD Integration

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

## Security Considerations

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


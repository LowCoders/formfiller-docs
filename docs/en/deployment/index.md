# Deployment Documentation

This documentation provides guidance for deploying and operating the FormFiller system.

## Contents

- [Traditional Deployment](./traditional.md) - VPS/server deployment
- [Docker Deployment](./docker.md) - Docker and Docker Compose
- [Kubernetes Deployment](./kubernetes.md) - Kubernetes/Helm deployment

## Deployment Methods Comparison

| Aspect | Traditional | Docker | Kubernetes |
|--------|-------------|--------|------------|
| Complexity | Low | Medium | High |
| Scalability | Limited | Medium | Excellent |
| Resource needs | Low | Medium | High |
| Multisite | Manual | Simple | Automatic |
| Maintenance | Manual | Simple | Automated |

## Prerequisites

### For All Deployment Methods

- MongoDB 4.4+ (external or embedded)
- Domain name (for production)
- SSL certificate (for production)

### For Traditional Deployment

- Linux server (Ubuntu 20.04+ recommended)
- Node.js 18+
- Nginx (reverse proxy)
- PM2 (process manager)

### For Docker Deployment

- Docker 20.10+
- Docker Compose 2.0+

### For Kubernetes Deployment

- Kubernetes cluster (1.24+)
- Helm 3.x
- kubectl
- Ingress controller

## Quick Start

### Docker (Simplest)

```bash
cd formfiller-deployment/docker-compose
cp .env.example .env
# Edit the .env file
docker-compose up -d
```

Available at: `http://localhost:3000`

### Kubernetes (Minikube)

```bash
# Start Minikube
minikube start --memory 4096

# Deploy
cd formfiller-deployment
./scripts/deploy.sh minikube
```

Available at: `http://formfiller.local` (after hosts file configuration)

## Environment Variables

The most important environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `MONGODB_URI` | MongoDB connection string | `mongodb://localhost:27017/formfiller` |
| `JWT_SECRET` | JWT encryption key | (to be generated) |
| `SESSION_SECRET` | Session encryption key | (to be generated) |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID | (optional) |
| `GOOGLE_CLIENT_SECRET` | Google OAuth secret | (optional) |
| `REDIS_URL` | Redis connection string | (optional) |

Full list: See the formfiller-deployment repository (coming soon)

## Multisite Settings

FormFiller supports three multisite modes:

### 1. Shared Backend - Shared DB

```bash
MULTISITE_DEPLOYMENT_MODEL=shared-backend-shared-db
```

Single backend and database with tenant filtering.

### 2. Shared Backend - Isolated DB

```bash
MULTISITE_DEPLOYMENT_MODEL=shared-backend-isolated-db
```

Single backend, but separate database per tenant.

### 3. Isolated Backend

```bash
MULTISITE_DEPLOYMENT_MODEL=isolated-backend
```

Separate backend and database pod per tenant.

## Security Checklist

### Before Production

- [ ] Strong JWT_SECRET and SESSION_SECRET generated
- [ ] SSL/TLS certificate configured
- [ ] MongoDB authentication enabled
- [ ] Rate limiting configured
- [ ] CORS properly configured
- [ ] Firewall rules set
- [ ] Backup strategy implemented
- [ ] Monitoring set up

### Ongoing

- [ ] Regular security updates
- [ ] Log monitoring
- [ ] Backup verification
- [ ] SSL certificate expiration monitoring

## Troubleshooting

### MongoDB Connection Error

1. Check that MongoDB is running
2. Check the `MONGODB_URI` value
3. Check network connectivity

### Frontend Cannot Reach Backend

1. Check the `VITE_API_URL` value
2. Check CORS settings
3. Check reverse proxy configuration

### Pods Not Starting (Kubernetes)

```bash
# Pod status
kubectl get pods -n formfiller-multisite

# Pod details
kubectl describe pod <pod-name> -n formfiller-multisite

# Logs
kubectl logs <pod-name> -n formfiller-multisite
```

## Additional Documentation

- [formfiller-deployment repo](https://github.com/LowCoders/formfiller-deployment) (coming soon)


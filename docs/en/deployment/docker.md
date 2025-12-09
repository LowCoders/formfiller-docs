# Docker Installation

This guide provides assistance for FormFiller Docker and Docker Compose installation.

## Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- Minimum 2GB RAM

## Quick Start

```bash
cd formfiller-deployment/docker-compose
cp .env.example .env
docker-compose up -d
```

Application available at: `http://localhost:3000`

## Docker Compose Configuration

### Default Structure

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}

  backend:
    build: ../../formfiller-backend
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://admin:${MONGO_PASSWORD}@mongodb:27017/formfiller?authSource=admin
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - mongodb

  frontend:
    build: ../../formfiller-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

volumes:
  mongodb_data:
```

## Environment Variables

### .env File

```env
# MongoDB
MONGO_PASSWORD=secure-password-here

# JWT
JWT_SECRET=your-jwt-secret-here

# Session
SESSION_SECRET=your-session-secret-here

# Frontend
VITE_API_URL=http://localhost:3001
```

### Generate Secret Key

```bash
# Linux/macOS
openssl rand -base64 32

# Or
head -c 32 /dev/urandom | base64
```

## Build and Start

### First Start

```bash
# Build
docker-compose build

# Start in background
docker-compose up -d

# Follow logs
docker-compose logs -f
```

### Rebuild

```bash
# After code changes
docker-compose build --no-cache
docker-compose up -d
```

### Stop

```bash
# Stop
docker-compose down

# Stop with data deletion
docker-compose down -v
```

## Production Configuration

### SSL Termination with Nginx

Add an nginx service:

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - backend
```

### nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server backend:3001;
    }

    upstream frontend {
        server frontend:80;
    }

    server {
        listen 80;
        server_name yourdomain.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl;
        server_name yourdomain.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location /api {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location / {
            proxy_pass http://frontend;
        }
    }
}
```

## Redis Cache (Optional)

### Docker Compose Extension

```yaml
services:
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  backend:
    environment:
      - REDIS_ENABLED=true
      - REDIS_URL=redis://redis:6379

volumes:
  redis_data:
```

## Health Check

### Docker Compose Health Check

```yaml
services:
  backend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Monitoring

### Logs

```bash
# All services logs
docker-compose logs -f

# Single service log
docker-compose logs -f backend

# Last 100 lines
docker-compose logs --tail=100
```

### Resource Usage

```bash
docker stats
```

### Container Shell

```bash
# Backend shell
docker-compose exec backend sh

# MongoDB shell
docker-compose exec mongodb mongosh
```

## Backup

### MongoDB Backup

```bash
# Create backup
docker-compose exec mongodb mongodump --out=/backup

# Copy backup to host
docker cp $(docker-compose ps -q mongodb):/backup ./backup
```

### Restore

```bash
# Copy backup to container
docker cp ./backup $(docker-compose ps -q mongodb):/backup

# Restore
docker-compose exec mongodb mongorestore /backup
```

## Updates

### Code Update

```bash
# Update repos
cd formfiller-backend && git pull && cd ..
cd formfiller-frontend && git pull && cd ..

# Rebuild
docker-compose build --no-cache

# Restart
docker-compose up -d
```

### Zero-Downtime Update

```bash
# Build new version
docker-compose build

# Restart services one by one
docker-compose up -d --no-deps backend
docker-compose up -d --no-deps frontend
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs backend

# Container status
docker-compose ps

# Restart
docker-compose restart backend
```

### Network Issues

```bash
# Check network
docker network ls
docker network inspect formfiller_default
```

### Disk Space

```bash
# Remove unused resources
docker system prune -a

# Remove volumes
docker volume prune
```

## Complete Production Example

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7
    restart: always
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    restart: always
    volumes:
      - redis_data:/data

  backend:
    build: 
      context: ../../formfiller-backend
      dockerfile: Dockerfile
    restart: always
    environment:
      - NODE_ENV=production
      - PORT=3001
      - MONGODB_URI=mongodb://admin:${MONGO_PASSWORD}@mongodb:27017/formfiller?authSource=admin
      - JWT_SECRET=${JWT_SECRET}
      - SESSION_SECRET=${SESSION_SECRET}
      - REDIS_ENABLED=true
      - REDIS_URL=redis://redis:6379
      - FRONTEND_URL=${FRONTEND_URL}
    depends_on:
      - mongodb
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build:
      context: ../../formfiller-frontend
      dockerfile: Dockerfile
      args:
        - VITE_API_URL=${API_URL}
    restart: always
    depends_on:
      - backend

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - backend

volumes:
  mongodb_data:
  redis_data:
```


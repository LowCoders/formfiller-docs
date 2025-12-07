# Telepítési Dokumentáció

Ez a dokumentáció a FormFiller rendszer telepítéséhez és üzemeltetéséhez nyújt útmutatást.

## Tartalom

- [Hagyományos Telepítés](./traditional.md) - VPS/szerver telepítés
- [Docker Telepítés](./docker.md) - Docker és Docker Compose
- [Kubernetes Telepítés](./kubernetes.md) - Kubernetes/Helm telepítés

## Telepítési Módok Összehasonlítása

| Szempont | Hagyományos | Docker | Kubernetes |
|----------|-------------|--------|------------|
| Komplexitás | Alacsony | Közepes | Magas |
| Skálázhatóság | Korlátozott | Közepes | Kiváló |
| Erőforrásigény | Alacsony | Közepes | Magas |
| Multisite | Manuális | Egyszerű | Automatikus |
| Karbantartás | Manuális | Egyszerű | Automatizált |

## Előfeltételek

### Minden Telepítési Módhoz

- MongoDB 4.4+ (külső vagy beépített)
- Domain név (éles környezethez)
- SSL tanúsítvány (éles környezethez)

### Hagyományos Telepítéshez

- Linux szerver (Ubuntu 20.04+ ajánlott)
- Node.js 18+
- Nginx (reverse proxy)
- PM2 (process manager)

### Docker Telepítéshez

- Docker 20.10+
- Docker Compose 2.0+

### Kubernetes Telepítéshez

- Kubernetes cluster (1.24+)
- Helm 3.x
- kubectl
- Ingress controller

## Gyors Indítás

### Docker (Legegyszerűbb)

```bash
cd formfiller-deployment/docker-compose
cp .env.example .env
# Szerkeszd a .env fájlt
docker-compose up -d
```

Elérhető: `http://localhost:3000`

### Kubernetes (Minikube)

```bash
# Minikube indítása
minikube start --memory 4096

# Telepítés
cd formfiller-deployment
./scripts/deploy.sh minikube
```

Elérhető: `http://formfiller.local` (hosts fájl beállítás után)

## Környezeti Változók

A legfontosabb környezeti változók:

| Változó | Leírás | Alapértelmezett |
|---------|--------|-----------------|
| `MONGODB_URI` | MongoDB kapcsolati string | `mongodb://localhost:27017/formfiller` |
| `JWT_SECRET` | JWT titkosítási kulcs | (generálandó) |
| `SESSION_SECRET` | Session titkosítási kulcs | (generálandó) |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID | (opcionális) |
| `GOOGLE_CLIENT_SECRET` | Google OAuth secret | (opcionális) |
| `REDIS_URL` | Redis kapcsolati string | (opcionális) |

A teljes lista: [Environment Variables](../../formfiller-deployment/docs/ENVIRONMENT_VARIABLES.md)

## Multisite Beállítások

A FormFiller három multisite módot támogat:

### 1. Shared Backend - Shared DB

```bash
MULTISITE_DEPLOYMENT_MODEL=shared-backend-shared-db
```

Egyetlen backend és adatbázis, tenant szűréssel.

### 2. Shared Backend - Isolated DB

```bash
MULTISITE_DEPLOYMENT_MODEL=shared-backend-isolated-db
```

Egyetlen backend, de tenant-enként külön adatbázis.

### 3. Isolated Backend

```bash
MULTISITE_DEPLOYMENT_MODEL=isolated-backend
```

Tenant-enként külön backend és adatbázis pod.

## Biztonsági Ellenőrzőlista

### Éles Környezet Előtt

- [ ] Erős JWT_SECRET és SESSION_SECRET generálva
- [ ] SSL/TLS tanúsítvány beállítva
- [ ] MongoDB autentikáció engedélyezve
- [ ] Rate limiting beállítva
- [ ] CORS megfelelően konfigurálva
- [ ] Tűzfal szabályok beállítva
- [ ] Backup stratégia implementálva
- [ ] Monitoring beállítva

### Folyamatos

- [ ] Rendszeres biztonsági frissítések
- [ ] Log monitorozás
- [ ] Backup-ok ellenőrzése
- [ ] SSL tanúsítvány lejárat figyelése

## Hibaelhárítás

### MongoDB Kapcsolódási Hiba

1. Ellenőrizd, hogy a MongoDB fut
2. Ellenőrizd a `MONGODB_URI` értékét
3. Ellenőrizd a hálózati kapcsolatot

### Frontend Nem Éri El a Backend-et

1. Ellenőrizd a `VITE_API_URL` értékét
2. Ellenőrizd a CORS beállításokat
3. Ellenőrizd a reverse proxy konfigurációt

### Pod-ok Nem Indulnak (Kubernetes)

```bash
# Pod-ok státusza
kubectl get pods -n formfiller-multisite

# Pod részletek
kubectl describe pod <pod-name> -n formfiller-multisite

# Logok
kubectl logs <pod-name> -n formfiller-multisite
```

## További Dokumentáció

- [formfiller-deployment repo](../../formfiller-deployment)
- [Helm Chart README](../../formfiller-deployment/helm-chart/README.md)
- [CI/CD Guide](../../formfiller-deployment/docs/CI_CD_GUIDE.md)


# Hagyományos Telepítés (VPS)

Ez az útmutató a FormFiller hagyományos szerver telepítéséhez nyújt segítséget.

## Előfeltételek

- Linux szerver (Ubuntu 20.04+ ajánlott)
- Root vagy sudo hozzáférés
- Domain név
- SSL tanúsítvány (Let's Encrypt ajánlott)

## Szerver Előkészítése

### 1. Csomagok Frissítése

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Node.js Telepítése

```bash
# NodeSource repo hozzáadása
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

# Node.js telepítése
sudo apt install -y nodejs

# Ellenőrzés
node --version  # v18.x.x
npm --version
```

### 3. MongoDB Telepítése

```bash
# MongoDB GPG kulcs
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Repository hozzáadása
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] http://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
   sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Telepítés
sudo apt update
sudo apt install -y mongodb-org

# Indítás és engedélyezés
sudo systemctl start mongod
sudo systemctl enable mongod
```

### 4. Nginx Telepítése

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
```

### 5. PM2 Telepítése

```bash
sudo npm install -g pm2
```

## Backend Telepítése

### 1. Kód Letöltése

```bash
cd /var/www
git clone <repo-url>/formfiller-backend
cd formfiller-backend
```

### 2. Függőségek Telepítése

```bash
npm install
```

### 3. Környezeti Változók

```bash
cp env.example .env
nano .env
```

Fontos beállítások:
```env
NODE_ENV=production
PORT=3001
MONGODB_URI=mongodb://localhost:27017/formfiller
JWT_SECRET=<generált-titkos-kulcs>
SESSION_SECRET=<generált-titkos-kulcs>
FRONTEND_URL=https://yourdomain.com
```

Titkos kulcs generálása:
```bash
openssl rand -base64 32
```

### 4. Build és Indítás

```bash
# Build
npm run build

# PM2-vel indítás
pm2 start ecosystem.config.js

# Auto-start beállítása
pm2 startup
pm2 save
```

## Frontend Telepítése

### 1. Kód Letöltése

```bash
cd /var/www
git clone <repo-url>/formfiller-frontend
cd formfiller-frontend
```

### 2. Függőségek és Build

```bash
npm install

# Környezeti változók
echo "VITE_API_URL=https://api.yourdomain.com" > .env.production

# Build
npm run build
```

### 3. Build Fájlok Mozgatása

```bash
sudo mkdir -p /var/www/html/formfiller
sudo cp -r dist/* /var/www/html/formfiller/
```

## Nginx Konfiguráció

### Backend (API)

```bash
sudo nano /etc/nginx/sites-available/formfiller-api
```

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Frontend

```bash
sudo nano /etc/nginx/sites-available/formfiller
```

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    root /var/www/html/formfiller;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### Aktiválás

```bash
sudo ln -s /etc/nginx/sites-available/formfiller /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/formfiller-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## SSL Tanúsítvány (Let's Encrypt)

```bash
# Certbot telepítése
sudo apt install -y certbot python3-certbot-nginx

# Tanúsítvány beszerzése
sudo certbot --nginx -d yourdomain.com -d api.yourdomain.com

# Auto-renewal tesztelése
sudo certbot renew --dry-run
```

## MongoDB Biztonság

### Autentikáció Engedélyezése

```bash
# MongoDB shell
mongosh

# Admin felhasználó létrehozása
use admin
db.createUser({
  user: "admin",
  pwd: "secure-password",
  roles: ["root"]
})

# Alkalmazás felhasználó
use formfiller
db.createUser({
  user: "formfiller",
  pwd: "secure-password",
  roles: ["readWrite"]
})
```

```bash
# MongoDB konfiguráció
sudo nano /etc/mongod.conf
```

```yaml
security:
  authorization: enabled
```

```bash
sudo systemctl restart mongod
```

Frissítsd a `.env` fájlt:
```env
MONGODB_URI=mongodb://formfiller:secure-password@localhost:27017/formfiller
```

## Tűzfal Beállítása

```bash
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

## Monitoring

### PM2 Monitoring

```bash
# Státusz
pm2 status

# Logok
pm2 logs

# Monitoring dashboard
pm2 monit
```

### Nginx Logok

```bash
# Access log
tail -f /var/log/nginx/access.log

# Error log
tail -f /var/log/nginx/error.log
```

## Frissítés

### Backend Frissítése

```bash
cd /var/www/formfiller-backend
git pull
npm install
npm run build
pm2 restart all
```

### Frontend Frissítése

```bash
cd /var/www/formfiller-frontend
git pull
npm install
npm run build
sudo cp -r dist/* /var/www/html/formfiller/
```

## Backup

### MongoDB Backup

```bash
# Backup készítése
mongodump --uri="mongodb://localhost:27017/formfiller" --out=/backup/$(date +%Y%m%d)

# Cron job napi backup-hoz
echo "0 2 * * * mongodump --uri='mongodb://localhost:27017/formfiller' --out=/backup/\$(date +\%Y\%m\%d)" | sudo crontab -
```

### Visszaállítás

```bash
mongorestore --uri="mongodb://localhost:27017/formfiller" /backup/20240101/formfiller
```


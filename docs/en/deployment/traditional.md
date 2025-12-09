# Traditional Installation (VPS)

This guide provides assistance for traditional server installation of FormFiller.

## Prerequisites

- Linux server (Ubuntu 20.04+ recommended)
- Root or sudo access
- Domain name
- SSL certificate (Let's Encrypt recommended)

## Server Preparation

### 1. Update Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Node.js

```bash
# Add NodeSource repo
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify
node --version  # v18.x.x
npm --version
```

### 3. Install MongoDB

```bash
# MongoDB GPG key
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Add repository
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] http://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
   sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Install
sudo apt update
sudo apt install -y mongodb-org

# Start and enable
sudo systemctl start mongod
sudo systemctl enable mongod
```

### 4. Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
```

### 5. Install PM2

```bash
sudo npm install -g pm2
```

## Backend Installation

### 1. Download Code

```bash
cd /var/www
git clone <repo-url>/formfiller-backend
cd formfiller-backend
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Environment Variables

```bash
cp env.example .env
nano .env
```

Important settings:
```env
NODE_ENV=production
PORT=3001
MONGODB_URI=mongodb://localhost:27017/formfiller
JWT_SECRET=<generated-secret-key>
SESSION_SECRET=<generated-secret-key>
FRONTEND_URL=https://yourdomain.com
```

Generate secret key:
```bash
openssl rand -base64 32
```

### 4. Build and Start

```bash
# Build
npm run build

# Start with PM2
pm2 start ecosystem.config.js

# Setup auto-start
pm2 startup
pm2 save
```

## Frontend Installation

### 1. Download Code

```bash
cd /var/www
git clone <repo-url>/formfiller-frontend
cd formfiller-frontend
```

### 2. Dependencies and Build

```bash
npm install

# Environment variables
echo "VITE_API_URL=https://api.yourdomain.com" > .env.production

# Build
npm run build
```

### 3. Move Build Files

```bash
sudo mkdir -p /var/www/html/formfiller
sudo cp -r dist/* /var/www/html/formfiller/
```

## Nginx Configuration

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

### Activation

```bash
sudo ln -s /etc/nginx/sites-available/formfiller /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/formfiller-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## SSL Certificate (Let's Encrypt)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d yourdomain.com -d api.yourdomain.com

# Test auto-renewal
sudo certbot renew --dry-run
```

## MongoDB Security

### Enable Authentication

```bash
# MongoDB shell
mongosh

# Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "secure-password",
  roles: ["root"]
})

# Application user
use formfiller
db.createUser({
  user: "formfiller",
  pwd: "secure-password",
  roles: ["readWrite"]
})
```

```bash
# MongoDB configuration
sudo nano /etc/mongod.conf
```

```yaml
security:
  authorization: enabled
```

```bash
sudo systemctl restart mongod
```

Update the `.env` file:
```env
MONGODB_URI=mongodb://formfiller:secure-password@localhost:27017/formfiller
```

## Firewall Setup

```bash
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

## Monitoring

### PM2 Monitoring

```bash
# Status
pm2 status

# Logs
pm2 logs

# Monitoring dashboard
pm2 monit
```

### Nginx Logs

```bash
# Access log
tail -f /var/log/nginx/access.log

# Error log
tail -f /var/log/nginx/error.log
```

## Updates

### Backend Update

```bash
cd /var/www/formfiller-backend
git pull
npm install
npm run build
pm2 restart all
```

### Frontend Update

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
# Create backup
mongodump --uri="mongodb://localhost:27017/formfiller" --out=/backup/$(date +%Y%m%d)

# Cron job for daily backup
echo "0 2 * * * mongodump --uri='mongodb://localhost:27017/formfiller' --out=/backup/\$(date +\%Y\%m\%d)" | sudo crontab -
```

### Restore

```bash
mongorestore --uri="mongodb://localhost:27017/formfiller" /backup/20240101/formfiller
```


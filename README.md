# Deploying NestJS & Next.js on a VPS

## Prerequisites

- A VPS server (Ubuntu 22.04 recommended)
- SSH access to your VPS
- A registered domain (optional but recommended)
- A database (MongoDB, PostgreSQL, MySQL, etc.)
- A process manager like `PM2` to keep the apps running

---

## Step 1: Connect to the VPS

```sh
ssh root@your-vps-ip
```

Replace `your-vps-ip` with your actual VPS IP.

---

## Step 2: Update and Install Required Packages

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx curl git unzip
```

---

## Step 3: Install Node.js and PM2

```sh
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
npm install -g pm2
```

---

## Step 4: Set Up Your Backend (NestJS)

### Clone Your NestJS Repo
```sh
cd /var/www
git clone https://github.com/your-repo/nestjs-backend.git
cd nestjs-backend
```

### Install Dependencies
```sh
npm install
```

### Configure the `.env` File
```sh
cp .env.example .env
nano .env
```

### Build and Run the App
```sh
npm run build
pm2 start dist/main.js --name nestjs-backend
pm2 save
pm2 startup
```

---

## Step 5: Set Up Your Frontend (Next.js)

### Clone Your Next.js Repo
```sh
cd /var/www
git clone https://github.com/your-repo/nextjs-frontend.git
cd nextjs-frontend
```

### Install Dependencies
```sh
npm install
```

### Configure the `.env` File
```sh
cp .env.example .env
nano .env
```

### Build and Start Next.js
```sh
npm run build
pm2 start npm --name "nextjs-frontend" -- start
pm2 save
```

---

## Step 6: Configure Nginx as a Reverse Proxy

### Install Nginx
```sh
sudo apt install nginx -y
```

### Configure Nginx for NestJS (Backend)
```sh
sudo nano /etc/nginx/sites-available/nestjs
```

**Add the following configuration:**
```
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Save and exit.

### Configure Nginx for Next.js (Frontend)
```sh
sudo nano /etc/nginx/sites-available/nextjs
```

**Add the following configuration:**
```
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Save and exit.

### Enable the Nginx Configurations
```sh
sudo ln -s /etc/nginx/sites-available/nestjs /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/nextjs /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---

## Step 7: Secure Your Server with SSL (HTTPS)

Install **Let's Encrypt** and generate SSL certificates:
```sh
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
sudo certbot --nginx -d api.yourdomain.com
```

Auto-renew SSL:
```sh
sudo crontab -e
```
Add this line at the bottom:
```
0 0 * * * certbot renew --quiet
```

---

## Step 8: Set Up a Database (If Not Done)

If using **MongoDB**, install and start it:
```sh
sudo apt install mongodb -y
sudo systemctl start mongodb
sudo systemctl enable mongodb
```

If using **PostgreSQL**:
```sh
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

---

## Step 9: Monitor and Keep Apps Running

Check running services:
```sh
pm2 list
```

Restart a service:
```sh
pm2 restart nestjs-backend
pm2 restart nextjs-frontend
```

---

## 🎉 Deployment Completed!

### Access your applications:
- **Backend (NestJS)** → `https://api.yourdomain.com`
- **Frontend (Next.js)** → `https://yourdomain.com`

Let me know if you need any modifications! 🚀


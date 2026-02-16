# How to deploy Mern in fresh EC2 instance with Linux OS

## 1️⃣ Connect to EC2
Command to connect ec2 instance using sssh remember os should be ubuntu only then this works because we are using ubuntu as a user.
```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

## 2️⃣ Update the system (FIRST command always)
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3️⃣ Install Node.js (recommended: NodeSource)
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
- Verify
```bash
node -v
npm -v
```
If `npm -v` command not works then `sudo apt install npm` run this command.

---

## 4️⃣ Install Git
- First by `git -v` this command if exists then don't don't this command if not then insatll.
```basah
sudo apt install git -y
```

## 5️⃣ Clone your project
```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

---

## 6️⃣ Install dependencies
```bash
npm install
```

---

## 7️⃣ Set environment variables
- Option A: .env file
```bash
nano .env
```
```env
PORT=3000
DB_URL=your_db_url
NODE_ENV=production
```

---

# Deploy React app
### (1) Follow Above commands
path - /var/www/frontend/ to deploy code or build
```swift
/var/www/frontend/
 ├── dist/
 │   ├── index.html
 │   ├── assets/
```
### (2) Build React app
```bash
npm run build
```
---
### (3) Install Nginx
```bash
sudo apt install nginx -y
nginx -v
```
---
### (4) Create web directory
```bash
sudo mkdir -p /var/www/frontend
```
---
### (5) Copy build files
```bash
sudo cp -r build/* /var/www/frontend/
```
---
### (6) Set permissions
```bash
sudo chown -R www-data:www-data /var/www/frontend
sudo chmod -R 755 /var/www/frontend
```
---
### (7) Create Nginx config
```bash
sudo nano /etc/nginx/sites-available/frontend
```
```nginx
server {
    listen 80;
    server_name _;

    root /var/www/frontend;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
Save → Exit (Using clt+o - to save, enter, then cltr+x - to exit)
```
---
### (8) Enable site
```bash
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```
---
### (9) Test Nginx config
```bash
sudo nginx -t
```
---
### (10) Reload Nginx
```bash
sudo systemctl reload nginx
```
---
### ✅ Application Live
```bash
http://<SERVER_PUBLIC_IP>
```
---
### Redeploy (after code change)
```bash
npm run build
sudo rm -rf /var/www/frontend/*
sudo cp -r build/* /var/www/frontend/
sudo systemctl reload nginx
```
---
### 🧠 Architecture
```text
Browser → Nginx → React Build (Static Files)
```
---

# 🚀 Deploy React as a Server using PM2

This guide explains **step-by-step deployment of a React application as a running Node.js server**, managed by **PM2**.
⚠️ This is **NOT a static build deployment**. React runs as a live server process.

---

## 🧱 Architecture Overview

```
Browser
  ↓
Nginx (Port 80)
  ↓
Node.js Server (Port 3000) – PM2
  ↓
React (Vite Dev Server – Port 5173)
```

---

## ✅ Requirements

* Ubuntu 20.04+ VPS / EC2
* Root or sudo access
* Domain or server IP
* Git installed

---

## 1️⃣ Server Setup

### Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Node.js (LTS)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify:

```bash
node -v
npm -v
```

---

## 2️⃣ Install PM2

```bash
sudo npm install -g pm2
```

---

## 3️⃣ Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## 4️⃣ Create Application Directory

```bash
sudo mkdir -p /var/www/react-server
sudo chown -R $USER:$USER /var/www/react-server
cd /var/www/react-server
```

---

## 5️⃣ Create React App (Vite)

```bash
npm create vite@latest .
```

Choose:

* Framework: **React**
* Variant: **JavaScript**

Install dependencies:

```bash
npm install
```

---

## 6️⃣ Add Node Server (Required)

Create `server.js`:

```js
import express from "express";
import { createProxyMiddleware } from "http-proxy-middleware";

const app = express();
const PORT = 3000;

app.use(
  "/",
  createProxyMiddleware({
    target: "http://localhost:5173",
    changeOrigin: true,
    ws: true
  })
);

app.listen(PORT, () => {
  console.log(`React server running on port ${PORT}`);
});
```

---

## 7️⃣ Update `package.json`

```json
{
  "type": "module",
  "scripts": {
    "dev": "vite",
    "server": "node server.js"
  }
}
```

---

## 8️⃣ Start Application with PM2

### Start React (Vite)

```bash
pm2 start npm --name react-ui -- run dev
```

### Start Node Server

```bash
pm2 start server.js --name react-server
```

Persist on reboot:

```bash
pm2 save
pm2 startup
```

---

## 9️⃣ Configure Nginx (Public Access)

Create config:

```bash
sudo nano /etc/nginx/sites-available/react
```

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_IP;

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

Enable config:

```bash
sudo ln -s /etc/nginx/sites-available/react /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔗 Access Application

```
http://YOUR_SERVER_IP
```

You should see:

```
Hello World
```

---

## 🔄 GitHub Integration (Manual)

```bash
git init
git remote add origin YOUR_REPO_URL
git pull origin main
```

After updates:

```bash
git pull
pm2 restart all
```

---

## 📊 PM2 Useful Commands

```bash
pm2 list
pm2 logs
pm2 restart react-ui
pm2 restart react-server
pm2 stop all
```

---

## ⚠️ Production Note (Important)

Running React in **dev mode** is suitable for:

* Admin panels
* Internal tools
* MVPs

For high-traffic production apps:

* Use **Next.js + PM2**
* Or **Vite SSR**
* Or static build with Nginx

---

## ✅ Deployment Summary

✔ React running as server
✔ Managed by PM2
✔ Reverse proxied by Nginx
✔ GitHub compatible
✔ No static build

---

**Next options available:**

* PM2 `ecosystem.config.js`
* GitHub auto-deploy (webhook)
* HTTPS with SSL
* Convert to Next.js server

Just say **next** 🚀
---
---

# 🚀 Deploy Next.js as a Server using PM2

This document explains **how to deploy a Next.js application as a running Node.js server**, managed by **PM2**, behind **Nginx**.

✅ This is the **recommended production approach** for React-based apps
❌ This is **NOT static export**

---

## 🧱 Architecture Overview

```
Browser
  ↓
Nginx (Port 80 / 443)
  ↓
Next.js Server (Port 3000) — PM2
```

---

## ✅ Requirements

* Ubuntu 20.04+ VPS / EC2
* Node.js 18+ or 20+
* Git
* Domain or Server IP
* sudo access

---

## 1️⃣ Server Setup

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Node.js (LTS)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify:

```bash
node -v
npm -v
```

---

## 2️⃣ Install PM2

```bash
sudo npm install -g pm2
```

---

## 3️⃣ Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## 4️⃣ Create Project Directory

```bash
sudo mkdir -p /var/www/nextjs-app
sudo chown -R $USER:$USER /var/www/nextjs-app
cd /var/www/nextjs-app
```

---

## 5️⃣ Create Next.js App

```bash
npx create-next-app@latest .
```

Recommended options:

* TypeScript: optional
* App Router: ✅ Yes
* ESLint: optional
* Tailwind: optional

Install dependencies (if prompted):

```bash
npm install
```

---

## 6️⃣ Build Next.js (Required for Production)

```bash
npm run build
```

This creates the `.next/` production build.

---

## 7️⃣ Start Next.js with PM2

### Run Next.js in production mode

```bash
pm2 start npm --name nextjs-app -- start
```

Persist after reboot:

```bash
pm2 save
pm2 startup
```

---

## 8️⃣ Verify Local Server

```bash
curl http://localhost:3000
```

You should see the Next.js homepage HTML.

---

## 9️⃣ Configure Nginx (Reverse Proxy)

Create config:

```bash
sudo nano /etc/nginx/sites-available/nextjs
```

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_IP;

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

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/nextjs /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🌍 Access Application

```
http://YOUR_DOMAIN_OR_IP
```

---

## 🔁 GitHub Deployment Flow (Manual)

```bash
git init
git remote add origin YOUR_REPO_URL
git pull origin main
```

After code changes:

```bash
git pull
npm install
npm run build
pm2 restart nextjs-app
```

---

## 📊 Useful PM2 Commands

```bash
pm2 list
pm2 logs nextjs-app
pm2 restart nextjs-app
pm2 stop nextjs-app
```

---

## 🔐 Enable HTTPS (Recommended)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com
```

Auto-renew:

```bash
sudo certbot renew --dry-run
```

---

## ⚠️ Production Notes (Important)

✔ Next.js **must always be built** before restart
✔ PM2 runs **Node server**, not static files
✔ Supports:

* SSR
* API routes
* Middleware
* Authentication

❌ Do NOT use `next dev` in production

---

## ✅ Deployment Checklist

✔ Next.js running as Node server
✔ Managed by PM2
✔ Reverse proxied via Nginx
✔ GitHub-ready
✔ Production-safe

---

## 🚀 Next Steps (Optional)

* PM2 `ecosystem.config.js`
* Zero-downtime reload
* GitHub webhook auto-deploy
* Dockerize Next.js
* CI/CD with GitHub Actions

Just tell me **next** and what you want 🔥





---

### Steps to install `nginx` in server.
```bash
sudo apt update && sudo apt install nginx -y
# OR You can run one by one
sudo apt update
sudo apt install nginx -y
```
Verify installation
```bash
sudo systemctl status nginx
```
Start / enable Nginx (if needed)
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
Test in browser
```bash
http://<EC2_PUBLIC_IP>
```
Now Add script to reverse from `IP 80 port To localhost 3000`.
- Create reverse proxy config (THIS IS THE KEY)
```bash
sudo nano /etc/nginx/sites-available/backend
```
Paste this 👇 -> this could be change according to url and port so always check it
```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save → Exit (Using clt+o - to save, enter, then cltr+x - to exit)

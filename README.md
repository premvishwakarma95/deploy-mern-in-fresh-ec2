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
Paste this 👇
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

# 🚀 MERN Stack App Deployment on AWS  
**Deploy Node.js + React App using EC2, PM2, NGINX & Free SSL (Let’s Encrypt)**

[![YouTube Demo](https://img.youtube.com/vi/FvwRSpmVYyw/maxresdefault.jpg)](https://youtu.be/FvwRSpmVYyw)

> 🎥 *Click the image above to watch the full deployment walkthrough video!*

---

## 🛠️ Table of Contents

- 🔹 [Introduction](#introduction)
- 📦 [Tech Stack](#tech-stack)
- 🧱 [Prerequisites](#prerequisites)
- ☁️ [Step 1 — AWS Setup](#step-1---aws-setup)
- 💻 [Step 2 — Connect to EC2](#step-2---connect-to-ec2)
- 🔄 [Step 3 — Update Server](#step-3---update-server)
- 🔧 [Step 4 — Install Node.js](#step-4---install-nodejs)
- 📁 [Step 5 — Clone & Install App](#step-5---clone--install-app)
- 🚀 [Step 6 — PM2 Setup](#step-6---pm2-setup)
- 🌐 [Step 7 — Install & Configure NGINX](#step-7---install--configure-nginx)
- 🌍 [Step 8 — Connect Domain to EC2](#step-8---connect-domain-to-ec2)
- 🔐 [Step 9 — Enable HTTPS with Let’s Encrypt](#step-9---enable-https-with-lets-encrypt)
- 🔁 [Step 10 — SSL Auto Renewal](#step-10---ssl-auto-renewal)
- 🛡️ [Production Best Practices](#production-best-practices)
- 🎉 [Deployment Complete](#deployment-complete)

---

## 🧠 Introduction

This guide helps you deploy a **MERN (MongoDB, Express, React, Node.js)** application to a **production-ready server** using:

✔️ AWS EC2 (Ubuntu)  
✔️ PM2 (Process Manager)  
✔️ NGINX (Reverse Proxy)  
✔️ Let’s Encrypt (Free SSL Certificate)

---

## 📦 Tech Stack

- MongoDB
- Express.js
- React.js
- Node.js
- AWS EC2
- PM2
- NGINX
- Let’s Encrypt

---

## 🧾 Prerequisites

Make sure you have:

✔️ AWS Account  
✔️ Domain Name  
✔️ GitHub repository of your MERN project  
✔️ Basic terminal knowledge  

---

# ☁️ Step 1 — AWS Setup

1. Go to https://aws.amazon.com/
2. Open **EC2 Dashboard**
3. Click **Launch Instance**
4. Select:
   - Ubuntu 22.04 LTS
   - Instance type: `t2.medium` (or `t2.micro` for free tier)
5. Create a Key Pair (.pem file)
6. Allow these ports:
   - 22 (SSH)
   - 80 (HTTP)
   - 443 (HTTPS)

---

# 💻 Step 2 — Connect to EC2

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

# 🔄 Step 3 — Update Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 🔧 Step 4 — Install Node.js

```bash
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y

node -v
npm -v
```

---

# 📁 Step 5 — Clone & Install App

```bash
git clone https://github.com/yourusername/yourrepo.git
cd yourrepo
npm install
```

If backend is inside a folder:

```bash
cd backend
npm install
```

---

# 🚀 Step 6 — PM2 Setup

Install PM2 globally:

```bash
sudo npm install pm2 -g
```

Start your app:

```bash
pm2 start index.js
```

Check running apps:

```bash
pm2 list
```

Enable auto-start on reboot:

```bash
pm2 startup
pm2 save
```

---

# 🌐 Step 7 — Install & Configure NGINX

Install NGINX:

```bash
sudo apt install nginx -y
```

Edit config:

```bash
sudo nano /etc/nginx/sites-available/default
```

Replace with:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Test configuration:

```bash
sudo nginx -t
```

Restart NGINX:

```bash
sudo systemctl restart nginx
```

---

# 🌍 Step 8 — Connect Domain to EC2

Go to your Domain Provider → DNS Settings

Add an A Record:

| Type | Name | Value |
|------|------|-------|
| A | @ | EC2_PUBLIC_IP |

Wait 5–10 minutes for DNS propagation.

---

# 🔐 Step 9 — Enable HTTPS with Let’s Encrypt

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Generate SSL:

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Choose redirect to HTTPS when prompted.

---

# 🔁 Step 10 — SSL Auto Renewal

```bash
sudo certbot renew --dry-run
```

Let’s Encrypt certificates renew automatically every 90 days.

---

# 🛡️ Production Best Practices

### Enable Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### Use Environment Variables

Create `.env` file:

```
PORT=5000
MONGO_URI=your_mongo_connection_string
JWT_SECRET=your_secret_key
```

Never push `.env` to GitHub.

---

# 🎉 Deployment Complete

Now visit:

```
https://yourdomain.com
```

Your MERN app is live 🚀

---

## ⭐ If This Helped You

Give this repository a ⭐ star!

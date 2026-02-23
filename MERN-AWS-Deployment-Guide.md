# 🚀 MERN Stack App Deployment on AWS

**Deploy Node.js + React App using EC2, PM2, NGINX & Free SSL (Let's Encrypt)**

---

## 🛠️ Table of Contents

- 🔹 [Introduction](#introduction)
- 📦 [Tech Stack](#tech-stack)
- 🧱 [Prerequisites](#prerequisites)
- ☁️ [Step 1 — AWS Setup](#step-1--aws-setup)
- 💻 [Step 2 — Connect to EC2](#step-2--connect-to-ec2)
- 🔄 [Step 3 — Update Server](#step-3--update-server)
- 🔧 [Step 4 — Install Node.js](#step-4--install-nodejs)
- 📁 [Step 5 — Clone & Install App](#step-5--clone--install-app)
- 🚀 [Step 6 — PM2 Setup](#step-6--pm2-setup)
- 🌐 [Step 7 — Install & Configure NGINX](#step-7--install--configure-nginx)
- 🌍 [Step 8 — Connect Domain to EC2](#step-8--connect-domain-to-ec2)
- 🔐 [Step 9 — Enable HTTPS with Let's Encrypt](#step-9--enable-https-with-lets-encrypt)
- 🔁 [Step 10 — SSL Auto Renewal](#step-10--ssl-auto-renewal)
- 🛡️ [Production Best Practices](#production-best-practices)
- 🔧 [Troubleshooting](#troubleshooting)
- 📬 [Support](#support)

---

## 🧠 Introduction

This guide helps you deploy a **MERN (MongoDB, Express, React, Node.js)** application to a **production-ready server** using:

✔️ AWS EC2 (Ubuntu 24.04 LTS)  
✔️ PM2 (Process Manager)  
✔️ NGINX (Reverse Proxy)  
✔️ Let's Encrypt (Free SSL Certificate)

---

## 📦 Tech Stack

- MongoDB (Atlas or Self-hosted)
- Express.js
- React.js
- Node.js (v20 LTS)
- AWS EC2
- PM2
- NGINX
- Let's Encrypt (Certbot)

---

## 🧾 Prerequisites

Make sure you have:

✔️ AWS Account  
✔️ Domain Name (from providers like Namecheap, GoDaddy, Cloudflare)  
✔️ GitHub repository of your MERN project  
✔️ Basic terminal/command line knowledge  
✔️ MongoDB database (MongoDB Atlas recommended)

---

# ☁️ Step 1 — AWS Setup

1. Go to https://aws.amazon.com/
2. Open **EC2 Dashboard**
3. Click **Launch Instance**
4. Configure:
   - **Name**: `mern-production-server`
   - **AMI**: Ubuntu Server 24.04 LTS (or 22.04 LTS)
   - **Instance type**:
     - `t2.medium` (recommended for production - 2 vCPU, 4GB RAM)
     - `t2.micro` (free tier eligible - 1 vCPU, 1GB RAM - for testing only)
     - `t3.small` (better performance - 2 vCPU, 2GB RAM)
5. **Key Pair**:
   - Click "Create new key pair"
   - Name: `mern-app-key`
   - Type: RSA
   - Format: `.pem` (for Mac/Linux) or `.ppk` (for Windows with PuTTY)
   - Download and save securely
6. **Network Settings** - Configure Security Group:
   - SSH (22) - Your IP only (for security)
   - HTTP (80) - 0.0.0.0/0 (anywhere)
   - HTTPS (443) - 0.0.0.0/0 (anywhere)
   - Custom TCP (3000-5000) - 0.0.0.0/0 (for development, remove in production)
7. **Storage**: 20-30 GB gp3 (recommended)
8. Click **Launch Instance**
9. Note down your **Public IPv4 address**

---

# 💻 Step 2 — Connect to EC2

### For Mac/Linux:

```bash
# Navigate to where your .pem file is located
cd ~/Downloads

# Set correct permissions
chmod 400 mern-app-key.pem

# Connect to EC2
ssh -i mern-app-key.pem ubuntu@<EC2_PUBLIC_IP>
```

### For Windows (using Git Bash or WSL):

```bash
# Same commands as Mac/Linux
chmod 400 mern-app-key.pem
ssh -i mern-app-key.pem ubuntu@<EC2_PUBLIC_IP>
```

### For Windows (using PuTTY):

1. Convert `.pem` to `.ppk` using PuTTYgen
2. Open PuTTY
3. Host: `ubuntu@<EC2_PUBLIC_IP>`
4. Connection → SSH → Auth → Browse to `.ppk` file
5. Click Open

---

# 🔄 Step 3 — Update Server

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential curl git

# Clean up
sudo apt autoremove -y
sudo apt autoclean

# Verify system
lsb_release -a
```

---

# 🔧 Step 4 — Install Node.js

### Install Node.js v20 LTS (Recommended for 2025):

```bash
# Install NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Load NVM
source ~/.bashrc

# Install Node.js LTS
nvm install 20
nvm use 20
nvm alias default 20

# Verify installation
node -v    # Should show v20.x.x
npm -v     # Should show v10.x.x
```

### Alternative: Install via NodeSource:

```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify
node -v
npm -v
```

---

# 📁 Step 5 — Clone & Install App

```bash
# Navigate to home directory
cd ~

# Clone your repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

# Install backend dependencies
npm install

# Build React frontend (if not using separate deployment)
cd client
npm install
npm run build

# Go back to root
cd ..
```

### Configure Environment Variables:

```bash
# Create .env file
nano .env
```

Add your configuration:

```env
NODE_ENV=production
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_super_secret_key
CLIENT_URL=https://yourdomain.com
```

Save: `Ctrl + X`, then `Y`, then `Enter`

### Test your application:

```bash
# Start the server
npm start

# Or if you have a different start command
node server.js
```

Visit `http://<EC2_PUBLIC_IP>:5000` in your browser.

If it works, press `Ctrl + C` to stop.

---

# 🚀 Step 6 — PM2 Setup

PM2 keeps your Node.js app running 24/7, auto-restarts on crashes, and handles logs.

```bash
# Install PM2 globally
sudo npm install -g pm2

# Start your app with PM2
pm2 start server.js --name "mern-app"

# Or if you use npm start
pm2 start npm --name "mern-app" -- start

# Save PM2 configuration
pm2 save

# Setup PM2 to start on boot
pm2 startup systemd

# Copy and run the command PM2 outputs (usually starts with sudo)
# Example: sudo env PATH=$PATH:/usr/bin...

# Useful PM2 Commands
pm2 status              # Check status
pm2 logs mern-app       # View logs
pm2 restart mern-app    # Restart app
pm2 stop mern-app       # Stop app
pm2 delete mern-app     # Delete app from PM2
pm2 monit               # Monitor resources
```

---

# 🌐 Step 7 — Install & Configure NGINX

NGINX acts as a reverse proxy, handling incoming requests and forwarding them to your Node.js app.

```bash
# Install NGINX
sudo apt install nginx -y

# Check NGINX status
sudo systemctl status nginx

# Start NGINX (if not running)
sudo systemctl start nginx

# Enable NGINX to start on boot
sudo systemctl enable nginx
```

### Configure NGINX:

```bash
# Remove default configuration
sudo rm /etc/nginx/sites-enabled/default

# Create new configuration
sudo nano /etc/nginx/sites-available/mern-app
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Increase upload size
    client_max_body_size 100M;

    # React app (if serving static build)
    location / {
        root /home/ubuntu/YOUR_REPO/client/build;
        try_files $uri $uri/ /index.html;
    }

   # API Routes
   # Make sure to replace the PORT with the port your backend application is running on.
   # Example: If your backend runs on PORT=5000, use http://localhost:5000
    location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket support (if needed)
    location /socket.io {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

**Note**: Replace `yourdomain.com` with your actual domain and adjust paths as needed.

```bash
# Create symbolic link to enable site
sudo ln -s /etc/nginx/sites-available/mern-app /etc/nginx/sites-enabled/

# Test NGINX configuration
sudo nginx -t

# Reload NGINX
sudo systemctl reload nginx
```

---

# 🌍 Step 8 — Connect Domain to EC2

### Option A: Using AWS Route 53

1. Go to Route 53 in AWS Console
2. Create Hosted Zone for your domain
3. Create A Record pointing to EC2 Public IP
4. Update nameservers at your domain registrar

### Option B: Using External DNS Provider (Namecheap, GoDaddy, etc.)

1. Login to your domain provider
2. Go to DNS Management
3. Add/Edit A Record:
   - **Type**: A
   - **Host**: @ (for root domain)
   - **Value**: Your EC2 Public IP
   - **TTL**: Automatic or 3600
4. Add another A Record for www:
   - **Type**: A
   - **Host**: www
   - **Value**: Your EC2 Public IP
   - **TTL**: Automatic or 3600

**Wait 15-60 minutes** for DNS propagation.

Verify with:

```bash
# Check DNS propagation
nslookup yourdomain.com
dig yourdomain.com
```

---

# 🔐 Step 9 — Enable HTTPS with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Follow the prompts:
# 1. Enter email address
# 2. Agree to terms
# 3. Choose whether to redirect HTTP to HTTPS (recommended: Yes)

# Verify certificate
sudo certbot certificates
```

Your site should now be accessible via `https://yourdomain.com` 🎉

---

# 🔁 Step 10 — SSL Auto Renewal

Let's Encrypt certificates expire every 90 days. Set up auto-renewal:

```bash
# Test renewal process
sudo certbot renew --dry-run

# Certbot automatically creates a cron job, verify it:
sudo systemctl status certbot.timer

# Or manually add to crontab (if needed)
sudo crontab -e

# Add this line:
0 0 * * * certbot renew --quiet --post-hook "systemctl reload nginx"
```

---

# 🛡️ Production Best Practices

### 1. Security Hardening

```bash
# Setup UFW Firewall
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status

# Disable root login
sudo nano /etc/ssh/sshd_config
# Set: PermitRootLogin no
sudo systemctl restart sshd
```

### 2. Enable NGINX Gzip Compression

```bash
sudo nano /etc/nginx/nginx.conf
```

Add inside `http` block:

```nginx
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/json;
```

```bash
sudo systemctl reload nginx
```

### 3. Setup Log Rotation

```bash
# PM2 logs
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

### 4. Enable Monitoring

```bash
# Install htop
sudo apt install htop -y

# Monitor resources
htop

# Check disk usage
df -h

# Check memory
free -h
```

### 5. Database Security (MongoDB)

- Use MongoDB Atlas with IP whitelist
- Strong passwords
- Enable authentication
- Use environment variables for credentials

### 6. Regular Backups

```bash
# Create backup script
nano ~/backup.sh
```

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/home/ubuntu/backups"
mkdir -p $BACKUP_DIR

# Backup application
tar -czf $BACKUP_DIR/app_backup_$DATE.tar.gz /home/ubuntu/YOUR_REPO

# Remove backups older than 7 days
find $BACKUP_DIR -name "app_backup_*.tar.gz" -mtime +7 -delete
```

```bash
chmod +x ~/backup.sh

# Add to crontab (daily at 2 AM)
crontab -e
# Add: 0 2 * * * /home/ubuntu/backup.sh
```

### 7. Environment-Specific Configurations

```javascript
// In your Node.js app
if (process.env.NODE_ENV === "production") {
  // Production-specific configs
  app.use(helmet());
  app.use(compression());
  // Disable detailed error messages
}
```

---

# 🔧 Troubleshooting

### NGINX 502 Bad Gateway

```bash
# Check if Node.js app is running
pm2 status

# Check NGINX error logs
sudo tail -f /var/log/nginx/error.log

# Restart services
pm2 restart mern-app
sudo systemctl restart nginx
```

### Port Already in Use

```bash
# Find process using port 5000
sudo lsof -i :5000

# Kill the process
sudo kill -9 <PID>

# Restart PM2
pm2 restart mern-app
```

### SSL Certificate Issues

```bash
# Renew certificate manually
sudo certbot renew

# Check certificate expiration
sudo certbot certificates

# Force renewal
sudo certbot renew --force-renewal
```

### Application Crashes

```bash
# View PM2 logs
pm2 logs mern-app --lines 100

# Check for errors
pm2 monit

# Restart with fresh logs
pm2 restart mern-app
pm2 flush
```

### DNS Not Resolving

```bash
# Check DNS propagation
dig yourdomain.com
nslookup yourdomain.com

# Flush DNS cache (on local machine)
# Mac: sudo dscacheutil -flushcache
# Windows: ipconfig /flushdns
# Linux: sudo systemd-resolve --flush-caches
```

---

# 📬 Support

### Useful Resources:

- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [NGINX Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt](https://letsencrypt.org/)
- [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/)

### Common Commands Reference:

```bash
# PM2
pm2 list                    # List all processes
pm2 restart all             # Restart all apps
pm2 logs                    # View all logs
pm2 delete all              # Delete all processes

# NGINX
sudo systemctl status nginx # Check status
sudo systemctl restart nginx # Restart
sudo nginx -t               # Test config

# System
df -h                       # Disk usage
free -h                     # Memory usage
top                         # Process monitor
```

---

## 🎉 Deployment Complete!

Your MERN stack application is now:

- ✅ Running on AWS EC2
- ✅ Managed by PM2 (auto-restart)
- ✅ Served through NGINX
- ✅ Secured with HTTPS (Let's Encrypt)
- ✅ Production-ready!

**Access your app at**: `https://yourdomain.com`

---

Made with ❤️ for the developer community

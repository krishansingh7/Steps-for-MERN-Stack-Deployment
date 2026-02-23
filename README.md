🚀 Deploy MERN Stack App on AWS (EC2 + PM2 + NGINX + SSL)

A complete beginner-friendly step-by-step guide to deploy a MERN Stack Application on AWS using:

🖥 EC2 (Ubuntu)

⚙ PM2 (Process Manager)

🌐 NGINX (Reverse Proxy)

🔐 Let’s Encrypt (Free SSL)

📌 Tech Stack

MongoDB

Express.js

React.js

Node.js

AWS EC2

PM2

NGINX

Let’s Encrypt

🧱 Deployment Architecture
User → Domain → NGINX → Node.js App (PM2) → MongoDB

NGINX handles incoming traffic

PM2 keeps Node app running

SSL secures your website (HTTPS)

✅ Step 1: Create AWS Account

Create a free account:
👉 https://aws.amazon.com/

✅ Step 2: Launch EC2 Instance

Open AWS Dashboard → EC2

Click Launch Instance

Choose:

Ubuntu 22.04 LTS

Instance type: t2.medium

Create a Key Pair (.pem file)

Allow ports:

22 (SSH)

80 (HTTP)

443 (HTTPS)

✅ Step 3: Connect to EC2 via SSH
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@your-ec2-public-ip
✅ Step 4: Update Server
sudo apt update && sudo apt upgrade -y
✅ Step 5: Install Node.js & NPM
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y
node -v
npm -v
✅ Step 6: Clone Your Project
git clone https://github.com/yourusername/your-repo.git
cd your-repo
npm install
✅ Step 7: Install PM2
sudo npm install pm2 -g

Start your app:

pm2 start index.js

Check running processes:

pm2 list

Enable auto restart on reboot:

pm2 startup
pm2 save
✅ Step 8: Install NGINX
sudo apt install nginx -y
✅ Step 9: Configure NGINX

Open config file:

sudo nano /etc/nginx/sites-available/default

Replace content with:

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

Test config:

sudo nginx -t

Restart NGINX:

sudo systemctl restart nginx
✅ Step 10: Point Domain to EC2

Go to your domain DNS settings:

Add an A Record

Type	Name	Value
A	@	your-ec2-public-ip

Wait 5–10 minutes.

✅ Step 11: Install SSL (HTTPS)

Install Certbot:

sudo apt install certbot python3-certbot-nginx -y

Generate SSL:

sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

Choose option to redirect HTTP → HTTPS.

🔐 Auto Renew SSL
sudo certbot renew --dry-run
🎉 Deployment Complete

Visit:

https://yourdomain.com

Your MERN app is live 🚀

🛠 If You Have React Frontend

Build frontend:

npm run build

Copy build files:

sudo cp -r build/* /var/www/html/

Update NGINX config:

root /var/www/html;
index index.html;

location / {
    try_files $uri /index.html;
}

Restart NGINX:

sudo systemctl restart nginx
🔥 Useful PM2 Commands
pm2 restart app
pm2 stop app
pm2 delete app
pm2 logs
🛡 Production Tips

Use .env file

Never push .env to GitHub

Use MongoDB Atlas

Enable firewall:

sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
⭐ Support

If this helped you:

Star ⭐ this repository

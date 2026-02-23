🚀 Deploying a MERN Stack App on AWS (EC2 + PM2 + NGINX + SSL)

A beginner-friendly, step-by-step guide to deploy your MERN (MongoDB, Express, React, Node.js) app on AWS using:

🖥️ EC2 (Ubuntu)

⚙️ PM2 (Process Manager)

🌐 NGINX (Reverse Proxy)

🔐 Let’s Encrypt (Free SSL)

📌 Prerequisites

Before starting, make sure you have:

AWS Account

A registered domain (e.g., from Namecheap, GoDaddy, etc.)

MERN app pushed to GitHub

Basic knowledge of terminal commands

🧱 Architecture Overview
User → Domain → NGINX → Node.js App (PM2) → MongoDB

NGINX handles traffic and forwards it to your Node app.

PM2 keeps your Node app running forever.

Let’s Encrypt provides HTTPS (SSL).

✅ Step 1: Create a Free AWS Account

👉 Create your account here:
https://aws.amazon.com/

✅ Step 2: Create & Launch EC2 Instance

Go to AWS Dashboard

Open EC2

Click Launch Instance

Choose:

Ubuntu (22.04 LTS recommended)

Instance type: t2.medium

Create a new key pair (download .pem file)

Allow:

HTTP (80)

HTTPS (443)

SSH (22)

🔐 Connect to EC2
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@your-ec2-public-ip
✅ Step 3: Update Server
sudo apt update && sudo apt upgrade -y
✅ Step 4: Install Node.js & NPM
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y
node -v
npm -v
✅ Step 5: Clone Your Project
git clone https://github.com/yourusername/your-repo.git
cd your-repo

Install dependencies:

npm install
✅ Step 6: Install & Configure PM2

Install PM2 globally:

sudo npm install pm2 -g

Start your app:

pm2 start index.js

Check status:

pm2 list

Make app restart automatically on reboot:

pm2 startup
pm2 save
✅ Step 7: Install & Configure NGINX

Install NGINX:

sudo apt install nginx -y

Edit default config:

sudo nano /etc/nginx/sites-available/default

Replace with:

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

Test NGINX config:

sudo nginx -t

Restart NGINX:

sudo systemctl restart nginx
✅ Step 8: Point Domain to EC2

Go to your domain provider DNS settings:

Add an A Record

Name: @

Value: your-ec2-public-ip

Wait 5–10 minutes for DNS propagation.

✅ Step 9: Install SSL (Let’s Encrypt)

Install Certbot:

sudo apt install certbot python3-certbot-nginx -y

Generate SSL:

sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

Follow instructions and choose redirect to HTTPS.

🔐 Auto Renew SSL
sudo certbot renew --dry-run

SSL renews automatically every 90 days.

🎉 Deployment Complete!

Now visit:

https://yourdomain.com

Your MERN app is live 🚀

🛠 If You Have a React Frontend
Option 1: Serve React Build via NGINX (Recommended)

Inside frontend folder:

npm run build

Copy build files to:

sudo cp -r build/* /var/www/html/

Modify NGINX:

root /var/www/html;
index index.html;

location / {
    try_files $uri /index.html;
}
🧠 Useful PM2 Commands
pm2 restart app
pm2 stop app
pm2 delete app
pm2 logs
📌 Production Tips

Use .env file for environment variables

Never push .env to GitHub

Use MongoDB Atlas for cloud DB

Enable firewall:

sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
📚 Tech Stack Used

MongoDB

Express.js

React.js

Node.js

AWS EC2

PM2

NGINX

Let’s Encrypt

👨‍💻 Author

Your Name

If this helped you, ⭐ the repo!

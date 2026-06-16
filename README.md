# 🚀 CI/CD Pipeline for Static Website (Webhooks + EC2 + Systemd + Nginx Optional)

## 📌 Project Overview

This project demonstrates a simple **CI/CD pipeline for a static website** using:

- GitHub Webhooks for automation
- AWS EC2 as deployment server
- systemd service for webhook handler
- Plain HTML, CSS, and JavaScript frontend
- Automatic deployment on every push

No backend framework (Node.js server) or reverse proxy logic is required for the app itself.

---

## 🏗️ Architecture
GitHub Repository
│
│ (Webhook Trigger on push)
▼
EC2 Webhook Listener (Node.js / Express)
│
▼
Deployment Script (pull + copy files)
│
▼
Static Files Folder (HTML / CSS / JS)
│
▼
Optional: Nginx or direct static hosting


---

## ⚙️ Tech Stack

- HTML – Structure
- CSS – Styling
- JavaScript – Frontend logic
- AWS EC2 – Hosting server
- GitHub Webhooks – CI/CD trigger
- systemd – Service manager
- Nginx (optional) – Static file serving

---

## 📁 Project Structure

---

## 🚀 CI/CD Workflow

### 1. Code Push
Developer pushes HTML/CSS/JS changes to GitHub.

### 2. Webhook Trigger
GitHub sends a webhook event to EC2 server.

### 3. Deployment Process
Server automatically:

- Pulls latest code
- Copies static files to `/var/www/html`
- Refreshes deployment

### 4. Website Update
Users instantly see updated frontend in browser.

---

## 🔧 Setup Instructions

### 1. Install Required Tools

```
sudo apt update
sudo apt install git -y
```

### 2. Clone Repository
```
git clone https://github.com/your-username/your-repo.git
cd your-repo
```
⚙️ Webhook Listener Setup

Install Node.js (only for webhook listener):

sudo apt install nodejs npm -y
Create Webhook Server
mkdir webhook
cd webhook
npm init -y
npm install express

server.js
const express = require("express");
const { exec } = require("child_process");

const app = express();
app.use(express.json());

app.post("/webhook", (req, res) => {
    console.log("Webhook received");

    exec("sh ../scripts/deploy.sh", (err, stdout, stderr) => {
        if (err) {
            console.error(err);
            return res.status(500).send("Deployment failed");
        }

        console.log(stdout);
        res.send("Deployment successful");
    });
});

app.listen(4000, () => {
    console.log("Webhook listener running on port 4000");
});
📜 Deployment Script

Create scripts/deploy.sh:

#!/bin/bash

echo "Starting deployment..."

cd /home/ubuntu/your-repo

git pull origin main

# Copy static files to web server directory
sudo cp -r public/* /var/www/html/

echo "Deployment completed successfully"

Make it executable:

chmod +x scripts/deploy.sh
⚙️ Systemd Service (Webhook Listener)

Create service file:

sudo nano /etc/systemd/system/webhook.service

Service Configuration

```
[Unit]
Description=Webhook Listener Service
After=network.target

[Service]
ExecStart=/usr/bin/node /home/ubuntu/your-repo/webhook/server.js
WorkingDirectory=/home/ubuntu/your-repo/webhook
Restart=always
User=ubuntu
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```
Enable Service

sudo systemctl daemon-reload
sudo systemctl enable webhook
sudo systemctl start webhook
sudo systemctl status webhook

🌐 Optional: Nginx Setup

Install Nginx:

sudo apt install nginx -y

Configure Site
sudo nano /etc/nginx/sites-available/default

Configuration

server {
    listen 80;
    server_name your_domain_or_ip;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

Restart Nginx
sudo systemctl restart nginx

🔔 GitHub Webhook Setup

Go to GitHub repository
Settings → Webhooks → Add Webhook

Payload URL
http://<EC2_PUBLIC_IP>:4000/webhook

Content Type
application/json

Event
push


🏁 Conclusion

This project demonstrates a simple but powerful CI/CD pipeline for static websites using:

GitHub Webhooks
AWS EC2
systemd service
HTML, CSS, JavaScript

Every push automatically updates the live website.


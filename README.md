#  Nginx Proxy Configuration with Custom Logging & SSL

##  Overview  
This document describes how to install and configure **Nginx** with **Proxy Pass**, **Access/Error Logging**, **Custom Log Format**, and **Self-Signed SSL** on Ubuntu.

---

## 📂 Table of Contents
- [1️⃣ Installation](#1️⃣-installation)
- [2️⃣ Configuring Proxy Pass](#2️⃣-configuring-proxy-pass)
- [3️⃣ Enabling Logs](#3️⃣-enabling-logs)
- [4️⃣ Setting Up SSL (Self-Signed)](#4️⃣-setting-up-ssl-self-signed)
- [5️⃣ Custom Log Format](#5️⃣-custom-log-format)


---

## 1️⃣ Installation
Update the system and install **Nginx**:
```bash
sudo apt update
sudo apt install nginx -y
```
Verify the installation:
```bash
systemctl status nginx
```
If the output shows **active (running)**, Nginx is successfully installed.

---

## 2️⃣ Configuring Proxy Pass
Modify the default Nginx configuration:
```bash
sudo nano /etc/nginx/sites-available/default
```
Replace the contents with the following:
```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Save and restart Nginx:
```bash
sudo systemctl restart nginx
```

---

## 3️⃣ Enabling Logs
Edit the Nginx config file:
```bash
sudo nano /etc/nginx/sites-available/proxy.conf
```
Add the following lines:
```nginx
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```
Restart Nginx:
```bash
sudo systemctl restart nginx
```
Check logs:
```bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

---

## 4️⃣ Setting Up SSL (Self-Signed)
Generate a self-signed SSL certificate:
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048     -keyout /etc/ssl/private/nginx-selfsigned.key     -out /etc/ssl/certs/nginx-selfsigned.crt
```
Modify the Nginx config:
```bash
sudo nano /etc/nginx/sites-available/default
```
Add the following:
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;

    location / {
        proxy_pass http://backend-server-ip:port/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    access_log /var/log/nginx/custom_access.log custom_format;
    error_log /var/log/nginx/custom_error.log warn;
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```
Restart Nginx:
```bash
sudo systemctl restart nginx
```
Test SSL:
```bash
curl -k https://<server-ip>
```

---

## 5️⃣ Custom Log Format
Edit the Nginx config:
```bash
sudo nano /etc/nginx/nginx.conf
```
Add:
```nginx
log_format custom_format '$remote_addr - $remote_user [$time_local] '
                         '"$request" $status $body_bytes_sent '
                         '"$http_referer" "$http_user_agent"';
access_log /var/log/nginx/custom_access.log custom_format;
```
Restart Nginx:
```bash
sudo systemctl restart nginx
```
Check the new log:
```bash
tail -f /var/log/nginx/custom_access.log
```

---


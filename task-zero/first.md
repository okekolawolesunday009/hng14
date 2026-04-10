# 🚀 HNG DevOps Hardened Linux Server Setup (run.md)

This guide documents the steps to configure a hardened Linux server for production with strict security, SSH-only access, firewall rules, Nginx setup, and SSL via Let’s Encrypt.

---

## 🔐 1. Create Non-Root User

```bash
adduser hngdevops
usermod -aG sudo hngdevops
```

---

## 🔒 2. Lock Down Root Access

Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Update:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 🔑 3. Configure SSH Access (Key-Based Only)

On your local machine:

```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id hngdevops@your_server_ip
```

Test:

```bash
ssh hngdevops@your_server_ip
```

---

## ⚙️ 4. Lock Down Sudo (No Password)

```bash
sudo visudo
```

Add:

```
hngdevops ALL=(ALL) NOPASSWD: ALL
```

---

## 🔥 5. Configure Firewall (UFW)

```bash
sudo apt update
sudo apt install ufw -y
```

Default policies:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow required ports:

```bash
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
```

Enable firewall:

```bash
sudo ufw enable
sudo ufw status
```

---

## 🌐 6. Install and Configure Nginx

```bash
sudo apt install nginx -y
```

Allow Nginx through firewall:

```bash
sudo ufw allow 'Nginx Full'
```

---

## 📄 7. Serve Static Page

Create web root:

```bash
sudo mkdir -p /var/www/hng
sudo chown -R hngdevops:hngdevops /var/www/hng
```

Create index:

```bash
nano /var/www/hng/index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
  <title>HNG DevOps</title>
</head>
<body>
  <h1>Welcome - hngdevops</h1>
</body>
</html>
```

---

## 🔌 8. Create API Endpoint (/api)

Edit Nginx config:

```bash
sudo nano /etc/nginx/sites-available/hng
```

Add:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    root /var/www/hng;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api {
        default_type application/json;

        if ($request_uri ~* "/API") {
            return 400 '{"error":"Bad Request - wrong capitalization"}';
        }

        return 200 '{"status":"success","user":"hngdevops"}';
    }
}
```

Enable config:

```bash
sudo ln -s /etc/nginx/sites-available/hng /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔐 9. Install SSL (Let’s Encrypt)

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run:

```bash
sudo certbot --nginx -d yourdomain.com
```

---

## 🔁 10. Enforce HTTPS (301 Redirect)

Certbot usually auto-configures this. Verify:

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}
```

---

## ✅ 11. Expected Results

| Endpoint             | Expected Response     |
| -------------------- | --------------------- |
| `http://domain`      | 301 redirect to HTTPS |
| `https://domain`     | 200 OK (HTML page)    |
| `https://domain/api` | 200 JSON response     |
| `/API` (wrong case)  | 400 error             |

---

## 🔍 12. Verification Commands

```bash
curl -I http://yourdomain.com
curl -I https://yourdomain.com
curl https://yourdomain.com/api
curl https://yourdomain.com/API
```

---

## 🛡️ Security Summary

* Root login disabled
* SSH key-based authentication only
* Password login disabled
* Firewall restricts all except 22, 80, 443
* Sudo without password (controlled user)
* HTTPS enforced with SSL

---

## 🎯 Done!

Your hardened Linux server is now production-ready with:

* Secure SSH access
* Minimal open ports
* Static + API response via Nginx
* SSL encryption enabled

---

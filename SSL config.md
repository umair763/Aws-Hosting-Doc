
# Setting Up Free SSL with Certbot and Nginx

### 1. Install Certbot and the Nginx plugin
Run the command:
```bash
sudo apt update && sudo apt install certbot python3-certbot-nginx -y
````

---

### 2. Obtain a Free SSL Certificate

Run the following command, replacing `yourdomain.com` with your actual domain:

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

> **Note:**
> If your domain name is not yet configured, update your DNS A Record to point to your EC2 instance before running Certbot.

---

### 3. Successful Certificate Installation

If the process is successful, you will see:

```
Congratulations! Your certificate and chain have been saved at:
  /etc/letsencrypt/live/yourdomain.com/fullchain.pem
```

---

### 4. Configure Nginx for SSL

Edit your Nginx configuration file (e.g., `/etc/nginx/sites-available/default`) and add the following:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/myproject;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
}
```

---

### 5. You are Done

Your website should now be accessible securely via HTTPS:

```
https://yourdomain.com
```

---

### 6. Verify SSL Setup

Run the command:

```bash
curl -I https://yourdomain.com
```

This will show the HTTP headers confirming your website is serving over HTTPS.

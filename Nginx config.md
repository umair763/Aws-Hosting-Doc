# Nginx and PM2 Setup Guide

### 1. Install Nginx
Run the command:
```bash
sudo apt-get install -y nginx
````

### 2. Install PM2 (Package Manager for Node.js backend)

Run the command:

```bash
sudo npm i -g pm2
```

### 3. Check PM2 package details

Run the command:

```bash
pm2
```

---

### 4. Setup Nginx (Popular web server)

* Navigate to the server directory:

```bash
cd /etc/nginx/sites-available
```

* You can see a file named `default` when you run:

```bash
ls
```

* Open the `default` file:

```bash
sudo nano default
```

---

### 5. Verify Nginx Server

* Locate the public IP of your AWS instance.
* Load it in the browser.
* If redirected to the "Welcome to nginx" page, the server is up.

---

### 6. Add backend API path and configure port

Add the following inside the server block (adjust `/api` and port number according to your project):

```nginx
location /api {
    rewrite ^\/api\/(.*)$ /api/$1 break;
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # Add these for better proxy handling
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_cache_bypass $http_upgrade;
}
```
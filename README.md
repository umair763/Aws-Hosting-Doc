# AWS Hosting Documentation - MERN Backend

A complete guide to host your MERN (MongoDB, Express.js, React, Node.js) backend on AWS EC2 instance.

## Table of Contents

- [Prerequisites](#prerequisites)
- [AWS EC2 Setup](#aws-ec2-setup)
- [Server Configuration](#server-configuration)
- [Node.js Installation](#nodejs-installation)
- [MongoDB Setup](#mongodb-setup)
- [Backend Deployment](#backend-deployment)
- [Environment Configuration](#environment-configuration)
- [Security Configuration](#security-configuration)
- [Domain and SSL Setup](#domain-and-ssl-setup)
- [Process Management with PM2](#process-management-with-pm2)
- [Monitoring and Logs](#monitoring-and-logs)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Prerequisites

Before starting, ensure you have:

- AWS Account with appropriate permissions
- Your MERN backend application ready for deployment
- Domain name (optional, for custom domain setup)
- Basic knowledge of Linux command line
- SSH key pair for secure access

## AWS EC2 Setup

### 1. Launch EC2 Instance

1. **Log into AWS Console** and navigate to EC2 Dashboard
2. **Click "Launch Instance"**
3. **Choose AMI**: Select "Ubuntu Server 22.04 LTS" (recommended)
4. **Choose Instance Type**: 
   - For development/testing: `t2.micro` (free tier eligible)
   - For production: `t2.small` or higher based on your needs
5. **Configure Instance Details**: Use default settings or customize as needed
6. **Add Storage**: 8GB minimum, 20GB+ recommended for production
7. **Configure Security Group**: Create a new security group with these rules:
   ```
   SSH (22)          - Your IP address
   HTTP (80)         - 0.0.0.0/0
   HTTPS (443)       - 0.0.0.0/0
   Custom TCP (3000) - 0.0.0.0/0 (for your backend port)
   MongoDB (27017)   - 127.0.0.1/32 (localhost only for security)
   ```
8. **Review and Launch** with your existing key pair or create a new one

### 2. Connect to Your Instance

```bash
# Replace 'your-key.pem' with your actual key file and 'your-instance-ip' with your instance's public IP
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@your-instance-ip
```

## Server Configuration

### 1. Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Essential Tools

```bash
sudo apt install -y curl wget git unzip software-properties-common
```

## Node.js Installation

### Option 1: Using NodeSource Repository (Recommended)

```bash
# Install Node.js 18.x (LTS)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

### Option 2: Using NVM (Node Version Manager)

```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc

# Install latest LTS Node.js
nvm install --lts
nvm use --lts
```

## MongoDB Setup

### 1. Install MongoDB Community Edition

```bash
# Import MongoDB public GPG key
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

# Create a list file for MongoDB
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

# Update package database
sudo apt-get update

# Install MongoDB
sudo apt-get install -y mongodb-org
```

### 2. Configure MongoDB

```bash
# Start MongoDB service
sudo systemctl start mongod
sudo systemctl enable mongod

# Check MongoDB status
sudo systemctl status mongod

# Secure MongoDB installation (recommended for production)
mongo --eval "db.adminCommand('listCollections')"
```

### 3. Configure MongoDB Security (Production)

```bash
# Connect to MongoDB shell
mongo

# Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "your-strong-password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})

# Create database user for your application
use your-database-name
db.createUser({
  user: "app-user",
  pwd: "app-password",
  roles: [ { role: "readWrite", db: "your-database-name" } ]
})

exit
```

### 4. Enable MongoDB Authentication

```bash
# Edit MongoDB configuration
sudo nano /etc/mongod.conf

# Add/modify these lines:
security:
  authorization: enabled

# Restart MongoDB
sudo systemctl restart mongod
```

## Backend Deployment

### 1. Clone Your Repository

```bash
# Navigate to web directory
cd /var/www

# Clone your backend repository
sudo git clone https://github.com/your-username/your-backend-repo.git
sudo chown -R ubuntu:ubuntu your-backend-repo
cd your-backend-repo
```

### 2. Install Dependencies

```bash
# Install npm dependencies
npm install

# For production, use:
npm ci --production
```

### 3. Build Your Application (if needed)

```bash
# If your app requires building
npm run build
```

## Environment Configuration

### 1. Create Environment File

```bash
# Create .env file
nano .env
```

### 2. Configure Environment Variables

```env
# Server Configuration
PORT=3000
NODE_ENV=production

# Database Configuration
MONGODB_URI=mongodb://app-user:app-password@localhost:27017/your-database-name

# JWT Secret (generate a strong secret)
JWT_SECRET=your-super-secret-jwt-key

# CORS Configuration
CORS_ORIGIN=https://your-frontend-domain.com

# Other API Keys
API_KEY=your-api-key
EMAIL_SERVICE_KEY=your-email-service-key
```

### 3. Secure Environment File

```bash
# Set proper permissions
chmod 600 .env
```

## Security Configuration

### 1. Configure Firewall (UFW)

```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow ssh

# Allow HTTP and HTTPS
sudo ufw allow 80
sudo ufw allow 443

# Allow your backend port (be specific about source if possible)
sudo ufw allow 3000

# Check firewall status
sudo ufw status
```

### 2. Install and Configure Fail2Ban

```bash
# Install fail2ban
sudo apt install fail2ban -y

# Configure fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Enable and start fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 3. Regular Security Updates

```bash
# Set up automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
```

## Domain and SSL Setup

### 1. Point Domain to Your Instance

- In your domain registrar's DNS settings, create an A record pointing to your EC2 instance's public IP
- Wait for DNS propagation (can take up to 48 hours)

### 2. Install Nginx (Reverse Proxy)

```bash
# Install Nginx
sudo apt install nginx -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 3. Configure Nginx

```bash
# Create Nginx configuration for your domain
sudo nano /etc/nginx/sites-available/your-domain.com
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 4. Install SSL Certificate with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Test automatic renewal
sudo certbot renew --dry-run
```

## Process Management with PM2

### 1. Install PM2

```bash
# Install PM2 globally
sudo npm install -g pm2
```

### 2. Create PM2 Configuration

```bash
# Create ecosystem file
nano ecosystem.config.js
```

```javascript
module.exports = {
  apps: [{
    name: 'mern-backend',
    script: './server.js', // or your main file
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true
  }]
};
```

### 3. Start Application with PM2

```bash
# Create logs directory
mkdir logs

# Start application
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 startup script
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

### 4. PM2 Commands

```bash
# View running processes
pm2 list

# View logs
pm2 logs

# Restart application
pm2 restart mern-backend

# Stop application
pm2 stop mern-backend

# Monitor resources
pm2 monit
```

## Monitoring and Logs

### 1. Application Logs

```bash
# View PM2 logs
pm2 logs --lines 100

# View specific app logs
pm2 logs mern-backend

# View error logs only
pm2 logs mern-backend --err
```

### 2. System Monitoring

```bash
# Install htop for system monitoring
sudo apt install htop -y

# View system resources
htop

# Check disk usage
df -h

# Check memory usage
free -h
```

### 3. MongoDB Logs

```bash
# View MongoDB logs
sudo tail -f /var/log/mongodb/mongod.log

# Check MongoDB status
sudo systemctl status mongod
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Application Won't Start

```bash
# Check Node.js version
node --version

# Check for missing dependencies
npm install

# Check environment variables
cat .env

# Check application logs
pm2 logs
```

#### 2. Database Connection Issues

```bash
# Check MongoDB status
sudo systemctl status mongod

# Test MongoDB connection
mongo --eval "db.runCommand({connectionStatus : 1})"

# Check MongoDB logs
sudo tail -f /var/log/mongodb/mongod.log
```

#### 3. Port Already in Use

```bash
# Check what's using the port
sudo lsof -i :3000

# Kill process if needed
sudo kill -9 PID
```

#### 4. SSL Certificate Issues

```bash
# Check certificate status
sudo certbot certificates

# Renew certificate
sudo certbot renew

# Check Nginx configuration
sudo nginx -t
```

#### 5. High Memory Usage

```bash
# Check memory usage
free -h

# Check PM2 processes
pm2 list

# Restart PM2 processes
pm2 restart all
```

### Log Files Locations

- **Application Logs**: `~/your-app/logs/`
- **PM2 Logs**: `~/.pm2/logs/`
- **Nginx Logs**: `/var/log/nginx/`
- **MongoDB Logs**: `/var/log/mongodb/`
- **System Logs**: `/var/log/syslog`

## Best Practices

### 1. Security

- ✅ Use strong passwords for all accounts
- ✅ Enable MongoDB authentication
- ✅ Configure firewall rules properly
- ✅ Keep system packages updated
- ✅ Use HTTPS with valid SSL certificates
- ✅ Limit SSH access to specific IP addresses
- ✅ Regular security audits

### 2. Performance

- ✅ Use PM2 cluster mode for better performance
- ✅ Enable gzip compression in Nginx
- ✅ Implement proper caching strategies
- ✅ Monitor resource usage regularly
- ✅ Optimize MongoDB queries and indexes

### 3. Maintenance

- ✅ Set up automated backups for MongoDB
- ✅ Monitor logs regularly
- ✅ Keep dependencies updated
- ✅ Implement proper error handling
- ✅ Set up monitoring and alerting

### 4. Backup Strategy

```bash
# MongoDB backup script
#!/bin/bash
BACKUP_DIR="/home/ubuntu/backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR
mongodump --out $BACKUP_DIR/mongodb_$DATE

# Keep only last 7 days of backups
find $BACKUP_DIR -name "mongodb_*" -mtime +7 -exec rm -rf {} +
```

### 5. Environment-Specific Configurations

#### Development
- Use `NODE_ENV=development`
- Enable detailed logging
- Allow CORS from localhost

#### Production
- Use `NODE_ENV=production`
- Implement rate limiting
- Enable security headers
- Use process clustering
- Monitor performance metrics

---

## Quick Deployment Checklist

- [ ] EC2 instance launched and configured
- [ ] Security groups properly configured
- [ ] Node.js and npm installed
- [ ] MongoDB installed and secured
- [ ] Application deployed and dependencies installed
- [ ] Environment variables configured
- [ ] PM2 configured and application running
- [ ] Nginx configured as reverse proxy
- [ ] SSL certificate installed
- [ ] Firewall rules configured
- [ ] Monitoring and logging set up
- [ ] Backup strategy implemented

---

## Support and Contributing

If you encounter any issues or have suggestions for improvement, please:

1. Check the troubleshooting section above
2. Review AWS and MongoDB documentation
3. Create an issue in this repository
4. Contribute improvements via pull requests

---

**⚠️ Important Notes:**
- Always test your deployment in a staging environment first
- Keep your AWS credentials and database passwords secure
- Monitor your AWS usage to avoid unexpected charges
- Regular backups are crucial for production applications

Happy hosting! 🚀

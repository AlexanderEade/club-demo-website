# Nginx Setup Guide for Network Security Demo Website

This guide will walk you through setting up nginx on a fresh VM to serve the Network Security Demo website.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installing Nginx](#installing-nginx)
3. [Configuring Nginx](#configuring-nginx)
4. [Deploying the Website](#deploying-the-website)
5. [Testing and Troubleshooting](#testing-and-troubleshooting)
6. [Firewall Configuration (Demonstrating the Concepts!)](#firewall-configuration)

---

## Prerequisites

- A fresh VM running Ubuntu/Debian or CentOS/RHEL
- Root or sudo access
- Basic command line knowledge

---

## Installing Nginx

### On Ubuntu/Debian:

```bash
# Update package lists
sudo apt update

# Install nginx
sudo apt install nginx -y

# Start nginx service
sudo systemctl start nginx

# Enable nginx to start on boot
sudo systemctl enable nginx

# Check nginx status
sudo systemctl status nginx
```

### On CentOS/RHEL/Rocky Linux:

```bash
# Update package lists
sudo yum update -y

# Install nginx
sudo yum install nginx -y

# Start nginx service
sudo systemctl start nginx

# Enable nginx to start on boot
sudo systemctl enable nginx

# Check nginx status
sudo systemctl status nginx
```

### Verify Installation

Open a web browser and navigate to your VM's IP address:
```
http://<your-vm-ip>
```

You should see the default nginx welcome page.

---

## Configuring Nginx

### 1. Create a Site Configuration

Create a new configuration file for your website:

```bash
sudo nano /etc/nginx/sites-available/network-security-demo
```

Add the following configuration:

```nginx
server {
    listen 80;
    listen [::]:80;
    
    # Replace with your domain or use _ for default
    server_name network-security-demo.local _;
    
    # Document root - where your website files are
    root /var/www/network-security-demo;
    index index.html;
    
    # Logging
    access_log /var/log/nginx/network-security-demo-access.log;
    error_log /var/log/nginx/network-security-demo-error.log;
    
    # Main location block
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Cache static files
    location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
 # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

### 2. Enable the Site

**On Ubuntu/Debian** (uses sites-enabled/sites-available pattern):

```bash
# Create symbolic link to enable the site
sudo ln -s /etc/nginx/sites-available/network-security-demo /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default
```

**On CentOS/RHEL** (uses conf.d directory):

```bash
# Copy or move the config to conf.d
sudo cp /etc/nginx/sites-available/network-security-demo /etc/nginx/conf.d/network-security-demo.conf

# Remove default config (optional)
sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

### 3. Test Nginx Configuration

```bash
# Test configuration for syntax errors
sudo nginx -t

# If successful, you'll see:
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

## Deploying the Website

### 1. Create Website Directory

```bash
# Create the directory
sudo mkdir -p /var/www/network-security-demo

# Set proper ownership
sudo chown -R $USER:$USER /var/www/network-security-demo

# Set proper permissions
sudo chmod -R 755 /var/www/network-security-demo
```

### 2. Upload Website Files

**Option A: Using Git (Recommended)**

```bash
# Clone your repository
cd /var/www/network-security-demo
git clone https://github.com/AlexanderEade/club-demo-website.git .

# Or if already cloned elsewhere, copy files:
# sudo cp -r /path/to/club-demo-website/* /var/www/network-security-demo/
```

**Option B: Using SCP from your local machine**

```bash
# From your local machine (Windows PowerShell or Linux terminal)
scp -r index.html styles.css user@<vm-ip>:/tmp/

# Then on the VM:
sudo mv /tmp/index.html /tmp/styles.css /var/www/network-security-demo/
```

**Option C: Using SFTP/FTP Client**

Use FileZilla, WinSCP, or similar to upload:
- `index.html`
- `styles.css`

To: `/var/www/network-security-demo/`

### 3. Verify Files

```bash
# Check files are in place
ls -la /var/www/network-security-demo/

# You should see:
# index.html
# styles.css
```

### 4. Set Correct Permissions

```bash
# Set ownership to nginx user
sudo chown -R nginx:nginx /var/www/network-security-demo  # CentOS/RHEL
# OR
sudo chown -R www-data:www-data /var/www/network-security-demo  # Ubuntu/Debian

# Set permissions
sudo chmod -R 755 /var/www/network-security-demo
sudo chmod 644 /var/www/network-security-demo/*.html
sudo chmod 644 /var/www/network-security-demo/*.css
```

### 5. Reload Nginx

```bash
# Reload nginx to apply changes
sudo systemctl reload nginx

# Or restart if reload doesn't work
sudo systemctl restart nginx
```

---

## Testing and Troubleshooting

### Test the Website

Open a browser and navigate to:
```
http://<your-vm-ip>
```

You should see your Network Security Demo website!

### Common Issues and Solutions

#### Issue 1: 403 Forbidden Error

```bash
# Check SELinux context (CentOS/RHEL)
sudo chcon -R -t httpd_sys_content_t /var/www/network-security-demo/

# Or disable SELinux temporarily for testing
sudo setenforce 0
```

#### Issue 2: 404 Not Found

```bash
# Verify files exist
ls -la /var/www/network-security-demo/

# Check nginx error logs
sudo tail -f /var/log/nginx/network-security-demo-error.log
```

#### Issue 3: Permission Denied

```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/network-security-demo  # Ubuntu
sudo chown -R nginx:nginx /var/www/network-security-demo  # CentOS

# Fix permissions
sudo chmod -R 755 /var/www/network-security-demo
```

#### Issue 4: Can't Access from Browser

```bash
# Check if nginx is running
sudo systemctl status nginx

# Check what ports nginx is listening on
sudo netstat -tlnp | grep nginx
# or
sudo ss -tlnp | grep nginx

# Should show port 80 listening
```

### View Logs

```bash
# Access logs
sudo tail -f /var/log/nginx/network-security-demo-access.log

# Error logs
sudo tail -f /var/log/nginx/network-security-demo-error.log

# General nginx logs
sudo tail -f /var/log/nginx/error.log
```

---

## Firewall Configuration (Demonstrating the Concepts!)

**This is a perfect opportunity to demonstrate the concepts from your website!**

### Ubuntu/Debian (using UFW):

```bash
# Enable firewall
sudo ufw enable

# Allow HTTP (port 80) - Demonstrating Layer 4 control!
sudo ufw allow 80/tcp
sudo ufw allow http

# Allow HTTPS (port 443) for future use
sudo ufw allow 443/tcp
sudo ufw allow https

# DENY SSH from public (or allow only from specific IP)
# Option 1: Allow SSH only from specific IP/network
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp

# Option 2: Block SSH entirely from public
# (Don't do this if you need SSH access!)
# sudo ufw deny 22/tcp

# Check firewall status
sudo ufw status verbose
```

### CentOS/RHEL (using firewalld):

```bash
# Start firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Allow HTTP (port 80)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-port=80/tcp

# Allow HTTPS (port 443)
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=443/tcp

# Restrict SSH to specific source IP/network
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'

# Remove default SSH rule (be careful!)
# sudo firewall-cmd --permanent --remove-service=ssh

# Reload firewall
sudo firewall-cmd --reload

# Check status
sudo firewall-cmd --list-all
```

### Using iptables (Manual approach - as shown on your website):

```bash
# Flush existing rules (careful in production!)
sudo iptables -F

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow HTTP (port 80)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow HTTPS (port 443)
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow SSH only from internal network
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Drop all other SSH attempts
sudo iptables -A INPUT -p tcp --dport 22 -j DROP

# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Save rules (Ubuntu/Debian)
sudo apt install iptables-persistent -y
sudo netfilter-persistent save

# Save rules (CentOS/RHEL)
sudo service iptables save
```

**This configuration demonstrates exactly what your website teaches:**
- ? HTTP (port 80) is ALLOWED - website accessible
- ? HTTPS (port 443) is ALLOWED - secure website accessible  
- ? SSH (port 22) is DENIED from public or restricted to internal network

---

## Quick Start Script

Here's a complete setup script you can run:

```bash
#!/bin/bash

# Network Security Demo - Nginx Setup Script
# Run as: sudo bash setup.sh

echo "Starting nginx setup for Network Security Demo..."

# Detect OS
if [ -f /etc/debian_version ]; then
    OS="debian"
 WEB_USER="www-data"
elif [ -f /etc/redhat-release ]; then
    OS="redhat"
    WEB_USER="nginx"
else
    echo "Unsupported OS"
    exit 1
fi

# Install nginx
echo "Installing nginx..."
if [ "$OS" = "debian" ]; then
    apt update
    apt install nginx git -y
else
    yum install nginx git -y
fi

# Start and enable nginx
systemctl start nginx
systemctl enable nginx

# Create website directory
echo "Creating website directory..."
mkdir -p /var/www/network-security-demo

# Clone repository or copy files
echo "Deploying website files..."
cd /var/www/network-security-demo
# git clone https://github.com/AlexanderEade/club-demo-website.git .

# Set permissions
chown -R $WEB_USER:$WEB_USER /var/www/network-security-demo
chmod -R 755 /var/www/network-security-demo

# Configure nginx
echo "Configuring nginx..."
cat > /etc/nginx/sites-available/network-security-demo << 'EOF'
server {
    listen 80;
    server_name _;
    root /var/www/network-security-demo;
    index index.html;
    
    location / {
 try_files $uri $uri/ =404;
    }
}
EOF

# Enable site
if [ "$OS" = "debian" ]; then
  ln -sf /etc/nginx/sites-available/network-security-demo /etc/nginx/sites-enabled/
    rm -f /etc/nginx/sites-enabled/default
else
    cp /etc/nginx/sites-available/network-security-demo /etc/nginx/conf.d/network-security-demo.conf
fi

# Test and reload nginx
nginx -t && systemctl reload nginx

# Configure firewall
echo "Configuring firewall..."
if [ "$OS" = "debian" ]; then
  ufw allow 80/tcp
    ufw allow 443/tcp
  # ufw enable  # Uncomment if UFW is not enabled
else
    firewall-cmd --permanent --add-service=http
    firewall-cmd --permanent --add-service=https
    firewall-cmd --reload
fi

echo "Setup complete!"
echo "Upload your index.html and styles.css to /var/www/network-security-demo/"
echo "Then access your website at http://$(hostname -I | awk '{print $1}')"
```

Save this as `setup.sh` and run:
```bash
chmod +x setup.sh
sudo ./setup.sh
```

---

## Summary

Your website is now:
1. ? Served by nginx on port 80 (HTTP)
2. ? Accessible to the public for demonstration
3. ? Protected with proper firewall rules
4. ? Demonstrating the very concepts it teaches!

**Next Steps:**
- Add HTTPS with Let's Encrypt (optional)
- Configure additional security headers
- Set up monitoring and logging
- Create backup procedures

---

## Need Help?

Common commands for nginx:
```bash
sudo systemctl start nginx      # Start nginx
sudo systemctl stop nginx       # Stop nginx
sudo systemctl restart nginx    # Restart nginx
sudo systemctl reload nginx     # Reload config without downtime
sudo systemctl status nginx     # Check status
sudo nginx -t      # Test configuration
```

## Linux Server Setup & Nginx Configuration

The HNG challenge is a fast-paced, fully remote bootcamp designed to find and develop the most talented software developers, Cloud & DevOps Engineers, designers, and digital experts across Africa and the world. It is a project-based program that simulates a real-world work environment through high-pressure tasks and team collaboration through stages 0 to 10.

# Task Name
Stage 0 (DevOps) Linux Server Setup & Nginx Configuration
Requirements / Task Brief
You will provision a Linux server, install and configure Nginx to serve two different locations, and secure it with a valid SSL certificate. No Docker, no Compose, no automation tools — just a bare Linux server and your hands.

# What You Must Do:

# Server Setup
Provision a Linux server (any cloud provider)
Create a non-root user called hngdevops with sudo privileges
Disable root SSH login
Disable password-based SSH authentication - key-based only
Configure UFW to allow only ports 22, 80, and 443 (all other ports closed)


# Nginx Configuration

Install Nginx and configure it to serve:
GET / : static HTML page containing your HNG username as visible text
GET /api : returns JSON:
							json

								{

 									"message": "HNGI14 Stage 1",

 									"track": "DevOps",

 									"username": "your-hng-username"

								}	


SSL
Obtain valid Let's Encrypt SSL certificate for your domain
Configure Nginx to serve both endpoints over HTTPS
HTTP requests must redirect to HTTPS with 301


# Technical Requirements:
/api must return Content-Type: application/json and HTTP status 200
username in JSON must match registered HNG username exactly (case-sensitive)
HTTP→HTTPS redirect must be 301 (not 302)
SSL certificate must be valid Let's Encrypt (no self-signed)
username on / must be visible text (not hidden/commented)
Nginx must be active web server
UFW must be active
Evaluation Criteria / Acceptance Criteria
Non-root user hngdevops exists with sudo
Root SSH login disabled
Password auth disabled (key-based only)
UFW active with only ports 22, 80, 443 open
Nginx installed and running
GET / returns HTML page with username visible
GET /api returns correct JSON with matching username
/api has Content-Type: application/json
/api returns HTTP 200
Valid Let's Encrypt SSL installed
HTTPS works on both endpoints
HTTP redirects to HTTPS with 301
No Docker/Compose/automation tools used

# Note a public ssh key was given to me to add to my server

# Solution

I already have an account with AWS and Azure so i provisioned my Linux server with AWS

1. Provision a Linux Server on AWS
AMI: Ubuntu 22.04 LTS
Instance type: t2.micro (free tier)
Key pair: create/download .pem file
Security Group:
Allow:
SSH (22)
HTTP (80)
HTTPS (443)

I connected to my AWS remote server with the public ip using mobaxterm

# Creating a Passwordless root user
sudo useradd -m -s /bin/bash hngdevops

# Add to sudo group
sudo usermod -aG sudo hngdevops

# Verify user exists
id hngdevops

# Expected output:
uid=1001(hngdevops) gid=1001(hngdevops) groups=1001(hngdevops),27(sudo)

# Configuring Passwordless Sudo
sudo visudo -f /etc/sudoers.d/hngdevops
Add this exact line:
hngdevops ALL=(ALL) NOPASSWD: /usr/sbin/sshd, /usr/sbin/ufw

Ctrl+X → Y → Enter to save

# Verify it saved correctly
sudo cat /etc/sudoers.d/hngdevops
# Expected output:
hngdevops ALL=(ALL) NOPASSWD: /usr/sbin/sshd, /usr/sbin/ufw

# To Add the public SSH Key given
Switch to hngdevops user
sudo su - hngdevops

# Create .ssh directory
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Create authorized_keys file and paste the ssh key
nano ~/.ssh/authorized_keys

Paste the public ssh key here (should look like):

ssh eed AAAAB3NzaC1yc2EAAAADAQABAAAB... @key

Ctrl+X → Y → Enter to save

# Set correct permissions
chmod 600 ~/.ssh/authorized_keys

# Verify the key was saved
cat ~/.ssh/authorized_keys

# Verify permissions are correct
ls -la ~/.ssh/

# Expected output should look like this:
drwx------ 2 hngdevops hngdevops 4096 Apr 12 11:37 .
drwxr-x--- 4 hngdevops hngdevops 4096 Apr 12 11:35 ..
-rw------- 1 hngdevops hngdevops  136 Apr 12 11:37 authorized_keys

# Now switch back to ubuntu user
exit

# Then edit SSH config
sudo nano /etc/ssh/sshd_config

# Find and update these lines:
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

# Save and restart SSH
sudo systemctl restart ssh

# Verify SSH is still running
sudo systemctl status ssh

# Expected Output:
Active: active (running) ✅

# Configure Firewall (UFW)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

## Note: Install firewall, if firewall is not installed already

# Then verify
sudo ufw status verbose

# Also Verify if Socket is Active
sudo systemctl status ssh.socket

![Firewall enabled](<firewall enabled.png>)

# Summary 
🔒 Root login disabled
🔑 Key-based SSH only
🛡️ All ports closed except 22, 80, 443
👤 hngdevops user ready for examiner
⚡ Passwordless sudo for sshd and ufw

## Phase 2   Nginx Configuration
## Note: You should have a domain name ready, You'll need one for the Let's Encrypt SSL certificate.
# Note: My domain is servimatch.online

# Step 1 — Install Nginx
sudo apt update
sudo apt install nginx -y
# Verify Nginx is running
sudo systemctl status nginx

# Step 2 — Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Step 3 — Configure Nginx First
Create the config file
sudo nano /etc/nginx/sites-available/domain

# Paste this configuration, edit and add your domain and preferred content
nginxserver {
    listen 80;
    server_name servimatch.online www.servimatch.online;

    # Static HTML page at /
    location / {
        root /var/www/servimatch.online;
        index index.html;
    }

    # JSON API at /api
    location /api {
        default_type application/json;
        return 200 '{"message": "HNG14 Stage 0", "track": "DevOps", "username": "Chinomso Daniel"}';
    }
}

# To close the nano editor
   Ctrl+X → Y → Enter to save

# Step 4 — Create the HTML Page
sudo mkdir -p /var/www/servimatch.online
sudo nano /var/www/servimatch.online/index.html

Paste this, edit and add your desired content:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>HNG DevOps</title>
</head>
<body>
    <h1>Chinomso Daniel</h1>
</body>
</html>

Ctrl+X → Y → Enter to save

## Step 5 — Enable the Site
# Enable site
sudo ln -s /etc/nginx/sites-available/servimatch.online /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test config
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Step 6: Configure DNS Records on Your Domain Provider
Login to your domain provider and set the A record

# Note @ means the root domain (servimatch.online) and www means www.servimatch.online


# Verify if DNS is propagated correctly
nslookup servimatch.online
Expected output should show your EC2 public IP.
or check online at https://dnschecker.org with your domain

# Expected output:
Name: servimatch.online
Address: your-ec2-public-ip

## Step 7 — Obtain SSL Certificate
sudo certbot --nginx -d servimatch.online -d www.servimatch.online

When prompted:

Enter your email address
Type Y to agree to terms
Type Y or N for marketing emails
Certbot will automatically configure HTTPS and 301 redirect

# Or use non-interactive mode 
# Note: Replace your-email@gmail.com and domain 
sudo certbot --nginx \
  -d domain \
  -d www.domain \
  --agree-tos \
  --email your-email@gmail.com \
  --no-eff-email \
  --non-interactive

  ![certificate installed](<certificate installed-1.png>)

## Step 7 — Verify SSL Auto-renewal
sudo certbot renew --dry-run

![verify ssl](<verify ssl.png>)

## Now Verify If Everything is Working
Run these one by one:
1. Test HTTP → HTTPS redirect (must be 301)
curl -I http://servimatch.online

Expected: HTTP/1.1 301 Moved Permanently

2. Test HTTPS homepage
bashcurl -I https://servimatch.online

Expected: HTTP/2 200

3. Test API endpoint
bashcurl https://servimatch.online/api

Expected:
json{"message": "HNGI14 Stage 1", "track": "DevOps", "username": "Chinomso Daniel"}

4. Test API headers
bashcurl -I https://servimatch.online/api

Expected: content-type: application/json












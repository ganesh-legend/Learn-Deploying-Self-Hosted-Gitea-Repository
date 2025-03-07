# _Learn-Deploying-Self-Hosted-Gitea-Repository_

# Objective:
Deploy a self-hosted Git server using Gitea on an Ubuntu machine, allowing private Git repository 
hosting, user management, and CI/CD automation. 

## Step-by-Step Implementation 

### 1. Install Ubuntu & Update System 
```bash
sudo apt update && sudo apt upgrade -y
```
#
#
### 2. Install Required Packages 
```bash
sudo apt install git mariadb-server nginx certbot python3-certbot-nginx -y
```
#
#
### ️3. Secure MariaDB 
• Set a strong root password 
• Remove anonymous users 
• Disable remote root login
```bash
sudo mysql_secure_installation 
```
#
#
### 4. Create Gitea Database & User 
```bash
sudo mysql -u root -p
```
Run: 
```bash
CREATE DATABASE gitea CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci'; 
CREATE USER 'gitea'@'localhost' IDENTIFIED BY 'Devopsshack@123'; 
GRANT ALL PRIVILEGES ON gitea.* TO 'gitea'@'localhost'; 
FLUSH PRIVILEGES; 
EXIT;
```
#
#
### ️5. Install & Configure Gitea 
#### i) Download & Install Gitea 
```bash
sudo mkdir -p /var/lib/gitea 
sudo useradd --system --home /var/lib/gitea --shell /bin/bash gitea 
sudo wget -O /usr/local/bin/gitea https://dl.gitea.com/gitea/1.23.4/gitea-1.23.4-linux-amd64 
sudo chmod +x /usr/local/bin/gitea
```
#### ii) Create Configuration & Data Directories 
```bash
sudo mkdir -p /etc/gitea /var/lib/gitea/{custom,data,log} 
sudo chown -R gitea:gitea /var/lib/gitea /etc/gitea 
sudo chmod -R 750 /var/lib/gitea /etc/gitea 
sudo touch /etc/gitea/app.ini 
sudo chmod 640 /etc/gitea/app.ini
```
#
#
### ️6. Configure app.ini 
#### i) Open the Gitea configuration file: 
```bash
sudo vi /etc/gitea/app.ini
```
#
#### ii) Update Repository Paths 
```bash
[repository] 
ROOT = /var/lib/gitea/data/gitea-repositories
```
#
#### iii) Update Server Configuration 
```bash
[server] 
APP_DATA_PATH = /var/lib/gitea/data 
DOMAIN = git.ganeshpawar.one 
SSH_DOMAIN = git.skjptpp.in 
HTTP_PORT = 3000 
ROOT_URL = https://git.ganeshpawar.one/ 
LFS_CONTENT_PATH = /var/lib/gitea/data/fs
```
#
#### iv) Update Log Configuration 
```bash
[log] 
ROOT_PATH = /var/lib/gitea/log
```
#
#### v) Fix permissions: 
```bash
sudo chown -R gitea:gitea /etc/gitea 
sudo chmod 640 /etc/gitea/app.ini
```
#
#
### 7. Configure Systemd Service for Gitea 
```bash
sudo vi /etc/systemd/system/gitea.service
```
#### Add: 
```bash
[Unit] 
Description=Gitea Self-Hosted Git Server 
After=network.target mariadb.service
```
```bash
[Service] 
User=gitea 
Group=gitea 
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini 
Restart=always 
WorkingDirectory=/var/lib/gitea 
Environment=USER=gitea HOME=/var/lib/gitea
```
```bash
[Install] 
WantedBy=multi-user.target
```
#### Reload Systemd & Start Gitea 
```bash
sudo systemctl daemon-reload 
sudo systemctl enable --now gitea 
sudo systemctl status gitea 
```
#
#
### ️8. Set Up Reverse Proxy with Nginx 
```bash
sudo vi /etc/nginx/sites-available/gitea
```
#### Add: 
```bash
server { 
  listen 80; 
  server_name git.ganeshpawar.one; 

  location / { 
    proxy_pass http://localhost:3000; 
    proxy_set_header X-Real-IP $remote_addr; 
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    proxy_set_header Host $host; 
  } 
}
```
#### Enable & Restart Nginx: 
```bash
sudo ln -s /etc/nginx/sites-available/gitea /etc/nginx/sites-enabled/ 
sudo systemctl restart nginx
```
#
#
### ️9. Secure Gitea with SSL (Let's Encrypt) 
```bash
sudo certbot --nginx -d git.ganeshpawar.one
```
#### Set up auto-renewal: 
```bash
sudo crontab -e
```
#### Add: 
```bash
0 3 * * * certbot renew --quiet
```

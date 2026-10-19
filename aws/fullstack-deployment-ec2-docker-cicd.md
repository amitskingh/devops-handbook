# 🚀 Fullstack Deployment Guide (EC2 + Docker + Nginx + CI/CD)

---
# 0️⃣ Create EC2 Instance

## Open EC2 Dashboard

Go to:

[AWS EC2 Console](https://console.aws.amazon.com/ec2/?utm_source=chatgpt.com)

---

## Select Region

Choose:

```txt
Mumbai (ap-south-1)
```

---

## Launch Instance

Click:

```txt
Launch Instance
```

---

## Configure Instance

### Name

Example:

```txt
tinyurl-backend-prod
```

---

### Select AMI

Choose:

* Ubuntu Server 24.04 LTS

---

### Select Instance Type

Recommended:

```txt
t2.micro
```

---

# Create Key Pair

Click:

```txt
Create new key pair
```

### Example Name

```txt
tinyurl-key
```

### Settings

| Option | Value |
| ------ | ----- |
| Type   | RSA   |
| Format | .pem  |

The `.pem` file will automatically download.

Example:

```txt
tinyurl-key.pem
```

Store this safely.

---

# Configure Network Settings

Click:

```txt
Edit
```

---

## Allow SSH

| Field  | Value |
| ------ | ----- |
| Type   | SSH   |
| Port   | 22    |
| Source | My IP |

---

## Allow HTTP

| Field  | Value    |
| ------ | -------- |
| Type   | HTTP     |
| Port   | 80       |
| Source | Anywhere |

---

## Allow HTTPS

| Field  | Value    |
| ------ | -------- |
| Type   | HTTPS    |
| Port   | 443      |
| Source | Anywhere |

---

# Configure Storage

Default is enough:

```txt
8 GB
```

---

# Launch EC2

Click:

```txt
Launch Instance
```

Wait around:

```txt
30–60 seconds
```

Your EC2 instance is now ready.

---

# 1️⃣ Connect to EC2

### Go to SSH folder

```bash
cd ~/.ssh
```

### Protect the key

```bash
chmod 400 key_name.pem
```

### Connect to EC2

```bash
ssh -i key_name.pem ubuntu@EC2_PUBLIC_IPV4
```

First time connection will ask confirmation:

```
Are you sure you want to continue connecting?
```

Type:

```
yes
```

Now you are inside a **clean Ubuntu EC2 machine**.

---

# 2️⃣ Install Basic Dependencies

### Update packages

```bash
sudo apt update
```

### Install Docker

```bash
sudo apt install docker.io
```

### Start Docker

```bash
sudo systemctl start docker
```

### Enable Docker on boot

```bash
sudo systemctl enable docker
```

This ensures Docker **automatically starts after EC2 reboot**.

---

# 3️⃣ Allow Ubuntu User to Run Docker

By default only **root** can run Docker.

Add ubuntu user to docker group:

```bash
sudo usermod -aG docker ubuntu
```

Now **logout and reconnect SSH**.

```bash
exit
ssh -i key_name.pem ubuntu@EC2_PUBLIC_IPV4
```

Verify:

```bash
docker --version
```

If it works **without sudo**, setup is correct.

---

# 4️⃣ Install Git

```bash
sudo apt install git
```

---

# 5️⃣ Clone Backend Repository

```bash
git clone <repo_url>.git
cd your_repo
```

Switch to production branch:

```bash
git checkout prod
```

---

# 6️⃣ Add Environment Variables

Create production env file.

```bash
vim .env.prod
```

Paste environment variables.

---

# 7️⃣ Run Backend with Docker

```bash
docker compose -f docker-compose.prod.yml up --build -d
```

Run in background using `-d`.

---

# 8️⃣ Current Architecture (Not Ideal)

```
Browser → EC2:8000 → Gunicorn → Django
```

Problems:

* Gunicorn exposed to internet
* No reverse proxy
* No HTTPS support
* No static optimization
* No frontend hosting

We fix this using **Nginx reverse proxy**.

---

# 9️⃣ Update EC2 Security Group

Remove public access to port **8000**.

Delete inbound rule:

```
Custom TCP 8000 → Anywhere
```

Reason:

Gunicorn should **not be accessible from internet**.

Only **Nginx inside EC2** should access it.

---

# 🔟 Allow HTTP Traffic

Add inbound rule:

| Type | Port | Source   |
| ---- | ---- | -------- |
| HTTP | 80   | Anywhere |

Now traffic becomes:

```
Browser → EC2:80 → Nginx
```

---

# 1️⃣1️⃣ Install Nginx

```bash
sudo apt install nginx
```

Start Nginx:

```bash
sudo systemctl start nginx
```

Enable on boot:

```bash
sudo systemctl enable nginx
```

---

# 1️⃣2️⃣ Test Nginx

Open browser:

```
http://YOUR_PUBLIC_IP
```

You should see:

```
Welcome to nginx!
```

⚠️ Do **not use https yet**.

---

# 1️⃣3️⃣ Connect Nginx → Gunicorn

Goal architecture:

```
Browser
   ↓
Nginx (80)
   ↓
Gunicorn (8000)
   ↓
Django
```

---

# 1️⃣4️⃣ Create Nginx App Config

```bash
sudo nano /etc/nginx/sites-available/medibooker
```

Add configuration:

```
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

# 1️⃣5️⃣ Enable Site

Create symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/medibooker /etc/nginx/sites-enabled/
```

Explanation:

```
ln = link
-s = symbolic link
```

---

# 1️⃣6️⃣ Remove Default Site

```bash
sudo rm /etc/nginx/sites-enabled/default
```

---

# 1️⃣7️⃣ Test Nginx Configuration

```bash
sudo nginx -t
```

Expected output:

```
syntax is ok
test is successful
```

---

# 1️⃣8️⃣ Reload Nginx

```bash
sudo systemctl reload nginx
```

Test again:

```
http://YOUR_PUBLIC_IP
```

Now Django should work **without :8000**.

---

# 1️⃣9️⃣ Why 127.0.0.1?

```
127.0.0.1 = localhost
```

Gunicorn runs on **port 8000 inside EC2**.

Nginx forwards requests internally.

Flow:

```
Browser → 80 → Nginx
            ↓
      127.0.0.1:8000 → Gunicorn
```

---

# 2️⃣0️⃣ Proxy Headers Explanation

### proxy_pass

```
proxy_pass http://127.0.0.1:8000;
```

Forward request to Gunicorn.

---

### Host Header

```
proxy_set_header Host $host;
```

Preserves original request host.

Without it Django might fail **ALLOWED_HOSTS check**.

---

### Real Client IP

```
proxy_set_header X-Real-IP $remote_addr;
```

Preserves user IP.

Otherwise Django would see:

```
127.0.0.1
```

for every request.

---

### Proxy Chain Header

```
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

Keeps track of client IP across **multiple proxies/load balancers**.

---

# 2️⃣1️⃣ Frontend Deployment

Clone frontend repo.

```bash
git clone <repo_link>
cd frontend
```

Checkout production branch:

```bash
git checkout prod
```

Setup environment variables.

---

# 2️⃣2️⃣ Build Frontend

```bash
npm install
npm run build
```

---

# 2️⃣3️⃣ Move Build to Nginx Directory

Create safe static folder:

```bash
sudo mkdir -p /var/www/medibooker
```

Clean existing files:

```bash
sudo rm -rf /var/www/medibooker/*
```

Copy build files:

```bash
sudo cp -r dist/* /var/www/medibooker/
```

Reason:

Serving from repo folder risks exposing:

```
.env
source code
```

---

# 2️⃣4️⃣ Update Nginx Config for Frontend + API

Edit config:

```bash
sudo vim /etc/nginx/sites-available/medibooker
```

```
server {
    listen 80;
    server_name _;

    root /var/www/medibooker;
    index index.html;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location / {
        try_files $uri /index.html;
    }
}
```

Architecture now:

```
Browser
   ↓
Nginx
 ├── /        → Frontend
 └── /api/    → Backend
```

---

# 2️⃣5️⃣ Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# 2️⃣6️⃣ Current Manual Deployment

```
Push Code
↓
SSH into EC2
↓
git pull
↓
Rebuild frontend
↓
Rebuild docker backend
↓
Reload nginx
```

We automate this using **CI/CD**.

---

# 🚀 CI/CD With GitHub Actions

Goal:

```
Push → GitHub
↓
GitHub Actions
↓
SSH into EC2
↓
Deploy automatically
```

---

# 2️⃣7️⃣ Generate SSH Key For GitHub

Run on **local machine**:

```bash
ssh-keygen -t ed25519 -C "github-deploy"
```

Name the key:

```
github_ec2_deploy
```

Files created:

```
github_ec2_deploy
github_ec2_deploy.pub
```

Leave passphrase empty.

---

# 2️⃣8️⃣ Add Public Key to EC2

Open public key:

```bash
cat github_ec2_deploy.pub
```

Copy content.

On EC2:

```bash
mkdir -p ~/.ssh
vim ~/.ssh/authorized_keys
```

Paste the key.

---

# 2️⃣9️⃣ Add GitHub Secrets

Go to:

```
Repo → Settings → Secrets → Actions
```

Add:

### Secret 1

```
EC2_SSH_KEY
```

Value:

```
contents of github_ec2_deploy
```

---

### Secret 2

```
EC2_HOST
```

Value:

```
YOUR_PUBLIC_IP
```

---

# 3️⃣0️⃣ Backend CI/CD Workflow

File:

```
backend/.github/workflows/cd.yml
```

```yaml
name: Deploy Backend to EC2

on:
  push:
    branches:
      - prod

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/backend
            git fetch origin
            git checkout prod
            git reset --hard origin/prod
            docker-compose -f docker-compose.prod.yml down
            docker-compose -f docker-compose.prod.yml up --build -d
```

---

# 3️⃣1️⃣ Frontend CI/CD Workflow

File:

```
frontend/.github/workflows/cd.yml
```

```yaml
name: Deploy Frontend to EC2

on:
  push:
    branches:
      - prod

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/frontend
            git fetch origin
            git checkout prod
            git reset --hard origin/prod
            npm install
            npm run build
            sudo rm -rf /var/www/medibooker/*
            sudo cp -r dist/* /var/www/medibooker/
            sudo systemctl reload nginx
```

---

# 🏆 Final Architecture

```
Browser
   ↓
Nginx (Port 80)
 ├── /        → Frontend (Static files)
 └── /api/    → Gunicorn → Django

Docker
 └── Backend containers
```

---

# 🎯 What You Achieved

You now have:

✅ EC2 infrastructure
✅ Dockerized Django backend
✅ Gunicorn production server
✅ Nginx reverse proxy
✅ Static frontend hosting
✅ API routing with `/api/`
✅ Backend CI/CD
✅ Frontend CI/CD

This is **real production-grade deployment**.

---
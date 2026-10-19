
# 🚀 Full Production Architecture (Dockerized + CI/CD)


# 🧱 1. Evolution of Architecture

---

## ❌ Old Architecture (Hybrid)

### Structure

```
EC2 (Ubuntu)
 ├── Nginx (OS)
 └── Docker
      └── Django + Gunicorn
```

### Flow

```
Browser → Nginx (OS) → Gunicorn (Docker) → Django
```

---

## ⚠️ Problems

* ❌ Server-dependent setup
* ❌ Manual installation (nginx, docker, configs)
* ❌ Not portable across environments
* ❌ Hard to scale

---

## ✅ New Architecture (Fully Containerized)

### Structure

```
EC2
└── Docker
    ├── nginx container (port 80)
    └── web container (Django + Gunicorn)
```

### Flow

```
Browser
   ↓
Nginx container (80)
   ↓
web container (8000)
   ↓
Django
```

---

# 🎯 2. Key Benefits

---

### 🔁 Reproducibility

```bash
docker compose up
```

---

### 🚫 No Server Dependency

* EC2 = Docker host only

---

### 📈 Scalability

* Kubernetes
* ECS
* Docker Swarm

---

### ⚙️ CI/CD Ready

* Full infra defined in code

---

# 📁 3. Project Structure

```
backend/
├── Dockerfile
├── docker-compose.yml          # local dev
├── docker-compose.prod.yml     # production
├── .env                        # compose variables
├── .env.prod                   # app variables
├── nginx/
│    └── nginx.conf
└── app/
```

---

# 🌐 4. Nginx Configuration

📁 `nginx/nginx.conf`

```nginx
events {}

http {
    server {
        listen 80;

        # Frontend
        root /var/www/medibooker;
        index index.html;

        location / {
            try_files $uri /index.html;
        }

        # Backend API
        location /api/ {
            proxy_pass http://web:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

---

# 🐳 5. Production Compose

📁 `docker-compose.prod.yml`

```yaml
version: "3.9"

services:
  web:
    image: ${DOCKER_USERNAME}/medibooker-backend:latest
    container_name: django_app
    env_file:
      - .env.prod
    restart: always

  nginx:
    image: nginx:latest
    container_name: nginx_proxy
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - /var/www/medibooker:/var/www/medibooker
    depends_on:
      - web
```

---

# 🧠 6. Environment Variables (CRITICAL)

---

## ⚠️ Rule

| File        | Purpose                     |
| ----------- | --------------------------- |
| `.env`      | Used by Docker Compose      |
| `.env.prod` | Used inside container (app) |

---

## ✅ `.env`

```env
DOCKER_USERNAME=harveydev
```

---

## ✅ `.env.prod`

```env
DATABASE_URL=...
SECRET_KEY=...
DEBUG=False
```

---

# 🐳 7. Docker Image Workflow

---

## 🏗️ Build Image

```bash
docker build -t harveydev/medibooker-backend:latest .
```

---

## 🔐 Login

```bash
docker login
```

---

## 📤 Push

```bash
docker push harveydev/medibooker-backend:latest
```

---

# 🔄 8. Production Deployment Flow

```
Push to GitHub (prod)
        ↓
GitHub Actions (CI)
        ↓
Build Docker Image
        ↓
Push to DockerHub
        ↓
EC2 pulls image
        ↓
Run containers
```

---

# 🤖 9. GitHub Actions (CI/CD)

```yaml
name: CI/CD Backend

on:
  push:
    branches:
      - prod

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # Checkout
      - name: Checkout code
        uses: actions/checkout@v3

      # Docker login
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # Build image
      - name: Build Docker Image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/medibooker-backend:latest .

      # Push image
      - name: Push Docker Image
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/medibooker-backend:latest

      # Deploy
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/medibooker-backend
            git fetch origin
            git checkout prod
            sudo git reset --hard origin/prod

            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d
```

---

# 🔐 10. Required GitHub Secrets

```
DOCKER_USERNAME
DOCKER_PASSWORD
EC2_HOST
EC2_SSH_KEY
```

---

# 🚨 11. Deployment Change (IMPORTANT)

---

## ❌ Old (Wrong)

```bash
docker compose up --build
```

---

## ✅ New (Correct)

```bash
docker compose pull
docker compose up -d
```

---

# 🧠 12. Final Mental Model

```
Write Code
   ↓
Push to GitHub
   ↓
CI builds Docker image
   ↓
Push to DockerHub
   ↓
EC2 pulls image
   ↓
Runs containers
```

---

# 🔮 13. Next Improvements

* 🔐 HTTPS (Let's Encrypt)
* ☁️ S3 for media
* ❤️ Health checks
* 🏷️ Image tagging (latest + commit hash)
* ⚡ Zero downtime deployment

---

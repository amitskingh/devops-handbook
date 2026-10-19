# 🐳 Docker Revision (Commands + Concepts)

## 1️⃣ Check Docker Installation

```bash
docker --version
docker info
```

Check if Docker daemon is running.

---

# 2️⃣ Docker Image Commands

## List Images

```bash
docker images
```

or

```bash
docker image ls
```

Shows:

| Column     | Meaning    |
| ---------- | ---------- |
| REPOSITORY | Image name |
| TAG        | Version    |
| IMAGE ID   | Unique ID  |
| CREATED    | When built |
| SIZE       | Image size |

---

## Build Image

```bash
docker build -t image-name:tag .
```

Example:

```bash
docker build -t medibooker-backend:v1 .
```

Explanation:

| Part                 | Meaning             |
| -------------------- | ------------------- |
| `-t`                 | tag image           |
| `medibooker-backend` | image name          |
| `v1`                 | version             |
| `.`                  | Dockerfile location |

---

## Remove Image

```bash
docker rmi image-name
```

or

```bash
docker rmi image-id
```

Force remove:

```bash
docker rmi -f image-id
```

---

# 3️⃣ Docker Container Commands

## Run Container

```bash
docker run image-name
```

Example:

```bash
docker run medibooker-backend:v1
```

---

## Run Container with Port Mapping

```bash
docker run -p hostPort:containerPort image-name
```

Example:

```bash
docker run -p 8000:8000 medibooker-backend:v1
```

Meaning:

| Host | Container |
| ---- | --------- |
| 8000 | 8000      |

Access via:

```
localhost:8000
```

---

## Run in Detached Mode

```bash
docker run -d -p 8000:8000 medibooker-backend:v1
```

`-d` → run in background

---

## Name the Container

```bash
docker run -d -p 8000:8000 --name backend medibooker-backend:v1
```

---

# 4️⃣ Container Management

## List Running Containers

```bash
docker ps
```

---

## List All Containers

```bash
docker ps -a
```

Shows stopped containers also.

---

## Stop Container

```bash
docker stop container-id
```

or

```bash
docker stop container-name
```

---

## Start Container

```bash
docker start container-id
```

---

## Restart Container

```bash
docker restart container-id
```

---

## Remove Container

```bash
docker rm container-id
```

Force remove:

```bash
docker rm -f container-id
```

---

# 5️⃣ Container Debugging

## See Logs

```bash
docker logs container-id
```

Live logs:

```bash
docker logs -f container-id
```

---

## Execute Command Inside Container

```bash
docker exec -it container-id bash
```

or

```bash
docker exec -it container-id sh
```

Example:

```bash
docker exec -it backend bash
```

---

# 6️⃣ Dockerfile Commands (Revision)

Example:

```dockerfile
FROM node:20

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8000

CMD ["npm","start"]
```

Meaning:

| Command | Purpose           |
| ------- | ----------------- |
| FROM    | base image        |
| WORKDIR | working directory |
| COPY    | copy files        |
| RUN     | execute commands  |
| EXPOSE  | container port    |
| CMD     | default command   |

---

# 7️⃣ Docker Networks (Important)

List networks:

```bash
docker network ls
```

Create network:

```bash
docker network create my-network
```

Run container in network:

```bash
docker run -d --network my-network nginx
```

---

# 8️⃣ Docker Volume (Persist Data)

Create volume:

```bash
docker volume create my-volume
```

Run container with volume:

```bash
docker run -v my-volume:/data nginx
```

---

# 9️⃣ Clean Up Commands

Remove stopped containers:

```bash
docker container prune
```

Remove unused images:

```bash
docker image prune
```

Remove everything unused:

```bash
docker system prune
```

---

# 🔟 Docker Compose (Quick Revision)

Start services:

```bash
docker compose up
```

Detached:

```bash
docker compose up -d
```

Stop:

```bash
docker compose down
```

Rebuild:

```bash
docker compose up --build
```

---

# 🧠 Real Development Workflow

Typical backend workflow:

```bash
docker build -t backend:v1 .

docker run -d -p 8000:8000 backend:v1

docker ps

docker logs backend

docker exec -it backend bash
```

---

# 🚀 Commands You Used For Kubernetes

Because you used **Minikube image build**:

```bash
minikube image build -t medibooker-backend:v1 .
```

Instead of:

```bash
docker build
```

Because image is built **inside minikube docker daemon**.

---

# ⭐ Most Important Docker Commands (Remember These)

```
docker build
docker run
docker ps
docker stop
docker rm
docker rmi
docker logs
docker exec
docker images
docker system prune
```

---

# DevOps Handbook

Personal notes and cheatsheets for Docker, Kubernetes, AWS deployments, CI/CD, GitHub, and code formatting.

---

## Repository Structure

```
devops-handbook/
├── aws/          # EC2 deployments, Nginx, production architecture
├── docker/       # Docker & Docker Compose
├── github/       # GitHub repository settings
├── kubernetes/   # Kubernetes concepts & local setup
├── tooling/      # Linting & formatting (ESLint, Prettier, Ruff)
└── README.md
```

---

## 🐳 Docker

- [Docker Basics](./docker/docker-basics.md) — Commands and core concepts
- [Docker Compose Commands](./docker/docker-compose-commands.md) — Common Compose workflows
- [Docker Compose Local](./docker/docker-compose-local.md) — Local development compose file
- [Docker Compose Production](./docker/docker-compose-production.md) — Production compose file and Dockerfile notes
- [Docker Compose Variable Escaping](./docker/docker-compose-variable-escaping.md) — Why `$$` is needed in Compose

## ☸️ Kubernetes

- [Kubernetes Basics](./kubernetes/kubernetes-basics.md) — Core concepts and orchestration overview
- [Kubernetes Naming Conventions](./kubernetes/kubernetes-naming-conventions.md) — Labels, ports, and name matching rules
- [Kubernetes Minikube Setup](./kubernetes/kubernetes-minikube-setup.md) — Local cluster setup with Minikube

## ☁️ AWS (Deployment & CI/CD)

- [Fullstack Deployment (EC2 + Docker + CI/CD)](./aws/fullstack-deployment-ec2-docker-cicd.md) — End-to-end EC2, Docker, Nginx, and CI/CD guide
- [Production Architecture (Docker + CI/CD)](./aws/production-architecture-docker-cicd.md) — Evolution to a fully containerized EC2 setup
- [Nginx Deployment Permissions](./aws/nginx-deployment-permissions.md) — Fixing Nginx / GitHub Actions frontend permissions on EC2

## 🐙 GitHub

- [GitHub Branch Protection](./github/github-branch-protection.md) — Branch rulesets vs classic protection

## 🧹 Code Formatting & Tooling

- [Fastify + TypeScript (ESLint + Prettier)](./tooling/fastify-typescript-eslint-prettier.md) — Lint, format, and Husky setup
- [JavaScript / React Linting](./tooling/javascript-react-linting.md) — ESLint + Prettier for JS/TS React projects
- [Python Ruff Formatting](./tooling/python-ruff-formatting.md) — Ruff setup, usage, and GitHub Actions lint workflow

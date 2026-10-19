# Kubernetes Basics (Core Concepts)

## What is Kubernetes

**Kubernetes (K8s)** is an **open-source container orchestration system** used to:

* Deploy applications
* Scale applications
* Manage containers automatically
* Maintain high availability

Instead of manually managing containers on servers, Kubernetes **automatically manages them for you**.

### Example Responsibilities

Kubernetes can automatically:

* Start containers
* Restart failed containers
* Scale applications up or down
* Distribute traffic across containers
* Manage configuration and secrets

---

# Why Kubernetes is Needed

Tools like Docker allow you to **run containers**, but they do not manage large systems well.

Example problem without Kubernetes:

```
You have 50 containers running on multiple servers.

If a container crashes:
→ Someone must manually restart it.

If traffic increases:
→ Someone must manually start more containers.

If a server fails:
→ Containers must be moved manually.
```

Kubernetes solves this by **automating container management**.

---

# Kubernetes Architecture

Kubernetes consists of **two major parts**.

```
Kubernetes Cluster
│
├── Control Plane (Brain)
│
└── Worker Nodes (Where applications run)
```

---

# Control Plane

The **Control Plane** manages the entire cluster.

It decides:

* Where containers should run
* How many replicas should run
* When containers should restart
* How networking works

Main components inside control plane:

| Component          | Role                      |
| ------------------ | ------------------------- |
| API Server         | Entry point to Kubernetes |
| Scheduler          | Assigns pods to nodes     |
| Controller Manager | Maintains desired state   |
| etcd               | Cluster database          |

---

## API Server

The **API Server** is the main interface of Kubernetes.

All commands go through it.

Example:

```
kubectl apply -f deployment.yaml
```

The API server receives this request and stores it in **etcd**.

---

## Scheduler

The scheduler decides:

```
Which node should run which pod
```

Example:

```
Pod created
     ↓
Scheduler chooses a worker node
     ↓
Pod runs on that node
```

---

## etcd

`etcd` is a **key-value database**.

It stores the entire cluster state.

Example stored data:

* Pods
* Services
* Deployments
* ConfigMaps
* Secrets

If etcd is lost → cluster state is lost.

---

# Worker Nodes

Worker nodes are machines where **applications actually run**.

Each node contains:

```
Node
│
├── kubelet
├── container runtime
└── kube-proxy
```

---

## Kubelet

Kubelet is an **agent running on each node**.

It communicates with the control plane and ensures:

```
containers are running correctly
```

Example:

```
Control plane: run 3 pods
kubelet: ensures pods are running
```

---

## Container Runtime

This runs containers.

Examples:

* Docker
* containerd
* CRI-O

In Minikube, Docker is commonly used.

---

## kube-proxy

`kube-proxy` manages **networking between pods and services**.

It enables:

```
Pod → Pod communication
Pod → Service communication
```

---

# Kubernetes Cluster

A **Kubernetes cluster** is a group of nodes managed together.

Example:

```
Cluster
│
├── Control Plane
│
├── Worker Node 1
│
├── Worker Node 2
│
└── Worker Node 3
```

Your application runs across these nodes.

---

# Containers

Containers package an application with:

* code
* dependencies
* runtime

Example container:

```
Node.js app
+ dependencies
+ OS libraries
```

Containers are created from **Docker images**.

---

# Pod

A **Pod** is the **smallest deployable unit in Kubernetes**.

Pods contain **one or more containers**.

Example:

```
Pod
│
└── Container (Node.js app)
```

Usually:

```
1 pod = 1 container
```

But sometimes:

```
Pod
├── Main container
└── Sidecar container
```

---

## Pod Characteristics

Pods are:

* ephemeral (temporary)
* replaceable
* automatically recreated

Example:

```
Pod crashes
↓
Kubernetes creates a new pod
```

---

# Deployment

A **Deployment** manages pods.

It ensures:

```
Desired number of pods are running
```

Example deployment:

```
replicas: 3
```

Kubernetes ensures:

```
3 pods are always running
```

If one pod crashes:

```
Kubernetes automatically creates a new pod
```

---

## Deployment Benefits

Deployments provide:

* scaling
* rolling updates
* rollback capability
* self-healing

---

# ReplicaSet

A **ReplicaSet** ensures a specified number of pod replicas are running.

Example:

```
replicas: 3
```

ReplicaSet ensures:

```
3 pods always exist
```

Deployment **automatically manages ReplicaSets**, so developers rarely interact with them directly.

---

# Service

Pods are **temporary and change IP addresses**.

So Kubernetes introduces **Services** to provide stable networking.

A Service provides:

```
Stable IP
Stable DNS name
Load balancing
```

Example:

```
backend-service
```

Pods can call:

```
http://backend-service:8000
```

Even if pods change.

---

## Service Types

Common service types:

| Type         | Purpose                        |
| ------------ | ------------------------------ |
| ClusterIP    | Internal communication         |
| NodePort     | Expose service outside cluster |
| LoadBalancer | Cloud load balancer            |
| ExternalName | External DNS mapping           |

---

# ConfigMap

A **ConfigMap** stores configuration data.

Example:

```
API URLs
Feature flags
Environment variables
```

Example data:

```
VITE_API_URL=http://backend-service:8000
```

Used by pods as environment variables.

---

# Secret

A **Secret** stores sensitive data.

Examples:

* API keys
* passwords
* database credentials

Example:

```
DATABASE_PASSWORD
JWT_SECRET
```

Secrets are encoded and injected into pods.

---

# Namespace

Namespaces divide a cluster into **logical groups**.

Example:

```
Cluster
│
├── default
├── dev
├── staging
└── production
```

This helps isolate environments.

---

# Kubernetes Networking Model

Kubernetes networking follows these rules:

```
Every pod has its own IP
Pods can communicate with each other
Services provide stable networking
```

Example:

```
Frontend Pod
     ↓
backend-service
     ↓
Backend Pod
```

---

# kubectl

`kubectl` is the **command line tool** used to interact with Kubernetes.

Examples:

### View nodes

```
kubectl get nodes
```

### View pods

```
kubectl get pods
```

### Apply configuration

```
kubectl apply -f deployment.yaml
```

### Delete resources

```
kubectl delete -f deployment.yaml
```

---

# Minikube

Minikube runs **a local Kubernetes cluster** on your machine.

Useful for:

* learning Kubernetes
* local development
* testing deployments

Example:

```
minikube start
```

Creates a single-node Kubernetes cluster locally.

---

# Basic Kubernetes Flow

Typical workflow:

```
Developer writes Dockerfile
        ↓
Build Docker Image
        ↓
Write Kubernetes YAML
        ↓
kubectl apply
        ↓
Kubernetes creates pods
        ↓
Services expose application
```

---

If you want, next we can write the **second part of your notes**, which usually comes after this:

**Kubernetes Objects Explained Deeply**

That includes:

* Deployment YAML breakdown
* Service YAML breakdown
* ConfigMap YAML breakdown
* Secret YAML breakdown
* Label & selector mechanics
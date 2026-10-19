# Kubernetes Setup for MediBooker (Minikube)

## 1. Start Minikube

Start a local Kubernetes cluster using Docker as the driver.

```bash
minikube start --driver=docker
```

Verify the cluster:

```bash
kubectl get nodes
```

Expected output:

```
NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane
```

---

# 2. Build Docker Images Inside Minikube

Ensure **both frontend and backend projects contain Dockerfiles**.

### Build Backend Image

```bash
minikube image build -t medibooker-backend:v1 ./medibooker-backend
```

### Build Frontend Image

```bash
minikube image build -t medibooker-frontend:v1 ./medibooker-frontend
```

### Verify Images

```bash
minikube image ls
```

---

# 3. Create Kubernetes Manifests

Create a directory for Kubernetes configuration files.

```bash
mkdir k8s
cd k8s
```

---

# 4. Backend Deployment

Create `backend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: medibooker-backend

spec:
  replicas: 2

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:
        - name: backend
          image: medibooker-backend:v1
          imagePullPolicy: Never
          ports:
            - containerPort: 8000
```

Apply deployment:

```bash
kubectl apply -f backend-deployment.yaml
```

Verify pods:

```bash
kubectl get pods
```

You should see **2 backend pods running**.

---

# 5. Backend Service

Create `backend-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service

spec:
  type: ClusterIP

  selector:
    app: backend

  ports:
    - port: 8000
      targetPort: 8000
```

Apply:

```bash
kubectl apply -f backend-service.yaml
```

Backend becomes accessible **inside the cluster** at:

```
http://backend-service:8000
```

---

# 6. Frontend Deployment

Create `frontend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: medibooker-frontend

spec:
  replicas: 2

  selector:
    matchLabels:
      app: frontend

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
        - name: frontend
          image: medibooker-frontend:v1
          imagePullPolicy: Never
          ports:
            - containerPort: 3000
```

Apply:

```bash
kubectl apply -f frontend-deployment.yaml
```

---

# 7. Frontend Service (Expose to Browser)

Create `frontend-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service

spec:
  type: NodePort

  selector:
    app: frontend

  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30007
```

Apply:

```bash
kubectl apply -f frontend-service.yaml
```

---

# 8. Access the Application

Run:

```bash
minikube service frontend-service
```

Your browser will open automatically.

---

# 9. Application Architecture

The request flow becomes:

```
Browser
   ↓
Frontend Service (NodePort)
   ↓
Frontend Pod
   ↓
Backend Service (ClusterIP)
   ↓
Backend Pod
```

---

# 10. Verify Kubernetes Resources

### Check Deployments

```bash
kubectl get deployments
```

### Check Pods

```bash
kubectl get pods
```

### Check Services

```bash
kubectl get svc
```

---

# 11. Create ConfigMap (Frontend Environment Variables)

Create `frontend-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-config

data:
  VITE_API_URL: "http://backend-service:8000"
```

Apply:

```bash
kubectl apply -f frontend-configmap.yaml
```

Verify:

```bash
kubectl get configmap
```

Reapply the frontend deployment:

```bash
kubectl apply -f frontend-deployment.yaml
```

Kubernetes will recreate the pods.

---

# 12. Verify Environment Variable Inside Pod

Find frontend pod:

```bash
kubectl get pods
```

Enter the pod:

```bash
kubectl exec -it <frontend-pod-name> -- /bin/sh
```

Check environment variables:

```bash
printenv | grep VITE
```

---

# 13. Create Secret for Backend Environment Variables

Create secret from `.env.local`

```bash
kubectl create secret generic backend-secret --from-env-file=.env.local
```

---

# 14. Update Backend Deployment to Use Secret

Modify `backend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: medibooker-backend

spec:
  replicas: 2

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:
        - name: backend
          image: medibooker-backend:v1
          imagePullPolicy: Never

          ports:
            - containerPort: 8000

          envFrom:
            - secretRef:
                name: backend-secret
```

Redeploy:

```bash
kubectl apply -f backend-deployment.yaml
```

---

# 15. Verify Backend Environment Variables

Find pod:

```bash
kubectl get pods
```

Enter pod:

```bash
kubectl exec -it <backend-pod-name> -- /bin/sh
```

Check variables:

```bash
printenv
```

---

# 16. Updating Your Application

Kubernetes **does not automatically detect code changes**.

## 1️⃣ If Application Code Changes

Rebuild image:

```bash
minikube image build -t medibooker-backend:v1 ./medibooker-backend
```

Restart deployment:

```bash
kubectl rollout restart deployment medibooker-backend
```

Or update image version:

```bash
kubectl set image deployment/medibooker-backend backend=medibooker-backend:v2
```

---

## 2️⃣ If Kubernetes YAML Changes

Simply reapply the file:

```bash
kubectl apply -f backend-deployment.yaml
```

---

## 3️⃣ If ConfigMap or Secret Changes

Update resource:

```bash
kubectl apply -f configmap.yaml
```

Recreate secret:

```bash
kubectl create secret generic backend-secret \
--from-env-file=.env.local \
--dry-run=client -o yaml | kubectl apply -f -
```

Restart pods:

```bash
kubectl rollout restart deployment medibooker-backend
```

---

# 17. Quick Decision Guide

| Change Type             | Action                           |
| ----------------------- | -------------------------------- |
| Backend / Frontend Code | Rebuild Docker image             |
| Dockerfile              | Rebuild Docker image             |
| Deployment YAML         | `kubectl apply`                |
| Service YAML            | `kubectl apply`                |
| ConfigMap               | `kubectl apply` + restart pods |
| Secret                  | recreate secret + restart pods   |

---

## Your Notes Are Good 👍

What you did here is **exactly how DevOps engineers learn Kubernetes**:

* deploy
* break
* debug
* document
* refine

This is **real learning**, not tutorial copying.

---

If you want, I can also show you **how real production Kubernetes manifests look in companies** (with **Ingress, namespaces, resources, health checks, autoscaling, etc.**) so your project becomes **production-grade instead of tutorial-grade**.

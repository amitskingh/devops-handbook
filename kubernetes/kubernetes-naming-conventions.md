Great idea. Understanding **what names must match and what can differ in Kubernetes YAML** is the difference between *copy-pasting YAML* and  *actually understanding Kubernetes* .

I'll break this into **3 critical matching rules** that confuse most beginners:

1. **Label Matching (Deployment ↔ Service)**
2. **Port Mapping (Container ↔ Service ↔ NodePort)**
3. **Resource Naming Conventions**

---

# 1️⃣ Label Matching (MOST IMPORTANT)

In Kubernetes,  **Services find Pods using labels** .

This is the  **#1 thing that must match correctly** .

### Deployment

```yaml
metadata:
  name: medibooker-backend

spec:
  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend
```

### Service

```yaml
spec:
  selector:
    app: backend
```

### What MUST Match

| Field                         | Must Match                        |
| ----------------------------- | --------------------------------- |
| `spec.selector.matchLabels` | `spec.template.metadata.labels` |
| `service.spec.selector`     | `pod labels`                    |

### Visual Flow

```
Deployment creates pods with label:

app: backend
      ↑
Service selector searches for pods with:

app: backend
```

If labels don't match:

```
Service → NO POD FOUND
```

Then your service returns  **no response** .

---

# 2️⃣ Naming Conventions (Important but flexible)

### Deployment Name

```yaml
metadata:
  name: medibooker-backend
```

Used for:

```
kubectl rollout restart deployment medibooker-backend
```

It  **does NOT need to match anything else** .

---

### Service Name

```yaml
metadata:
  name: backend-service
```

This name becomes the  **internal DNS name inside Kubernetes** .

Pods can call it like this:

```
http://backend-service:8000
```

So the important rule is:

```
Service name = internal hostname
```

Example:

| Service Name    | Internal URL                                             |
| --------------- | -------------------------------------------------------- |
| backend-service | [http://backend-service:8000](http://backend-service:8000/) |
| auth-service    | [http://auth-service:8000](http://auth-service:8000/)       |
| payment-service | [http://payment-service:8000](http://payment-service:8000/) |

---

# 3️⃣ Port Mapping (VERY IMPORTANT)

Ports confuse many developers.

Let's break it clearly.

---

## Container Port

Inside **Deployment**

```yaml
ports:
  - containerPort: 8000
```

This is the port  **your application runs on inside the container** .

Example:

```
FastAPI / Django / Express server → port 8000
```

---

## Service Port

Inside **Service**

```yaml
ports:
  - port: 8000
```

This is the port  **other pods use to access the service** .

Example:

```
http://backend-service:8000
```

---

## Target Port

```yaml
targetPort: 8000
```

This tells Kubernetes:

```
Service port  → container port
```

Example mapping:

```
Service:8000  → Pod:8000
```

---

## NodePort

Used only when  **exposing service outside cluster** .

```yaml
nodePort: 30007
```

This becomes the  **port on the Minikube node** .

Example:

```
localhost:30007
```

---

# 4️⃣ Complete Port Flow

Example frontend service:

```yaml
ports:
  - port: 3000
    targetPort: 3000
    nodePort: 30007
```

Flow:

```
Browser
   ↓
NodePort 30007
   ↓
Service port 3000
   ↓
Pod container port 3000
```

---

# 5️⃣ What MUST Match vs What Can Differ

| Field                      | Must Match?    | Explanation                     |
| -------------------------- | -------------- | ------------------------------- |
| Deployment labels          | ✅ Yes         | Service must find pods          |
| Service selector           | ✅ Yes         | Must match pod labels           |
| containerPort & targetPort | ✅ Usually     | Service must route to container |
| Service port               | ❌ Can differ  | Internal service port           |
| NodePort                   | ❌ Can differ  | External access port            |
| Deployment name            | ❌ Independent | Used only by kubectl            |
| Service name               | ❌ Independent | Used for DNS                    |

---

# 6️⃣ Example With Different Ports

This is  **valid Kubernetes config** :

```yaml
Deployment

containerPort: 5000
```

```yaml
Service

port: 80
targetPort: 5000
nodePort: 30007
```

Flow:

```
Browser:30007
    ↓
Service:80
    ↓
Pod:5000
```

---

# 7️⃣ Production Naming Convention (Used in Companies)

Most teams follow this structure:

### Deployment

```
<app>-deployment
```

Example

```
medibooker-backend-deployment
```

---

### Service

```
<app>-service
```

Example

```
medibooker-backend-service
```

---

### Labels

```
app: medibooker-backend
```

---

Example production standard:

```yaml
labels:
  app: medibooker-backend
```

Service:

```yaml
selector:
  app: medibooker-backend
```

---

# 8️⃣ Quick Visual Cheat Sheet

```
Deployment
   │
   │ creates
   ▼
Pods (labels)

app: backend
      ▲
      │ selector
      │
Service
```

Ports:

```
Browser
   │
NodePort (30007)
   │
Service Port (3000)
   │
TargetPort
   │
ContainerPort (3000)
```

---

# 9️⃣ The 3 Things You Must Always Check When Debugging

When Kubernetes  **doesn't work** , 90% of the time it's:

### 1️⃣ Labels mismatch

```
selector ≠ labels
```

---

### 2️⃣ Wrong port mapping

```
targetPort ≠ containerPort
```

---

### 3️⃣ Wrong service name in frontend env

```
http://backend-service:8000
```

not

```
http://localhost:8000
```

---

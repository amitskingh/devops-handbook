## 📄 `docker-compose.local.yml` (Clean Version)

```yaml
version: "3.9"

services:

  db:
    image: postgres:15
    container_name: postgres_db

    ports:
      - "5433:5432"

    env_file:
      - .env.local

    volumes:
      - postgres_data:/var/lib/postgresql/data

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5


  web:
    build: .
    container_name: django_app

    command: >
      sh -c "
      python manage.py makemigrations &&
      python manage.py migrate &&
      python manage.py runserver 0.0.0.0:8000
      "

    ports:
      - "8000:8000"

    env_file:
      - .env.local

    volumes:
      - .:/app

    depends_on:
      db:
        condition: service_healthy


volumes:
  postgres_data:
```

---

# 🧠 Simple Explanation (Keep It Simple)

Think of **Docker Compose** as:

```
Run multiple containers together with one file
```

This file starts **2 containers**:

```
1️⃣ Postgres Database
2️⃣ Django Backend
```

---

# 1️⃣ version

```yaml
version: "3.9"
```

Just tells Docker which **compose format** we are using.

Nothing special here.

---

# 2️⃣ services

```yaml
services:
```

Everything under this becomes a **container**.

Here we have:

```
db
web
```

So Docker will run **2 containers**.

---

# 3️⃣ Database Container

```yaml
db:
  image: postgres:15
```

Instead of building an image, Docker **pulls Postgres from Docker Hub**.

---

### Container Name

```yaml
container_name: postgres_db
```

Without this Docker gives random names.

Now container name will be:

```
postgres_db
```

---

### Port Mapping

```yaml
ports:
  - "5433:5432"
```

Format:

```
host_port : container_port
```

So:

```
localhost:5433 → postgres container:5432
```

---

### Environment Variables

```yaml
env_file:
  - .env.local
```

Docker loads variables from:

```
.env.local
```

Example inside file:

```
POSTGRES_DB=mydb
POSTGRES_USER=admin
POSTGRES_PASSWORD=password
```

---

# 4️⃣ Database Volume (Persistence)

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Meaning:

```
Docker volume → database storage folder
```

So:

```
postgres_data → /var/lib/postgresql/data
```

### Why?

If container deletes:

```
docker rm postgres_db
```

Database **still survives**.

Without volume:

```
container deleted = data gone
```

---

# 5️⃣ Healthcheck

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
```

Docker checks if **Postgres is ready**.

Because database needs time to start.

---

# 6️⃣ Web Container (Django)

```yaml
web:
  build: .
```

Docker builds image using:

```
Dockerfile in current folder
```

Same as running:

```
docker build .
```

---

### Container Name

```yaml
container_name: django_app
```

Container name becomes:

```
django_app
```

---

# 7️⃣ Command

```yaml
command: >
  sh -c "
  python manage.py makemigrations &&
  python manage.py migrate &&
  python manage.py runserver 0.0.0.0:8000
  "
```

When container starts it runs:

```
makemigrations
migrate
runserver
```

---

# 8️⃣ Port Mapping

```yaml
ports:
  - "8000:8000"
```

Access Django via:

```
localhost:8000
```

---

# 9️⃣ Hot Reloading (Important)

```yaml
volumes:
  - .:/app
```

This is **bind mount**.

Meaning:

```
your project folder → container /app
```

Example:

If you edit:

```
views.py
models.py
```

Container **sees the changes instantly**.

So Django reloads automatically.

This is called **hot reloading**.

Without this you would need:

```
docker build
docker run
```

every time you change code.

---

# 🔟 depends_on

```yaml
depends_on:
  db:
    condition: service_healthy
```

Meaning:

```
Start Django only after DB is ready
```

Without this Django might start **before Postgres**.

---

# 1️⃣1️⃣ Volumes Section (Bottom)

```yaml
volumes:
  postgres_data:
```

This **declares the volume**.

Even though Docker can create it automatically, we declare it for safety.

Why?

```
Clear
Readable
Safer
Team friendly
```

---

# ⚡ Final Mental Model

This single file runs:

```
Postgres container
        +
Django container
        +
Persistent database
        +
Hot reloading code
```

---

# 🚀 How To Run It

Start containers:

```bash
docker compose -f docker-compose.local.yml up
```

Detached mode:

```bash
docker compose -f docker-compose.local.yml up -d
```

Stop containers:

```bash
docker compose -f docker-compose.local.yml down
```

Remove containers + volumes:

```bash
docker compose -f docker-compose.local.yml down -v
```

---

# 🧩 One Small Concept That Is VERY Important

Inside containers **Django connects to database using**:

```
db:5432
```

NOT

```
localhost
```

Because Docker creates an **internal network** where services talk using their **service name**.

---

If you want, next I can show you **the 5 Docker Compose mistakes almost every backend developer makes** (these cause most production bugs).

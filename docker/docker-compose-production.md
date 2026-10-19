## 📄 `docker-compose.prod.yml` (Production)

```yaml
version: "3.9"

services:

  web:
    build: .
    container_name: django_app

    command: >
      sh -c "
      python manage.py migrate &&
      python manage.py collectstatic --noinput &&
      gunicorn medibooker.wsgi:application --bind 0.0.0.0:8000
      "

    ports:
      - "8000:8000"

    env_file:
      - .env.prod

    restart: always
```

---

# 🧠 Simple Explanation (Production Setup)

This compose file runs **one container only**:

```
Django App (Production)
```

Unlike local setup:

```
Local → Django + Postgres
Prod  → Only Django (DB usually external)
```

In production, the database is often:

* Managed DB (AWS RDS, Cloud SQL, etc.)
* Or separate container/server

---

# 1️⃣ version

```yaml
version: "3.9"
```

Compose file format version.

Nothing special.

---

# 2️⃣ services

```yaml
services:
```

Defines containers.

Here we have only **one service**:

```
web
```

---

# 3️⃣ build

```yaml
build: .
```

Docker builds the image using the **Dockerfile in the current folder**.

Equivalent to:

```bash
docker build .
```

---

# 4️⃣ container_name

```yaml
container_name: django_app
```

Container will be named:

```
django_app
```

instead of random Docker names.

---

# 5️⃣ command

```yaml
command: >
  sh -c "
  python manage.py migrate &&
  python manage.py collectstatic --noinput &&
  gunicorn medibooker.wsgi:application --bind 0.0.0.0:8000
  "
```

When container starts, it runs these commands.

### Step-by-step

#### 1️⃣ Run migrations

```
python manage.py migrate
```

Applies database migrations.

---

#### 2️⃣ Collect static files

```
python manage.py collectstatic --noinput
```

Moves all static files to:

```
/staticfiles
```

Used by production servers.

---

#### 3️⃣ Start Gunicorn

```
gunicorn medibooker.wsgi:application --bind 0.0.0.0:8000
```

Gunicorn is a **production WSGI server**.

Important difference:

| Dev         | Production |
| ----------- | ---------- |
| `runserver` | `gunicorn` |

`runserver` is **slow and unsafe** for production.

---

# 6️⃣ ports

```yaml
ports:
  - "8000:8000"
```

Format:

```
host_port : container_port
```

So users access app via:

```
EC2_IP:8000
```

---

# 7️⃣ env_file

```yaml
env_file:
  - .env.prod
```

Loads environment variables from:

```
.env.prod
```

Example:

```
DEBUG=False
SECRET_KEY=....
DATABASE_URL=...
```

Production environment variables.

---

# 8️⃣ restart policy

```yaml
restart: always
```

Important production feature.

Meaning:

If container crashes:

```
Docker automatically restarts it
```

Example situations:

| Situation      | Result             |
| -------------- | ------------------ |
| server reboot  | container restarts |
| app crash      | container restarts |
| Docker restart | container restarts |

Without this your app could **stay down**.

---

# ⚡ Major Differences (Local vs Prod)

| Feature        | Local        | Production       |
| -------------- | ------------ | ---------------- |
| DB container   | yes          | usually external |
| hot reload     | yes          | no               |
| bind mounts    | yes          | no               |
| server         | runserver    | gunicorn         |
| restart policy | no           | yes              |
| env file       | `.env.local` | `.env.prod`      |

---

# 🚀 Running Production Compose

Start container:

```bash
docker compose -f docker-compose.prod.yml up --build -d
```

Stop container:

```bash
docker compose -f docker-compose.prod.yml down
```

View logs:

```bash
docker compose -f docker-compose.prod.yml logs -f
```

---

# ⚠️ One Important Production Note

Currently your production setup exposes:

```
EC2_IP:8000
```

In real production we usually add:

```
Nginx
```

Architecture becomes:

```
Internet
   ↓
Nginx (port 80/443)
   ↓
Gunicorn (8000)
   ↓
Django
```

Nginx handles:

* HTTPS
* static files
* load balancing
* security

---

> # Why Section Starts Here:

---

# 1️⃣ `PYTHONDONTWRITEBYTECODE=1`

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
```

### What Python normally does

When Python runs, it creates:

```
__pycache__/
*.pyc
```

These are compiled bytecode files.

Inside Docker, they:

* Take space
* Are useless
* Will be regenerated anyway

### So this line tells Python:

> Do NOT create `.pyc` files.

### Why is this good in Docker?

Containers are:

* Ephemeral
* Rebuilt often
* Should stay clean and minimal

It prevents unnecessary file generation.

---

# 2️⃣ `PYTHONUNBUFFERED=1`

```dockerfile
ENV PYTHONUNBUFFERED=1
```

### What Python normally does

Python buffers output.

That means logs may not immediately appear.

In Docker, logs are streamed to:

```
docker logs container_name
```

If buffering is enabled:

* Logs appear late
* Logs appear in chunks
* Debugging becomes painful

### This line forces:

> Print logs immediately.

Very important in production.

---

# 3️⃣ Installing system dependencies

```dockerfile
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*
```

Now this is important.

---

## 🔹 `gcc`

Needed when installing Python packages that require compilation.

Example:

* psycopg2
* Pillow
* cryptography

If you don’t install gcc:

```
error: command 'gcc' failed
```

Build fails.

---

## 🔹 `libpq-dev`

This is required for PostgreSQL drivers like:

```
psycopg2
```

Without it:

```
pg_config executable not found
```

Build fails.

---

## 🔹 `rm -rf /var/lib/apt/lists/*`

This removes apt cache to:

* Reduce image size
* Keep Docker image lean

Best practice.

---

# 🧠 Do You Always Need These?

Depends.

If you're using:

* PostgreSQL → Yes
* ImageField with Pillow → Usually yes
* SQLite only → Maybe not
* psycopg2-binary → Might not need gcc

But in real production, it's safer to include them.

---

# 🎯 Summary

| Line                    | Why                          |
| ----------------------- | ---------------------------- |
| PYTHONDONTWRITEBYTECODE | Prevent useless `.pyc` files |
| PYTHONUNBUFFERED        | Real-time logs in Docker     |
| gcc                     | Compile Python packages      |
| libpq-dev               | Required for PostgreSQL      |
| rm -rf apt lists        | Reduce image size            |

---

# 🏆 For You Specifically

You are using:

* PostgreSQL ✔
* Image upload ✔
* Docker ✔

So yes — you should include them.

---

You’re now moving from “copy Dockerfile” to “understand Dockerfile”.

That’s how DevOps confidence builds.

If you want next, I can show you how to make your Docker image 30–40% smaller properly.

---

🔥 Very good question. This shows you’re thinking about build vs runtime.

Short answer:

👉 **`.dockerignore` does NOT block Docker from using `env_file` in docker-compose.**
👉 It only affects what gets copied into the image during `COPY . .`

Let’s break this clearly.

---

# 🧠 Important Concept

There are two different phases:

### 1️⃣ Build Time (Dockerfile)

This is when:

```dockerfile
COPY . .
```

happens.

`.dockerignore` affects ONLY this step.

---

### 2️⃣ Runtime (docker-compose)

This is when container starts and this happens:

```yaml
env_file:
  - .env.local
```

At this stage:

* Docker reads `.env.local`
* Injects variables into container environment
* No need for the file to be inside the image

---

# 🎯 So What Happens If You Add:

```dockerignore
.env
.env.*
```

### During build:

`.env.local` is NOT copied into image ✅

### During runtime:

Docker Compose still reads `.env.local` from your host machine and injects variables ✅

So everything still works.

---

# 🧠 Why This Is Actually GOOD

You do NOT want your `.env` file baked into the image.

Because:

* Image might be pushed to Docker Hub
* Secrets could leak
* Production security risk

Best practice:

❌ Never copy `.env` into image
✅ Inject environment variables at runtime

---

# 🔥 Clean Mental Model

| Thing             | Where It Lives                         |
| ----------------- | -------------------------------------- |
| `.env.local`      | On your host                           |
| Docker image      | Does NOT contain `.env`                |
| Running container | Receives variables from docker-compose |

---

# 🚀 Example

You have:

```yaml
services:
  web:
    build: .
    env_file:
      - .env.local
```

Docker:

1. Builds image (ignores .env)
2. Starts container
3. Reads .env.local
4. Injects variables

So inside container:

```bash
echo $POSTGRES_USER
```

Works.

Even though `.env.local` file is NOT inside container.

---

# 🏆 So Yes — You SHOULD Add `.dockerignore`

It will NOT break env injection.

It makes your image safer and cleaner.

---

You just crossed into real container security understanding.

If you want next, I can explain:

👉 Difference between `ARG` and `ENV` in Docker
👉 When to use docker secrets
👉 How production EC2 env injection works

That’s next-level DevOps thinking.

---


Perfect. These are two **core production commands**.
If you understand these deeply, you understand Django deployment.

Let’s break them properly.

---

# 1️⃣ `python manage.py collectstatic --noinput`

## 🧠 What Problem Does This Solve?

In development:

```text
Django serves static files automatically.
```

In production:

```text
Django does NOT serve static files.
```

So we need to collect all static files into one folder.

---

## 🔥 What `collectstatic` Does

It gathers static files from:

* Your apps (`app/static/`)
* Django admin
* Third-party packages

And copies them into:

```python
STATIC_ROOT
```

Example:

```python
STATIC_ROOT = BASE_DIR / "staticfiles"
```

After running:

```
/app/staticfiles/
    admin/
    css/
    js/
```

Now Nginx (later) can serve everything from one location.

---

## ⚙️ What `--noinput` Means

Normally, Django asks:

```
This will overwrite existing files. Are you sure? (yes/no)
```

In Docker, we cannot answer interactively.

So:

```bash
--noinput
```

Means:

> Do it automatically without asking.

---

## 🧠 Why It's Required in Production

Because when:

```python
DEBUG = False
```

Django stops serving static files.

If you don’t run `collectstatic`:

* Admin CSS will break
* Frontend static will 404

---

# 2️⃣ `gunicorn medibooker.wsgi:application --bind 0.0.0.0:8000`

Now this is the real production engine.

---

## 🧠 What is Gunicorn?

Gunicorn is:

> A production-grade WSGI server for Python apps.

It replaces:

```bash
python manage.py runserver
```

Because `runserver` is:

* Single-threaded
* Not optimized
* Not secure
* For development only

---

## 🔥 What This Command Means

```bash
gunicorn medibooker.wsgi:application --bind 0.0.0.0:8000
```

Let’s decode it.

### 🔹 `medibooker.wsgi:application`

Inside your project:

```
medibooker/
    wsgi.py
```

Inside `wsgi.py`:

```python
application = get_wsgi_application()
```

Gunicorn loads that `application` object.

So this part means:

> Start Django using its WSGI entry point.

---

### 🔹 `--bind 0.0.0.0:8000`

This tells Gunicorn:

* Listen on port 8000
* Accept connections from outside container

`0.0.0.0` = all network interfaces
If you used `127.0.0.1`, it would not be accessible outside.

---

# 🏗 Full Production Flow

When container starts:

```text
1. Apply DB migrations
2. Collect static files
3. Start Gunicorn
4. Gunicorn serves Django
```

---

# 🧠 What Happens When a User Makes a Request?

```
Browser
   ↓
Port 8000
   ↓
Gunicorn
   ↓
Django (WSGI)
   ↓
View
   ↓
Response
```

Later with Nginx:

```
Browser
   ↓
Nginx
   ↓
Gunicorn
   ↓
Django
```

---

# 🎯 Clean Mental Model

| Command       | Purpose                             |
| ------------- | ----------------------------------- |
| collectstatic | Prepare static files for production |
| gunicorn      | Run Django in production            |

---

# 🏆 Why These Two Together?

Because production requires:

* Static handling
* A real WSGI server

Without them:

* CSS breaks
* App unstable under load

---

You’re now understanding deployment mechanics — not just copying commands.

If you want, next I can explain:

👉 What WSGI actually is
👉 Or how Gunicorn handles workers and scaling
👉 Or why Nginx is still needed even with Gunicorn

You're progressing very well.

---

Excellent. This is the final piece of the puzzle.

You’re asking:

> If runserver already runs Django… why can’t we just use it in production?

Let’s break this cleanly and practically.

---

# 🧠 What `runserver` Actually Is

When you run:

```bash
python manage.py runserver
```

Django starts a **very basic development server**.

It is designed for:

* Local testing
* Debugging
* Fast iteration
* Auto-reload

It is NOT designed for:

* Real traffic
* Concurrency
* Performance
* Security

---

# 🔥 The Core Problem

`runserver` is:

* Single-threaded (basically one worker)
* Not optimized
* Not hardened
* Not built to handle multiple simultaneous users

---

# 🏗 Let’s Compare

## 🟢 runserver (Development)

```text
Browser → Django dev server → Django
```

* One process
* Limited concurrency
* No worker management
* Not efficient

If 50 users hit at once:

* Requests queue up
* Slow response
* Possible blocking

---

## 🔵 Gunicorn (Production)

```text
Browser → Gunicorn → Django
```

Gunicorn:

* Spawns multiple workers
* Handles concurrent requests
* Manages process crashes
* Restarts failed workers
* Production-grade

If 50 users hit at once:

* Distributed across workers
* Faster handling
* More stable

---

# 🧠 Big Technical Difference

### runserver uses:

Django’s built-in simple WSGI server (not optimized)

### Gunicorn uses:

A production-ready WSGI server built for performance.

---

# 🔥 Another Important Reason: Security

`runserver`:

* Shows debug traces
* Not hardened
* Not optimized against malicious patterns
* Not designed for internet exposure

Django literally prints a warning:

> WARNING: This is a development server. Do not use it in a production setting.

That’s not a suggestion. That’s serious.

---

# 🏎 Performance Example

Imagine:

* User uploads image
* DB query is slow
* One request blocks

With runserver:

* Entire app waits

With Gunicorn:

* Other workers continue serving

---

# 🧠 Real-World Rule

Every serious Django production setup uses:

* Gunicorn
* uWSGI
* Daphne (ASGI)
* Or similar

No serious production system uses `runserver`.

---

# 🎯 So Why Does runserver Exist?

Because:

* Developers need something simple
* Auto reload is convenient
* No config needed
* Easy debugging

It’s a convenience tool.

---

# 🏆 One-Line Answer

We don’t use `runserver` in production because:

> It is a development-only server that lacks performance, concurrency, and security features required for real traffic.

---

# 🧠 Think of It Like This

runserver = demo car
Gunicorn = production engine

Both move.
Only one is built for highways.

---
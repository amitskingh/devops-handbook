# 🐳 Docker Compose Commands (Revision)

## 1️⃣ Build + Start Containers

```bash
docker compose -f <compose-file.yml> up --build
```

### Use When

* Dockerfile changed
* `docker-compose.yml` changed
* dependencies changed
* you want to rebuild images

### Example

```bash
docker compose -f docker-compose.local.yml up --build
```

This will:

1. Build image again
2. Create containers
3. Start containers

---

# 2️⃣ Start Containers Normally

```bash
docker compose -f <compose-file.yml> up
```

### Use When

Nothing changed.

Just start containers using **existing images**.

Example:

```bash
docker compose -f docker-compose.local.yml up
```

---

# 3️⃣ Run in Background (Very Common)

```bash
docker compose -f <compose-file.yml> up -d
```

`-d` = detached mode

Containers run in background.

Example:

```bash
docker compose -f docker-compose.local.yml up -d
```

---

# 4️⃣ Stop & Remove Containers

```bash
docker compose -f <compose-file.yml> down
```

This will:

✔ Stop containers
✔ Remove containers
✔ Remove networks

BUT it **does NOT remove volumes**.

Example:

```bash
docker compose -f docker-compose.local.yml down
```

---

# 5️⃣ Remove Containers + Volumes

```bash
docker compose -f <compose-file.yml> down -v
```

This removes:

* containers
* networks
* **volumes (database data)**

Use when you want **fresh DB**.

Example:

```bash
docker compose -f docker-compose.local.yml down -v
```

---

# 6️⃣ Check Running Containers

```bash
docker compose ps
```

Shows:

```
NAME          STATUS        PORTS
django_app    running       8000
postgres_db   running       5433
```

---

# 7️⃣ View Logs

```bash
docker compose logs
```

Live logs:

```bash
docker compose logs -f
```

For specific service:

```bash
docker compose logs web
```

---

# 8️⃣ Rebuild One Service Only

```bash
docker compose build web
```

Then run:

```bash
docker compose up
```

---

# 9️⃣ Restart Containers

```bash
docker compose restart
```

---

# 🔟 Stop Containers (Without Removing)

```bash
docker compose stop
```

Containers stay but **stopped**.

Start again:

```bash
docker compose start
```

---

# ⚡ Important Dev Workflow

Typical development flow:

### First Run

```bash
docker compose -f docker-compose.local.yml up --build
```

---

### Normal Run

```bash
docker compose -f docker-compose.local.yml up
```

or

```bash
docker compose -f docker-compose.local.yml up -d
```

---

### Stop Everything

```bash
docker compose -f docker-compose.local.yml down
```

---

### Reset Database

```bash
docker compose -f docker-compose.local.yml down -v
```

---

# 🧠 One Small Correction in Your Notes

You wrote:

```
docker compose -f up
```

Correct syntax:

```bash
docker compose -f <compose-file.yml> up
```

Example:

```bash
docker compose -f docker-compose.local.yml up
```

---

# ⭐ The 5 Commands You’ll Use 90% of Time

```bash
docker compose up --build
docker compose up -d
docker compose down
docker compose down -v
docker compose logs -f
```

---
# Fixing Nginx / GitHub Actions Frontend Deployment Permissions

## Commands

```bash
sudo chown -R ubuntu:www-data /var/www/tinyurl-frontend
sudo chmod -R 755 /var/www/tinyurl-frontend
```

---

# 1. `chown` — Change Ownership

```bash
sudo chown -R ubuntu:www-data /var/www/tinyurl-frontend
```

## What it does

Changes ownership of:

* `/var/www/tinyurl-frontend`
* all files
* all subdirectories

to:

* owner → `ubuntu`
* group → `www-data`

---

## Why it is needed

GitHub Actions deploys using:

```yaml
username: ubuntu
```

So the `ubuntu` user must own the files to:

* overwrite files
* upload new builds
* delete old assets

Nginx usually runs as:

```txt
www-data
```

So using the `www-data` group allows Nginx to read the files.

---

## When to use it

Use this when you see errors like:

```txt
Permission denied
Cannot open
Cannot utime
File exists
```

especially after:

* deploying with sudo/root previously
* copying files manually
* switching deployment users
* failed CI/CD deployments

---

# 2. `chmod` — Change Permissions

```bash
sudo chmod -R 755 /var/www/tinyurl-frontend
```

## What it does

Sets permissions recursively.

`755` means:

| User Type | Permissions        |
| --------- | ------------------ |
| Owner     | read/write/execute |
| Group     | read/execute       |
| Others    | read/execute       |

---

## Why it is needed

Allows:

* `ubuntu` → deploy/update files
* `www-data` (Nginx) → read and serve files
* directories → traversable (`execute` bit)

Without proper permissions, Nginx may fail to serve static files.

---

## When to use it

Use this when:

* Nginx returns 403 Forbidden
* static files are inaccessible
* deploy succeeds but site breaks
* directory traversal permissions are missing

---

# Typical Production Setup

```txt
Owner  : ubuntu
Group  : www-data
Folders: 755
Files  : 644 or 755
```

Common for:

* Nginx static hosting
* Vite frontend deployments
* GitHub Actions CI/CD
* Ubuntu EC2 instances

---

# Verification Commands

Check ownership:

```bash
ls -la /var/www
ls -la /var/www/tinyurl-frontend
```

Check permissions:

```bash
stat /var/www/tinyurl-frontend
```

# ✅ 1️⃣ Install Ruff

If you're using a virtual environment:

```bash
pip install ruff
```

Or if you want it globally:

```bash
pip install --user ruff
```

---

# ✅ 2️⃣ Basic `pyproject.toml` Setup

Create or update `pyproject.toml` in your project root:

```toml
[tool.ruff]
line-length = 100
target-version = "py311"

exclude = [
    "migrations",
    "venv",
    "__pycache__",
]

[tool.ruff.lint]
select = ["E", "F", "I"]  # Errors, Pyflakes, Import sorting
ignore = []

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

---

## 🔎 What This Does

| Code | Meaning                                     |
| ---- | ------------------------------------------- |
| E    | Pycodestyle errors                          |
| F    | Pyflakes errors (unused imports, variables) |
| I    | Import sorting (like isort)                 |

---

# ✅ 3️⃣ Run Ruff

Check issues:

```bash
ruff check .
```

Auto-fix issues:

```bash
ruff check . --fix
```

Format code:

```bash
ruff format .
```

---

# ✅ 4️⃣ VS Code Setup (Recommended)

Install extension:

👉 **Ruff (charliermarsh.ruff)**

```

---

# ✅ 5️⃣ For Django Projects (Recommended Upgrade)


```toml
[tool.ruff.lint]
select = ["E", "F", "I", "UP", "DJ"]
```

| Code | Meaning                    |
| ---- | -------------------------- |
| UP   | Python upgrade rules       |
| DJ   | Django-specific lint rules |

This makes your backend production-level clean.

---

# 🧠 Important (From Backend Perspective)

Ruff helps you:

* Avoid unused imports (common mistake)
* Keep imports structured (important for scalability)
* Catch bugs early (undefined variables)
* Maintain clean architecture

This matters when:

* Working in large teams
* Contributing to open source
* Preparing for SDE interviews

---

# 🔥 Pro-Level Setup (Optional but Strong)

Add pre-commit hook:

```bash
pip install pre-commit
```

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff
      - id: ruff-format
```

Then:

```bash
pre-commit install
```

Now Ruff runs before every commit.

---

# 🚀 Your Next Level Move

Since you're serious about backend engineering, combine:

* Ruff
* Black (optional, but Ruff formatter replaces it)
* mypy (for strict typing)

---

## 🚀 Setting Up GitHub Workflows (CI)

GitHub Workflows (GitHub Actions) allow you to run automated checks like linting and testing whenever code is pushed or a pull request is created.

### 📁 1. Create Workflow Folder

Inside your project root:

```
.github/workflows/
```

---

### 📄 2. Create a YAML File

Example:

```
.github/workflows/lint.yml
```

---

### 🧾 3. Add Basic Lint Workflow (Ruff)

```yaml
name: lint

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"

      - name: Install Ruff
        run: pip install ruff

      - name: Run Ruff
        run: ruff check .
```

---

### ✅ What This Does

* Runs on every push to `main`
* Runs on every pull request
* Automatically checks code using Ruff
* Fails if lint errors are found

---

### 🧠 Why This Matters

* Ensures clean code before merging
* Prevents broken code in main branch
* Mimics real company CI pipelines
* Improves engineering discipline

---
# 🚀 ESLint + Prettier + Husky Setup (Fastify + TypeScript) — Quick Notes

---

## 🧠 Goal of This Setup

This setup ensures:

* ✅ No bad code enters the repo
* ✅ Consistent coding style across team
* ✅ Early detection of async bugs
* ✅ Production-level discipline

👉 Think of ESLint as your **first defense layer before production**

---

# 🧱 1. Install Dependencies

```bash
npm install -D eslint typescript-eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-prettier eslint-plugin-import prettier husky lint-staged
```

### 💡 Why these?

* `eslint` → core linting
* `typescript-eslint` → TS support + type-aware rules
* `eslint-plugin-import` → import structure discipline
* `prettier` → formatting
* `husky` → git hooks
* `lint-staged` → lint only changed files (faster + efficient)

---

# ⚡ 2. ESLint Config (Flat Config)

```js
// eslint.config.js
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import importPlugin from "eslint-plugin-import";

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,

  {
    files: ["**/*.ts"], // apply rules only to TS files

    languageOptions: {
      parser: tseslint.parser, // tells ESLint it's TypeScript
      parserOptions: {
        project: "./tsconfig.json", // 🔥 enables type-aware linting
      },
    },

    plugins: {
      import: importPlugin,
    },

    rules: {
      // 🔥 Core Discipline
      "no-console": "off", // Fastify logger is allowed
      "no-debugger": "error", // never allowed in production

      // ⚡ Async Safety (VERY IMPORTANT)
      "@typescript-eslint/no-floating-promises": "error",
      // prevents missing await → avoids silent failures

      "@typescript-eslint/no-misused-promises": "error",
      // prevents wrong async usage in callbacks

      "@typescript-eslint/await-thenable": "error",
      // prevents useless await

      // 🧠 Code Quality
      "@typescript-eslint/no-unused-vars": ["warn"],
      "@typescript-eslint/consistent-type-imports": "warn",
      // enforces: import type { X }

      "@typescript-eslint/no-explicit-any": "warn",
      // discourages unsafe typing

      // 📦 Import Discipline
      "import/order": [
        "warn",
        {
          groups: ["builtin", "external", "internal"],
          "newlines-between": "always",
          alphabetize: { order: "asc", caseInsensitive: true },
        },
      ],
      // keeps imports clean and readable

      // 🧼 Clean Code
      "prefer-const": "error",
      "no-var": "error",
    },
  },
];
```

---

## 🔥 MOST IMPORTANT LINE

```js
project: "./tsconfig.json"
```

👉 This enables **type-aware linting**

Without this:

* ❌ ESLint is just syntax checker
  With this:
* ✅ ESLint becomes **mini TypeScript compiler**

---

# 🎨 3. Prettier (Formatting)

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100
}
```

### 💡 Why Prettier?

* Removes formatting debates
* Auto-formats on save
* Keeps code consistent

👉 ESLint = logic
👉 Prettier = formatting

---

# 🧹 4. lint-staged

```json
// package.json
{
  "lint-staged": {
    "*.{ts,js}": ["eslint --fix", "prettier --write"]
  }
}
```

### 💡 Why?

* Only lints changed files
* Faster commits
* Prevents unnecessary processing

---

# 🪝 5. Husky (Git Hooks)

```bash
npx husky init
npx husky add .husky/pre-commit "npx lint-staged"
```

### 💡 What happens now?

On every commit:

1. ESLint runs
2. Fixes issues automatically
3. Blocks commit if errors exist

👉 Prevents bad code from entering repo

---

# ⚙️ 6. Scripts

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write ."
  }
}
```

---

# 📁 7. Ignore Files

### `.eslintignore`

```
node_modules
dist
.env
```

### `.prettierignore`

```
node_modules
dist
```

---

# 🧠 8. TypeScript Strict Mode

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### 💡 Why this matters?

* catches edge cases early
* prevents runtime bugs
* improves type safety

👉 This is a **big differentiator in interviews**

---

# 🔥 9. CI Integration (GitHub Actions)

```yaml
# .github/workflows/lint.yml
name: Lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npm run lint
```

### 💡 Why?

* ensures all PRs follow rules
* prevents broken code from merging

---

# ⚡ 10. Fastify-Specific Notes

* ✅ `no-console` OFF → Fastify logger is used
* ⚠️ Always handle async properly
* ⚠️ Never ignore promise errors

---

# 🚀 Final Outcome

With this setup:

* ✅ Clean and consistent code
* ✅ No async bugs slipping through
* ✅ Enforced coding discipline
* ✅ Production-ready backend setup

---

# 🧠 Key Insight

Most devs stop at:

* ❌ basic ESLint

Strong backend engineers do:

* ✅ type-aware linting
* ✅ commit hooks
* ✅ CI enforcement
* ✅ strict async rules

---

# 🧩 Mental Model

> ESLint = **automated code reviewer**
> Husky = **gatekeeper**
> Prettier = **code stylist**

---

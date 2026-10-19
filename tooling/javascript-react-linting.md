
# React Linting (JS / TS)

**ESLint →**
Code correctness
Hook rules
No unused code

**Prettier →**
Consistent formatting

**Both →**
Stable + clean codebase

**Hooks plugin →**
No stale closures

**TypeScript →**
Type safety

**GitHub Workflow →**
Prevent bad merges

**Mistake →**
Only Prettier.

---


# 🚀 React (JS / TS) – ESLint + Prettier Setup Notes

---

# ✅ 1️⃣ Install Dependencies

### 🔹 For React (JavaScript)

```bash
npm install -D eslint prettier eslint-config-prettier eslint-plugin-prettier \
eslint-plugin-react eslint-plugin-react-hooks
```

---

### 🔹 For React (TypeScript)

```bash
npm install -D eslint prettier eslint-config-prettier eslint-plugin-prettier \
@typescript-eslint/parser @typescript-eslint/eslint-plugin \
eslint-plugin-react eslint-plugin-react-hooks
```

---

# ✅ 2️⃣ Create `eslint.config.js` (Modern Flat Config)

---

## 🔹 React (JavaScript)

```js
import js from "@eslint/js";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import prettier from "eslint-plugin-prettier";

export default [
  js.configs.recommended,
  {
    files: ["**/*.js", "**/*.jsx"],
    plugins: {
      react,
      "react-hooks": reactHooks,
      prettier,
    },
    rules: {
      "react/react-in-jsx-scope": "off",
      "react-hooks/rules-of-hooks": "error",
      "react-hooks/exhaustive-deps": "warn",
      "prettier/prettier": "error",
    },
    settings: {
      react: { version: "detect" },
    },
  },
];
```

---

## 🔹 React (TypeScript)

```js
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import prettier from "eslint-plugin-prettier";

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    files: ["**/*.ts", "**/*.tsx"],
    plugins: {
      react,
      "react-hooks": reactHooks,
      prettier,
    },
    languageOptions: {
      parser: tseslint.parser,
      parserOptions: {
        ecmaFeatures: { jsx: true },
      },
    },
    rules: {
      "react/react-in-jsx-scope": "off",
      "react-hooks/rules-of-hooks": "error",
      "react-hooks/exhaustive-deps": "warn",
      "prettier/prettier": "error",
    },
    settings: {
      react: { version: "detect" },
    },
  },
];
```

---

# ✅ 3️⃣ Create `.prettierrc`

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

---

# ✅ 4️⃣ Add Scripts

```json
"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix",
  "format": "prettier --write ."
}
```

---

# ✅ 5️⃣ Run

```bash
npm run lint
npm run lint:fix
npm run format
```

---

# ✅ 6️⃣ VS Code Setup

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "always"
  },
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

Install:

* ESLint extension
* Prettier extension

---

# ✅ 7️⃣ GitHub Actions (CI)

`.github/workflows/lint.yml`

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
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run lint
```

---

# 🔎 What Each Part Ensures

| Tool              | Purpose                     |
| ----------------- | --------------------------- |
| ESLint            | Prevents logical mistakes   |
| React plugin      | React best practices        |
| Hooks plugin      | Prevents stale closure bugs |
| TypeScript plugin | Type-aware linting          |
| Prettier          | Formatting consistency      |
| CI                | Blocks bad merges           |

---

# 🧠 Why This Matters

Without this:

* Hook dependency bugs reach production
* Inconsistent formatting across team
* Unused variables remain
* PR reviews waste time on style

With this:

✔ Cleaner code
✔ Safer hooks
✔ Team consistency
✔ Production-level discipline

---

# 🎯 Interview Summary (Memorize This)

> I use ESLint for code quality and React Hooks correctness, and Prettier for consistent formatting. In TypeScript projects, I enable type-aware linting. I also integrate linting in CI to prevent bad merges and enforce team discipline.

---

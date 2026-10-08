# Portfolio Pre-development Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Tailwind CSS v4 support and local Git version control to the existing Vite/Vue portfolio starter without changing the starter UI.

**Architecture:** Vite processes Tailwind through Tailwind’s official Vite plugin. The existing `src/style.css` globally imports Tailwind before its existing CSS. Git tracks the project locally; GitHub setup and deployment remain intentionally out of scope.

**Tech Stack:** Vue 3, TypeScript, Vite 8, Tailwind CSS v4, npm, Git.

## Global Constraints

- Use `tailwindcss` and `@tailwindcss/vite`; do not add a UI-component package.
- Preserve `src/App.vue`, `src/components/HelloWorld.vue`, and existing CSS rules.
- Do not configure a GitHub remote or publish the project.
- Validation must pass with `npm run build`.

---

### Task 1: Add Tailwind CSS build integration

**Files:**
- Modify: `package.json`
- Modify: `package-lock.json`
- Modify: `vite.config.ts:1-6`
- Modify: `src/style.css:1`

**Interfaces:**
- Consumes: Existing Vite configuration exporting `defineConfig({ plugins: [vue()] })`.
- Produces: Tailwind utility classes can be compiled from Vue component templates by Vite.

- [ ] **Step 1: Install the official Tailwind packages**

Run:
```bash
npm install -D tailwindcss @tailwindcss/vite
```

Expected: `package.json` lists both packages in `devDependencies` and the lockfile updates.

- [ ] **Step 2: Register the Tailwind Vite plugin**

Replace `vite.config.ts` with:
```ts
import tailwindcss from '@tailwindcss/vite'
import vue from '@vitejs/plugin-vue'
import { defineConfig } from 'vite'

// https://vite.dev/config/
export default defineConfig({
  plugins: [vue(), tailwindcss()],
})
```

- [ ] **Step 3: Import Tailwind before the existing global rules**

Insert the following as line 1 of `src/style.css`:
```css
@import 'tailwindcss';
```

- [ ] **Step 4: Verify the production build**

Run:
```bash
npm run build
```

Expected: command exits with status 0 and emits a production bundle in `dist/`.

### Task 2: Initialize local Git version control

**Files:**
- Create: `.git/` (Git metadata directory)

**Interfaces:**
- Consumes: Existing `.gitignore` file.
- Produces: A local Git repository ready for a later GitHub remote.

- [ ] **Step 1: Initialize the repository**

Run:
```bash
git init
```

Expected: Git reports it initialized an empty repository in this project.

- [ ] **Step 2: Confirm Git sees the project files**

Run:
```bash
git status --short
```

Expected: project files appear as untracked and ignored dependencies do not appear.

- [ ] **Step 3: Do not create a commit**

Leave files uncommitted so the user can review the baseline before the first commit.

## Self-review

- Scope coverage: Task 1 configures the only approved styling dependency without modifying the starter UI. Task 2 provides local version control without creating a remote or publishing.
- Placeholder scan: no placeholders or deferred implementation steps are present; deferred dependencies are explicitly out of scope.
- Consistency: the package names, Vite plugin import, and stylesheet import use Tailwind v4’s official Vite integration.

# Codebase Orientation

Use this when the PM is looking at a repo for the first time, or asks something like "give me a tour", "what is this codebase", "what's in this repo", "help me get oriented".

## Goal

In ~5 minutes of exploration, give the PM a one-page mental model of: **what kind of thing this is**, **what the major pieces are**, **how active it is**, and **where to look next** for whatever they care about.

## Step 1 — Identify the stack and shape

Run these in order. Don't show raw output to the PM; synthesize.

```bash
# What's at the top level?
ls -la
# What's it written in? Package files reveal a lot.
ls package.json pyproject.toml requirements.txt go.mod Cargo.toml pom.xml build.gradle composer.json Gemfile 2>/dev/null
# README — the team's own intro to the codebase
cat README.md 2>/dev/null | head -100
```

The package file tells you the language and major frameworks. The README tells you what the team says it is (which sometimes differs from what it actually is — keep that in mind).

**Classify the repo shape:**
- **Single-app frontend** (React/Vue/Next/Svelte) — has `src/`, `pages/` or `app/`, components, routes
- **Single-app backend** — has controllers/handlers/routes + models + a server entry point
- **Full-stack monolith** — has both frontend and backend in one tree, often `client/` + `server/` or `frontend/` + `backend/`
- **Monorepo** — has `packages/`, `apps/`, or `services/` with multiple sub-projects, often with a workspace config (`pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`)
- **Library/SDK** — exports a public API, has `lib/` or `src/` with index file, often published to a package registry
- **Mobile** — has `ios/`, `android/`, or React Native / Flutter / native project files
- **Infrastructure / config repo** — mostly YAML, Terraform, Helm charts, GitHub Actions

The shape determines where features live, and you'll use this to set expectations with the PM.

## Step 2 — Find the entry points

Entry points are where execution starts. They're the doorways into the product. Look for:

- Frontend: `src/index.*`, `src/main.*`, `src/App.*`, `pages/_app.*`, `app/layout.*`, `app/page.*`
- Backend: `server.*`, `main.*`, `index.*`, `app.*`, anything with `if __name__ == "__main__"`, or a `cmd/` directory in Go
- Mobile: `App.tsx`, `MainActivity.kt`, `AppDelegate.swift`

```bash
# Find common entry points
find . -maxdepth 4 \( -name "main.*" -o -name "index.*" -o -name "App.*" -o -name "server.*" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | head -20
```

`view` one or two of these. Note what they import and wire together — those imports are usually the major subsystems.

## Step 3 — Find the routes / surface area

Routes (URLs, screens, API endpoints) are the **product surface** as far as the PM is concerned. Each one corresponds to something a user can do or see.

**Web frontend (Next.js, React Router, etc.):**
```bash
# Next.js app router
find . -path "*/app/*/page.*" -not -path "*/node_modules/*" | head -50
# Next.js pages router
find . -path "*/pages/*" -name "*.tsx" -o -name "*.jsx" -not -path "*/node_modules/*" | head -50
# React Router — search for route definitions
grep -rn "<Route\|createBrowserRouter\|useRoutes" --include="*.tsx" --include="*.jsx" --include="*.ts" --include="*.js" . 2>/dev/null | head -30
```

**Backend APIs:**
```bash
# Express, Fastify, etc.
grep -rn "app\.\(get\|post\|put\|delete\|patch\)\|router\.\(get\|post\|put\|delete\|patch\)" --include="*.ts" --include="*.js" . 2>/dev/null | head -50
# FastAPI / Flask
grep -rn "@app\.\(get\|post\|put\|delete\|router\)\|@router\.\(get\|post\)" --include="*.py" . 2>/dev/null | head -50
# Rails
find . -name "routes.rb" 2>/dev/null
# Spring
grep -rn "@\(Get\|Post\|Put\|Delete\|Request\)Mapping" --include="*.java" . 2>/dev/null | head -50
```

**Mobile screens:** usually a navigation config file (`navigation.ts`, `Routes.kt`, `Router.swift`) or screen components grouped under `screens/` or `views/`.

Count the routes. Group them by area (auth, billing, settings, etc.). This is the *closest thing to a feature list* the PM will get from the code alone.

## Step 4 — Skim the major directories

Look at the top-level directories that aren't config/build/dependencies. For each one, the name plus a glance at one or two files inside usually tells you the purpose.

```bash
# Top-level directories, ignoring noise
ls -d */ 2>/dev/null | grep -vE "^(node_modules|\.git|dist|build|coverage|\.next|target|vendor)/" | head -20
```

For directories that aren't self-explanatory, peek inside:
```bash
ls <dir>/ | head -10
```

Common patterns to recognize:
- `components/` — UI building blocks
- `pages/` or `app/` — screens/routes
- `lib/` or `utils/` — shared helpers
- `api/` or `services/` — calls to backend or external services
- `models/` or `db/` or `prisma/` — data layer; the schema files here are gold for understanding the domain
- `hooks/` (React) — reusable logic
- `controllers/` + `routes/` + `models/` — classic MVC backend
- `features/` or `modules/` — feature-organized code, each subdirectory is roughly one product area

## Step 5 — Check activity and team size signals

How healthy and active is this codebase?

```bash
# Commits in the last 90 days
git log --since="90 days ago" --oneline 2>/dev/null | wc -l
# Recent contributors
git log --since="90 days ago" --format="%an" 2>/dev/null | sort -u | head -20
# Most-recently-touched files (often where active work is)
git log --since="30 days ago" --name-only --format="" 2>/dev/null | grep -v "^$" | sort | uniq -c | sort -rn | head -20
```

Activity signals matter for PM context: a sleepy codebase with 2 commits last quarter and a 50-commits-per-week codebase need very different product-strategy framings.

## Step 6 — Synthesize the tour

Give the PM a short writeup, roughly this shape (prose, not a long bulleted list — but use a couple of headers if it helps):

> **What this is.** A [Next.js web app | FastAPI backend | React Native mobile app | …] for [best guess at purpose, from README and routes].
>
> **Major pieces.** [2–4 sentences walking through the main directories and what each is for.]
>
> **Product surface.** Roughly [N] user-facing routes/screens, grouped into [auth, dashboard, settings, billing, …].
>
> **Activity.** [N] commits in the last 90 days from [N] contributors. Most recent work is concentrated in [areas].
>
> **Where would you like to go first?** [Offer 2–3 specific next directions based on what stood out — a feature area, an unusually large module, a recent refactor in the commit log, etc.]

The last line matters. Orientation is rarely the end goal — it's the setup for a real question. Tee that up.

## Gotchas

- **Generated code lies.** `dist/`, `build/`, `.next/`, compiled output — ignore. They inflate file counts and obscure the real source.
- **READMEs lie too, sometimes.** Compare what the README claims to what you see in the routes. If the README mentions a feature but you can't find code for it, flag that to the PM.
- **Vendored dependencies look like the codebase but aren't.** `vendor/`, `third_party/`, anything under `node_modules/` is not the team's code.
- **Monorepos need a second pass.** If you detect a monorepo, do a top-level pass first, then ask the PM which sub-project they want to dig into. Don't try to map all of them.

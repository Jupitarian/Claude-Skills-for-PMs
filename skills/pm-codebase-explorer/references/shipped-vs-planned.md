# Shipped vs Planned

Use this when the PM asks "is feature X actually live?", "what feature flags are running?", "is this code actually being used?", "what's deprecated?", or any question about the gap between what's in the repo and what's actually reaching users.

## Why this matters

A codebase is the union of:
- Code that runs in production for everyone
- Code that runs only when a flag is on (which might be on for nobody, some employees, some customers, or rolling out gradually)
- Code that runs only in non-production environments
- Code that's wired up but unreachable (dead code)
- Code that's reachable but for a feature that's been deprecated and is on its way out
- Code in active development that hasn't shipped

Looking at the repo and saying "we have feature X" without checking which of these applies is a classic mistake that leads to PMs talking past engineers.

## Step 1 — Find feature flags

Feature flag systems are often centralized. Identify the system first.

```bash
# Common feature-flag library imports / SDKs
grep -rni "launchdarkly\|statsig\|optimizely\|unleash\|flagsmith\|posthog.feature\|growthbook\|@vercel/flags" \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" \
  --exclude-dir=node_modules . 2>/dev/null | head -10

# Generic feature flag patterns
grep -rni "featureFlag\|isFeatureEnabled\|isEnabled\|useFlag\|getFlag\|hasFeature" \
  --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null | head -20

# Environment-variable-based flags (poor-man's feature flags)
grep -rn "process\.env\.[A-Z_]*\(ENABLE\|FEATURE\|FLAG\)" \
  --include="*.ts" --include="*.tsx" --include="*.js" \
  --exclude-dir=node_modules . 2>/dev/null | head -20

# Config files with flags
find . \( -name "flags.*" -o -name "features.*" -o -name "config.*" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | head -10
```

Once you know which system is in use, search for every flag-check call. Each one is a gate — extract the flag names.

**Important caveat to share with the PM:** the code only tells you flags *exist*. It does NOT tell you the rollout state — whether the flag is on for everyone, on for 5% of users, on for employees only, etc. That state lives in the flag system's dashboard (LaunchDarkly UI, Statsig console, etc.), not in the code. Be explicit about this:

> "I found 17 feature flag checks in the code, gating features like A, B, and C. I can tell you these gates exist, but I can't tell you from the code whether each flag is currently on or off in production — that's in your flag dashboard."

## Step 2 — Find dead code

Dead code is code that exists but isn't reachable. Common signs:

**Files no one imports:**
```bash
# For each file in a feature area, check if anything imports it
for f in $(find src/features/checkout -name "*.tsx" -o -name "*.ts" 2>/dev/null); do
  basename=$(basename "$f" | sed 's/\.[^.]*$//')
  count=$(grep -rln "from.*${basename}" --include="*.ts" --include="*.tsx" \
    --exclude-dir=node_modules . 2>/dev/null | grep -v "$f" | wc -l)
  if [ "$count" -eq 0 ]; then echo "Unimported: $f"; fi
done
```

**Routes/endpoints with no handler reference:** if a URL pattern exists in routing config but no client calls it, that's a likely-dead endpoint.

**Old component versions:** files named `LegacyX`, `OldX`, `XV1`, `XDeprecated` are usually candidates:
```bash
find . -type f \( -iname "*legacy*" -o -iname "*old*" -o -iname "*deprecated*" \
  -o -iname "*v1*" -o -iname "*backup*" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | head -20
```

**Flagged-off code that's never enabled.** This is the trickiest — code wrapped in a flag check that's been turned off for so long no one remembers. Look at flag names that suggest features that have been "in development" for over a year:
```bash
# Git log on the flag definition file to see how long this flag has existed
git log --follow --oneline <flag-config-file> 2>/dev/null | tail -20
```

## Step 3 — Find deprecations

Code can be reachable but on its way out. Signals:

```bash
# Explicit deprecation markers
grep -rn "@deprecated\|@Deprecated\|DEPRECATED:\|# deprecated" \
  --include="*.ts" --include="*.tsx" --include="*.py" --include="*.java" --include="*.kt" \
  --exclude-dir=node_modules . 2>/dev/null | head -30

# Sunset / removal warnings
grep -rni "will be removed\|sunset\|end of life\|EOL\|migrate to" \
  --include="*.ts" --include="*.py" --include="*.md" \
  --exclude-dir=node_modules . 2>/dev/null | head -20

# Old API versions still around
find . -type d -name "v1" -o -name "v2" 2>/dev/null | grep -v node_modules
```

## Step 4 — Find work-in-progress

Code that's checked in but not yet ready for users.

```bash
# WIP markers
grep -rn "TODO\|FIXME\|XXX\|WIP\|HACK\|@todo" \
  --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null | head -30

# Commented-out routes / disabled tests
grep -rn "^[[:space:]]*//.*route\|^[[:space:]]*#.*route\|\.skip\|\.todo\|xdescribe\|xit\(" \
  --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null | head -20

# Recently-created files that aren't yet referenced
git log --since="30 days ago" --diff-filter=A --name-only --format="" 2>/dev/null \
  | grep -v "^$" | sort -u | head -20
```

Cross-check: a new file recently added + flag-gated + few imports = a feature that's been built but not turned on yet.

## Step 5 — Find environment-specific code

```bash
# Dev/staging-only behavior
grep -rn "NODE_ENV.*development\|ENV.*staging\|isDev\|__DEV__\|debug.*true" \
  --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null | head -20

# Admin / internal flags
grep -rni "isAdmin\|isInternal\|isEmployee\|@company\.com" \
  --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null | head -20
```

## Step 6 — Synthesize the audit

Give the PM a clear separation. Use headers here because the categories really matter:

> ## What's actually shipped to all users
> [List of features confirmed by: route exists, no flag gating, no env restriction.]
>
> ## What's gated by feature flags
> [Feature flag names + the rough product area each guards. Remind the PM you can't see rollout state from code.]
>
> ## What's deprecated or being phased out
> [Things marked @deprecated, old API versions, sunset warnings in comments.]
>
> ## What looks like work in progress
> [Recently-added flag-gated code, files with WIP markers, commented-out routes.]
>
> ## What looks like dead code
> [Files no one imports, old `Legacy*` versions, flags that have been off for years.]
>
> ## What I can't tell you from the code alone
> - Current flag rollout percentages (need the flag system's dashboard)
> - Whether a feature is actually being used by customers (need analytics)
> - Whether something marked @deprecated has a removal date set

## Common PM questions and how to answer them

**"Is feature X live?"** → Check (1) does the code exist, (2) is it behind a flag, (3) is it behind an env check, (4) is the route actually registered. Report each.

**"Why does our app still mention [old thing]?"** → Search the codebase for the old thing's name. It's usually one of: still actively used (PM forgot), partially migrated (some places use new, some old), in a vendored dependency, or in a hardcoded string in a translation file no one updated.

**"What was added in the last quarter?"** →
```bash
git log --since="90 days ago" --pretty=format:"%h %s" --no-merges 2>/dev/null | head -50
git log --since="90 days ago" --diff-filter=A --name-only --format="" 2>/dev/null \
  | grep -v "^$" | sort -u
```
Group the commits by area and translate the commit messages into product language.

**"Are there features in the code that we never launched?"** → Look for flag-gated code where the flag has been off-by-default since creation, plus files that exist but aren't reachable from any route. These are "ghosts" — code that almost shipped.

## Gotchas

- **Flags can be checked from multiple places.** Make sure you've searched all common flag-checking patterns in the codebase, not just one.
- **Configuration overrides code.** A feature can look "off" in code defaults but be turned on by environment config at deploy time. If the PM cares about production state, the answer ultimately requires checking the deployed config, not just the repo.
- **"Dead" code sometimes isn't.** Code that's imported only via dynamic strings, reflection, or runtime registration can look unreferenced but actually run. Be careful with confident "this is dead" claims — phrase as "this *appears* unused; worth confirming with engineering before deleting."
- **A/B test code looks like dead code.** A variant arm that's currently allocated 0% traffic looks unused, but isn't dead — it might get traffic again. Don't flag A/B variants as dead.

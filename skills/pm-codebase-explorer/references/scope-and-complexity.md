# Scope and Complexity

Use this when the PM asks "how big is this?", "how complex is X?", "how hard would it be to add Y?", "what's the surface area of feature Z?" — anything where they're trying to size something.

## A crucial caveat to share with the PM up front

You cannot estimate engineering effort. Anyone who tells you they can from looking at code is wrong. What you *can* do is describe the shape of the work — files affected, abstractions involved, blast radius — and let the PM's engineers translate that into time.

When the PM asks "how long will this take?", reframe: "I can't give you a time estimate, but I can tell you what's involved, and that should help you and the team scope it."

## Step 1 — Get a size baseline for the codebase as a whole

Useful context before talking about any specific change.

```bash
# Total lines of code, ignoring noise — use cloc if available
which cloc >/dev/null 2>&1 && cloc . --exclude-dir=node_modules,dist,build,.git,vendor,target,.next 2>/dev/null | tail -20

# Fallback: rough line count by file extension
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \
  -o -name "*.py" -o -name "*.rb" -o -name "*.go" -o -name "*.java" \
  -o -name "*.kt" -o -name "*.swift" -o -name "*.rs" -o -name "*.vue" -o -name "*.svelte" \) \
  -not -path "*/node_modules/*" -not -path "*/dist/*" -not -path "*/build/*" \
  -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/.next/*" \
  -exec wc -l {} + 2>/dev/null | tail -1

# File count by area
find . -type d -maxdepth 3 -not -path "*/node_modules/*" -not -path "*/.git/*" \
  -not -path "*/dist/*" -not -path "*/build/*" 2>/dev/null | head -30
```

Rough mental scale:
- **Small**: < 10k lines, < 200 files. One person can hold it in their head.
- **Medium**: 10k–100k lines. Familiar engineers can navigate confidently; newcomers need a few weeks.
- **Large**: 100k–1M lines. Specialization is necessary; no one knows the whole thing.
- **Very large**: > 1M lines. Each team owns a slice; cross-cutting changes are projects in themselves.

Share this baseline with the PM as context, not as the answer.

## Step 2 — Size a specific feature area

If the PM asks "how big is the checkout feature?", find the relevant directory (use the `feature-to-code.md` techniques) and measure it:

```bash
# Size of a specific directory
find src/features/checkout -type f \( -name "*.ts" -o -name "*.tsx" \) \
  -exec wc -l {} + 2>/dev/null | tail -1

# File count
find src/features/checkout -type f \( -name "*.ts" -o -name "*.tsx" \) 2>/dev/null | wc -l

# How many other files depend on this area?
grep -rln "from ['\"].*features/checkout" --include="*.ts" --include="*.tsx" \
  --exclude-dir=node_modules . 2>/dev/null | wc -l
```

The last one — incoming dependencies — is the most useful number for the PM. A feature with 3 incoming imports is mostly self-contained. A feature with 200 incoming imports is woven into everything; changes there ripple.

## Step 3 — Size a hypothetical change

When the PM asks "how complex would it be to add X?", you're trying to identify:

1. **What files would change?** Find the analogous existing feature. If they want to add "Apple Pay" and "Stripe" is already in the codebase, list every file that mentions Stripe — that's roughly the set of files Apple Pay would also touch.
2. **What abstractions are in the way?** Is there a `PaymentProvider` interface that makes adding a new one a clean extension, or is Stripe hardcoded throughout? The former is a few-day job; the latter is weeks.
3. **What's the blast radius?** Will the change touch the database (migrations needed), the public API (versioning concerns), shared components (regression risk), or just one feature area (lower risk)?

```bash
# Find every file mentioning the analogous existing thing
grep -rln "stripe\|Stripe" --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null

# Look for the abstraction it sits behind
grep -rn "interface.*Payment\|class.*Payment\|PaymentProvider\|PaymentMethod" \
  --include="*.ts" --include="*.py" --exclude-dir=node_modules . 2>/dev/null | head -20
```

## Step 4 — Read complexity signals in code

Beyond raw size, certain signals predict that something will be harder than its line count suggests. Note these for the PM:

**Signals of higher-than-expected complexity:**
- **Deep conditional logic.** Lots of `if/else` branches, especially nested. Each branch is a separate code path that needs testing.
- **Custom state machines** (look for `switch (state)`, `case "PENDING"`, `transition()`). These are reasoning-heavy.
- **Concurrency primitives** — `async`, locks, queues, transactions. Easy to break in subtle ways.
- **External integrations.** Each third-party API, webhook, or SDK is a dependency that can fail in its own special ways.
- **Hand-rolled implementations of things that have libraries.** Custom date logic, custom auth, custom routing — usually means there are weird requirements baked in.
- **Comments that say "// HACK", "// TODO", "// FIXME", "// XXX".** The developers themselves are flagging fragility:
  ```bash
  grep -rn "HACK\|FIXME\|XXX\|@deprecated" --include="*.ts" --include="*.py" \
    --exclude-dir=node_modules <feature-dir> 2>/dev/null | head -20
  ```
- **A lot of files for the apparent feature size.** If "send email notification" spans 30 files, there's complexity hiding somewhere.

**Signals of lower-than-expected complexity:**
- **Clean abstractions** — interfaces, dependency injection, plugin patterns. Adding a new variant is mechanical.
- **Strong test coverage of the feature area**:
  ```bash
  find <feature-dir> -name "*.test.*" -o -name "*.spec.*" 2>/dev/null | wc -l
  ```
- **Recently refactored code** — recent commits often mean the area is in good shape and someone understands it.

## Step 5 — Translate to PM-readable scope

Synthesize into something the PM can take to a planning conversation. Use prose, not a giant table:

> **Adding Apple Pay would touch roughly the same surface area as Stripe currently does — about 14 files across the frontend (payment form, checkout flow, settings) and 8 files in the backend (payment service, webhook handlers, order processing). The good news: there's a `PaymentProvider` interface, so most of the work is implementing a new provider class. The harder part is the webhook handling — Stripe's webhook code has several "FIXME" comments suggesting it's fragile, and a new provider means a second webhook integration with its own quirks.**
>
> **Rough scope shape:** new provider class + frontend payment option + webhook handler + tests + a database column for payment method preference.
>
> **Risks to flag to your engineers:** (1) the webhook handler is marked fragile in comments; adding to it without refactoring could compound that. (2) `Stripe` is referenced directly in 4 places outside the provider abstraction — those would need to be cleaned up first or the abstraction won't actually help.
>
> **What I can't tell you:** how long this takes — that depends on your team's familiarity with payment integrations and how thorough you want the testing to be.

## Gotchas

- **Tests count.** When sizing a change, doubling the implementation line count is a decent rough guess for the testing work. Mention this.
- **Migrations are slower than they look.** A 5-line database migration can be a multi-day project if the table has a billion rows.
- **Customer impact ≠ code size.** A 3-line change in a pricing function can be the most consequential change of the quarter. Don't conflate complexity with importance.
- **Past commits are not a great proxy for future work.** Just because the original Stripe integration took a week doesn't mean a new one will — the codebase is different now, the team is different, the requirements may be different.

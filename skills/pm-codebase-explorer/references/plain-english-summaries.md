# Plain-English Summaries

Use this when the PM asks you to read a file, a PR, a function, a commit, or a directory and explain what it does — without engineering jargon.

## Goal

Turn code into a description a product person can act on. They should leave knowing **what this thing does for the user**, **why it exists**, and **what's notable about how it does it** — without needing to learn the language it's written in.

## Step 1 — Read carefully and identify the audience-relevant parts

Read the whole file/PR first. Don't paraphrase line by line — that produces bad summaries. Instead, ask yourself:

1. **What does this do from the outside?** If a user or another part of the system uses this code, what happens?
2. **What product behavior changes if this code changes?** That's what the PM actually cares about.
3. **What's deliberately interesting?** Special cases, business logic, validation rules, magic numbers, gates, defaults.
4. **What's noise the PM doesn't need?** Imports, type definitions, helper plumbing, framework boilerplate.

Strip away the noise and lift the signal.

## Step 2 — Identify the type of code

The shape of the summary depends on what you're summarizing:

| Type | What to emphasize |
|---|---|
| UI component / screen | What the user sees, what they can do, what state changes |
| API endpoint / handler | Inputs accepted, what's done with them, what's returned, side effects |
| Background job | When it runs, what it processes, what changes as a result |
| Business logic function | The rule it enforces, the inputs, the outputs, edge cases |
| Database model / schema | What entity this represents, key fields, relationships to other entities |
| Configuration | What this configures, what changes if it changes |
| Test file | What behavior is being verified (good for understanding intended behavior!) |

## Step 3 — Write the summary

Lead with one sentence that captures the purpose. Then expand. Keep total length proportional to the code's complexity — most files can be summarized in 4–6 sentences.

### Template for a file/component

> **`<filename>` — <one-sentence purpose>.**
>
> <2–4 sentences describing what it does, in order of what the user would experience or what happens at runtime.>
>
> <1–2 sentences on anything notable: special cases, dependencies, oddities, things that seem important to flag.>
>
> <Optional: "Want me to also look at X?" if there's an obvious next file to read.>

### Template for a PR / commit

> **What this PR does:** <one sentence.>
>
> **Why it exists:** <best guess based on commit message, description, and code — flag if you're inferring.>
>
> **User-facing impact:** <what changes for the user. "None" is a valid answer for refactors.>
>
> **Files changed:** <a count + the categories: e.g., "8 files — 3 frontend components, 2 API handlers, 3 tests.">
>
> **Worth flagging:** <anything that should raise an eyebrow — DB migration, public API change, removed feature, suspicious lack of tests, etc.>

### Template for a function

> **`<functionName>` — <one sentence on its purpose>.**
>
> Takes <plain-English description of inputs>. <What it does.> Returns <plain-English description of output>.
>
> <Edge cases or notable behavior.>

## Examples of the translation

These illustrate the "engineer language → PM language" move you're making.

**Engineer language:** "This function memoizes a debounced API call to the search endpoint, returning a cleanup function for the effect."
**PM language:** "When the user types in the search box, this waits a moment for them to stop typing, then asks the server for results. The pause prevents the app from spamming the server on every keystroke."

**Engineer language:** "Validates the request body against a Zod schema, throws a 400 if invalid, then persists the order with an idempotency key."
**PM language:** "Checks that the order details are valid (and rejects them with an error message if not), then saves the order — with a safeguard so that if the same order gets submitted twice by accident, we only create it once."

**Engineer language:** "Migrates the `users` table to add a `tenant_id` column with a foreign key constraint, backfills existing rows to the default tenant, and adds a non-null constraint."
**PM language:** "This is a database change to associate every user with a tenant (organization). Existing users are assigned to a default tenant, and going forward every new user must have one. This is the kind of change that requires care to deploy because it modifies a major table."

## When the code is itself unclear

Sometimes code is genuinely hard to follow — clever, poorly named, undocumented, or all three. Don't fake confidence. Say what you can tell, flag what you can't:

> "This function appears to be calculating some kind of pricing adjustment based on user tier and order history, but the logic is dense and uses variables named `x`, `y`, `tmp1`. I can tell you the inputs and outputs with confidence, but I wouldn't trust myself to fully explain the rule without checking with whoever wrote it. The git blame might help — last touched by [name] in [date]."

The PM benefits more from honesty here than from a confident-sounding guess.

## Special case: explaining a PR for review purposes

If the PM is reviewing a PR (or trying to decide if they care about one):

1. Read the PR description / commit message first — that's the author's own framing.
2. Compare what the description says to what the code actually changes. Flag any mismatch.
3. Look for the "blast radius": does this touch shared code, public APIs, or things that affect users beyond the stated feature?
4. Note what's *not* in the PR but maybe should be — e.g., implementation but no tests, or a flag-gating that's missing.

## Gotchas

- **Don't reproduce large chunks of code in your summary.** A line or two as illustration is fine; a full function dumped into the response is not what the PM asked for. Paraphrase.
- **Don't translate variable names into product terms incorrectly.** If a variable is called `customerLifetimeValue`, it might or might not actually be CLV — sometimes engineers reuse names sloppily. Read what's done with it before describing it.
- **Don't say "this is well-written" or "this is bad code."** Code quality judgments aren't useful to the PM and often aren't fair without more context. Stick to what it does.
- **Tests are documentation.** If you're confused about what a function is supposed to do, the test file often tells you in product terms ("it returns an error when the user has insufficient balance"). Read the tests.

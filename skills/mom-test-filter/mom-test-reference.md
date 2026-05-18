# The Mom Test — Practitioner's Reference

> **Source:** *The Mom Test* by Rob Fitzpatrick
> **Purpose:** Quick-reference framework for designing customer research questions that surface truth instead of validation.

The Mom Test is a set of rules for asking questions that even your mom can't lie to you about. The name comes from a simple observation: if you pitch your idea to your mom, she'll tell you it's great — not because it is, but because she loves you. Enterprise stakeholders have their own reasons to be polite. The framework exists to defeat that politeness.

---

## The Three Principles

### 1. Past behavior > Future hypotheticals

People are terrible predictors of their own future behavior. They'll say "yes, I'd totally use that" because it costs them nothing to say. What they've *actually done* tells you what they'll actually do.

| | Example |
|---|---------|
| **Bad** | "Would you use X?" |
| **Good** | "How do you handle X now?" |

Ask about what they already do, not what they might do. If they're not already solving the problem somehow — hacking together spreadsheets, using a competitor, or just suffering through it — they probably won't adopt your solution either.

### 2. Their life > Your idea

The moment you describe your idea, you've contaminated the conversation. They stop telling you about their reality and start reacting to your pitch. Keep the focus on their world — their workflows, their frustrations, their workarounds.

| | Example |
|---|---------|
| **Bad** | "What if we built Y?" |
| **Good** | "Walk me through your last procurement cycle" |

You learn more from understanding how they buy software today than from asking whether they'd buy yours. Their procurement story reveals budget authority, timeline expectations, blockers, and decision-makers — without you having to ask about any of those directly.

### 3. Specifics > Generics

Generic questions get generic answers. "Is security important?" will always get a yes. But when you ask about a specific incident, you get the real story — what actually happened, what they actually did about it, and how much it actually mattered.

| | Example |
|---|---------|
| **Bad** | "Is security important?" |
| **Good** | "Tell me about the last time security blocked a purchase" |

Specifics also expose the gap between stated priorities and revealed behavior. Someone might rank security as their #1 concern on a survey, but when you ask for the last time it actually blocked something, they can't name one.

---

## Common Violations

Watch for these patterns in your own questions and in interview guides:

**Hypothetical framing** — Any question starting with "Would you..." or "Could you see yourself..." is asking for a prediction, not a fact. Replace with "Have you ever..." or "When was the last time..."

**Leading questions** — "Don't you think X is a problem?" tells the interviewee what answer you want. Ask "How do you handle X?" and let them tell you whether it's a problem.

**Fishing for compliments** — "What do you think of our approach to Y?" invites polite validation. Instead: "What are you using for Y right now? What's working? What's not?"

**The premature solution** — Describing your feature before understanding their problem. Once you say "we're building SSO," they'll talk about SSO. If you ask "walk me through your last security review," you might learn SSO was never the real issue.

**Future commitment traps** — "Would you pay for this?" is meaningless. "What's your budget for tools like this?" is better. "What did you pay for the last tool you adopted?" is best. Past spending predicts future spending. Hypothetical willingness predicts nothing.

---

## Enterprise Application

In B2B and enterprise research, the Mom Test violations are especially dangerous because stakeholders have sophisticated reasons to mislead you — not out of malice, but out of habit. Procurement people recite requirements lists. Executives give strategic non-answers. Champions oversell internal buy-in.

**Instead of asking enterprise prospects about requirements, ask about behavior:**

| Don't ask | Ask instead | Why it's better |
|-----------|-------------|-----------------|
| "Would you pay for SSO?" | "Tell me about the last time security blocked a tool purchase" | Reveals whether SSO is a real blocker or a checkbox on a form nobody reads |
| "Do you need role-based permissions?" | "Who touches your current PM tool and what do they do in it?" | Surfaces actual permission patterns vs. imagined ones |
| "Is compliance important to your org?" | "Walk me through your last vendor security review" | Exposes what compliance actually looks like in practice — the real process, not the stated policy |
| "Would your team adopt this?" | "How did your team adopt the last tool you rolled out?" | Reveals adoption patterns, resistance points, and what "adoption" actually means to them |
| "What features do you need for enterprise?" | "What made you outgrow your last tool?" | Gets to root causes instead of wishlists |

**The pattern:** Enterprise buyers will give you a requirements list if you ask for one. They'll hand you an RFP template. That list tells you what they think they need — which is often inherited from a previous evaluation and disconnected from actual usage. Past behavior cuts through the template.

---

## Quick Self-Check

Before you send an interview guide, scan every question against this list:

- [ ] Does this question ask about the past or the future? (Past = good)
- [ ] Am I learning about their world or pitching mine? (Their world = good)
- [ ] Am I asking for specifics or generics? (Specifics = good)
- [ ] Could someone answer this with a polite lie? (If yes, rewrite it)
- [ ] Am I asking about behavior or opinion? (Behavior = good)

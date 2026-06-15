# Module 0 — Orientation & Mental Model

> Before any tactic, get the mental model right. Everything else is an application of this.

---

## The one idea: Claude Code is a loop, not a chat

A normal chatbot does: you talk → it talks back. Done.

Claude Code runs an **agentic loop**:

```
read your request
  → decide on an action (call a tool: read file, run command, edit code...)
  → observe the result
  → decide on the next action
  → ... repeat until the task is done
  → report back
```

Each pass through that loop is a **turn**, and each turn sends the *entire current
context* to the model and gets a response back. This single fact explains almost
every cost and quality lesson in this curriculum.

---

## The cost equation

You are billed (roughly) per **token**, split into:

- **input tokens** — everything the model reads: your message, the system prompt,
  tool definitions, file contents, prior conversation, tool results.
- **output tokens** — everything the model writes: its reasoning, text, tool calls.

Output tokens are several times more expensive than input tokens, but **input
volume is usually the silent budget-killer** because the whole context is re-sent
*every single turn*.

> Total cost ≈ Σ over every turn of:  (input tokens + output tokens) × model's rate

Three things you control sit inside that formula:

| Lever | What it changes | Module |
|-------|-----------------|--------|
| **Model choice** | the per-token rate (Haiku ≪ Sonnet ≪ Opus) | 2 |
| **Context size** | input tokens *per turn* (and it compounds!) | 1 |
| **Turn count** | how many times you pay for the whole context | 1, 3, 7 |

Everything in this course is ultimately about pulling one of these three levers.

---

## Why context size compounds (the key intuition)

Say a task takes 10 turns. If your context is 50K tokens, you don't pay for 50K —
you pay for roughly 50K × 10 = 500K input tokens, because the context rides along
on every turn. Bloat the context to 150K and you now pay ~1.5M. **A bigger context
doesn't cost more once — it costs more on every remaining turn.**

That's why Module 1 (context management) comes before the flashy stuff: it's the
lever with the largest multiplier.

### "But doesn't prompt caching fix this?" (important nuance)

Claude Code **automatically uses prompt caching**, and it genuinely softens the
compounding — but it doesn't eliminate it. Worth understanding precisely, because
it changes *which* part of context you should worry about.

**What caching does:** it caches a stable **prefix** of your context (system prompt
→ tool definitions → CLAUDE.md → earlier, unchanged conversation). On the next
turn that prefix is re-read at roughly **10% of normal input price** instead of
full price. So the "re-sent every turn" tax on the *unchanging* part drops ~90%.

**Why context size still matters anyway:**

1. **Cache reads cost ~10%, not 0%.** Ten percent of a 150K context every turn is
   still far more than ten percent of a 30K context. Smaller still wins — gentler
   slope, same direction.
2. **Only the stable prefix is cached.** Every turn appends new content (your
   message, tool results, the model's reply). That tail was never cached, so it's
   billed at **full price** — and caching it for next time adds a **~25% write
   premium**. The *growing conversation tail* is the expensive part.
3. **The cache expires** (~5-minute TTL, refreshed on use). Step away for ten
   minutes and the next turn re-pays full price to rebuild the whole cache.
4. **Invalidation cascades.** Changing anything in the prefix (e.g. editing
   CLAUDE.md mid-session) invalidates everything after it — you re-pay to rewrite
   it. This is why **stable, append-only context is cheap and churny context is not.**

**Refined takeaway:** caching makes the cached *prefix* cheap to carry, so the
things that actually drive cost become (a) the **growing conversation tail** and
(b) **anything that busts the cache**. That's exactly what Module 1's habits target:
`/clear` between tasks (resets everything), and keeping CLAUDE.md stable (don't
invalidate the prefix).

---

## Harness vs. model (don't confuse them)

- The **model** (Opus/Sonnet/Haiku) is the brain — it reasons and decides.
- The **harness** (the Claude Code program) is the body — it runs tools, enforces
  permissions, executes hooks, manages your context window.

Why it matters for cost: work you can push onto the **harness** (a hook, a script,
a deterministic command) is effectively *free* — it doesn't burn model tokens.
Work you hand to the **model** costs tokens. A recurring theme: "can the harness do
this instead of the model?"

---

## 🔬 Hands-on: see the loop and the cost

You don't need to write any code for Module 0. Just observe.

1. **Watch the loop.** Ask Claude Code something that needs more than one action,
   e.g. *"How many markdown files are in this project and what are their titles?"*
   Notice it makes multiple tool calls (a search, then reads) before answering.
   Each of those was a turn.

2. **See your token usage.** Run the built-in command:

   ```
   /cost
   ```

   It reports tokens and spend for the current session. Note the number.

3. **Feel the compounding.** Now ask three more follow-up questions in the same
   session, then run `/cost` again. The cost climbs faster than you'd expect for
   short questions — because each turn re-sent the growing conversation. That gap
   *is* the context-size lever, and it's what Module 1 teaches you to control.

> If `/cost` isn't available in your setup, `/context` shows what's currently
> filling the context window instead — also useful.

---

---

## Where the *actionable* cost-cutting lives

This module is the **why**. You'll want the **what-do-I-actually-do**, so here's the
promise for the rest of the course:

- **Every module ends with a concrete "cost move"** — a specific habit or command,
  not just understanding. (Module 1's is "`/clear` between unrelated tasks.")
- **Module 8 assembles them all into one playbook** — a single checklist/decision
  list you can run through when you sit down to work, drawn from modules 1–7.

So: learn the mechanism here, collect one practical move per module, and walk away
with a consolidated cost playbook at the end.

---

## Takeaways

- Claude Code is an **agentic loop**; every turn re-sends the whole context.
- Cost ≈ (input + output tokens) × rate, summed over turns.
- You have exactly three levers: **model choice, context size, turn count**.
- **Context size compounds** across turns — it's the highest-leverage lever.
- Push work to the **harness** (free) instead of the **model** (paid) when you can.

**Next:** Module 1 — Context Management, the lever with the biggest multiplier.

# Module 0 — Orientation & Mental Model

Get the mental model right first; the rest of the course is application.

---

## Claude Code is a loop, not a chat

A normal chatbot does one thing: you talk, it talks back. Claude Code runs an
agentic loop:

```
read your request
  → decide on an action (call a tool: read file, run command, edit code...)
  → observe the result
  → decide on the next action
  → ... repeat until the task is done
  → report back
```

Each pass through that loop is a turn, and each turn sends the entire current
context to the model and gets a response back. That fact explains most of the cost
and quality lessons that follow.

---

## The cost equation

You are billed per token, split into:

- **input tokens** — everything the model reads: your message, the system prompt,
  tool definitions, file contents, prior conversation, tool results.
- **output tokens** — everything the model writes: its reasoning, text, tool calls.

Output tokens cost several times more than input tokens, but input volume is
usually what quietly drives the bill, because the whole context is re-sent on every
turn.

> Total cost ≈ Σ over every turn of:  (input tokens + output tokens) × model's rate

Three things you control sit inside that formula:

| Lever | What it changes | Module |
|-------|-----------------|--------|
| Model choice | the per-token rate (Haiku < Sonnet < Opus) | 2 |
| Context size | input tokens per turn, which compounds | 1 |
| Turn count | how many times you pay for the whole context | 1, 3, 7 |

Everything in this course pulls one of these three levers.

---

## Why context size compounds

Say a task takes 10 turns. If your context is 50K tokens, you don't pay for 50K —
you pay roughly 50K × 10 = 500K input tokens, because the context rides along on
every turn. Bloat it to 150K and you pay about 1.5M. A bigger context doesn't cost
more once; it costs more on every remaining turn.

That is why context management (Module 1) comes before the rest: it has the largest
multiplier.

### "But doesn't prompt caching fix this?"

Claude Code uses prompt caching automatically, and it does soften the compounding —
but it doesn't eliminate it, and it changes which part of context to worry about.

What caching does: it caches a stable prefix of your context (system prompt → tool
definitions → CLAUDE.md → earlier, unchanged conversation). On the next turn that
prefix is re-read at roughly 10% of normal input price instead of full price, so
the re-sent cost of the unchanging part drops about 90%.

Why context size still matters:

1. Cache reads cost about 10%, not 0%. Ten percent of a 150K context every turn is
   still far more than ten percent of a 30K context. Smaller still wins.
2. Only the stable prefix is cached. Every turn appends new content — your message,
   tool results, the model's reply — which is billed at full price, plus about a
   25% premium to write it into the cache for next time. The growing conversation
   tail is the expensive part.
3. The cache expires (about a 5-minute TTL, refreshed on use). Step away for ten
   minutes and the next turn pays full price to rebuild it.
4. Invalidation cascades. Changing anything in the prefix — editing CLAUDE.md
   mid-session, for example — invalidates everything after it, and you re-pay to
   rewrite it. Stable, append-only context is cheap; churning context is not.

So caching makes the cached prefix cheap to carry, which means the real cost drivers
become the growing conversation tail and anything that busts the cache. That is what
Module 1's habits target: `/clear` between tasks, and keeping CLAUDE.md stable.

---

## Harness vs. model

- The model (Opus/Sonnet/Haiku) is the brain — it reasons and decides.
- The harness (the Claude Code program) is the body — it runs tools, enforces
  permissions, executes hooks, and manages your context window.

This matters for cost: work you push onto the harness — a hook, a script, a
deterministic command — costs no model tokens, while work you hand to the model
costs tokens. A recurring question in this course is "can the harness do this
instead of the model?"

---

## Hands-on: see the loop and the cost

No code required. Just observe.

1. Ask Claude Code something that needs more than one action, e.g. *"How many
   markdown files are in this project and what are their titles?"* It makes several
   tool calls — a search, then reads — before answering. Each was a turn.

2. Run `/cost`. It reports tokens and spend for the current session. Note the
   number.

3. Ask three more short follow-up questions in the same session, then run `/cost`
   again. The cost climbs faster than the questions' length would suggest, because
   each turn re-sent the growing conversation. That is the context-size lever, and
   Module 1 is about controlling it.

> If `/cost` isn't available in your setup, `/context` shows what's currently
> filling the context window — also useful.

---

## Where the actionable cost-cutting lives

This module is the why. For the what-to-do:

- Every module ends with a concrete cost move — a specific habit or command, not
  just understanding. Module 1's is "`/clear` between unrelated tasks."
- Module 8 assembles them into one checklist drawn from Modules 1–7.

Learn the mechanism here, collect one practical move per module, and end with a
consolidated checklist.

---

## Takeaways

- Claude Code is an agentic loop; every turn re-sends the whole context.
- Cost ≈ (input + output tokens) × rate, summed over turns.
- There are three levers: model choice, context size, turn count.
- Context size compounds across turns, so it has the biggest multiplier.
- Push work to the harness (free) instead of the model (paid) where you can.

**Next:** Module 1 — Context Management, the lever with the biggest multiplier.

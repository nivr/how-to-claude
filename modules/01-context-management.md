# Module 1 — Context Management

> The highest-leverage skill, because context size is an input cost that
> **compounds on every turn** (see Module 0). Master this and everything else
> gets cheaper automatically.

---

## What "context" actually contains

At any moment, the model's context window holds:

1. The **system prompt** (Claude Code's own instructions — fixed, you don't control it)
2. **Tool definitions** (schemas for every available tool — MCPs add to this; Module 5)
3. **CLAUDE.md** files (your project/user instructions — loaded every session)
4. **Recalled memory** (persistent facts, if you use the memory feature)
5. The **conversation so far** (every message + every tool result)
6. The **files and command outputs** you've pulled in this session

Items 3–6 are the ones you influence. The biggest variable is usually #5 and #6:
conversation history and the file/command output you drag in.

---

## The mental rule

> **Keep in context only what the current task needs. Nothing more.**

Every token that isn't helping the current task is still being paid for, every
turn. A 2,000-line file you read "just in case" is a tax on all subsequent turns.

---

## Tool 1: `/context` and `/cost` — measure first

You can't manage what you can't see.

- `/context` — shows what's currently *filling* the window (system prompt, tools,
  messages, files). This tells you *where* the bloat is.
- `/cost` — shows tokens/spend so far. This tells you *how much* it's costing.

Always diagnose before optimizing.

---

## Tool 2: `/clear` — the cheapest optimization there is

When you finish a task and start an unrelated one, the old conversation is pure
dead weight — it'll ride along on every future turn for no benefit.

```
/clear
```

wipes the conversation back to empty (keeps CLAUDE.md). **Starting a new task in
a fresh session is the single most effective cost habit.** New task → `/clear`.

Rule of thumb: if your next question doesn't depend on the last 20 messages,
clear them.

---

## Tool 3: compaction — know what it does *to* you

When context gets near full, Claude Code **compacts**: it summarizes the
conversation so far into a shorter form and continues. You can also trigger it:

```
/compact
```

Trade-offs:
- ✅ Lets a long task continue without overflowing the window.
- ⚠️ The summary is lossy — specific details (exact strings, line numbers) can be
  dropped or blurred.

Best practice: don't *rely* on compaction as your context strategy. Prefer to
`/clear` between tasks and keep individual tasks scoped. Let compaction be the
safety net, not the plan. If you know a detail must survive, write it to a file
or memory (below) rather than trusting the summary.

---

## Tool 4: CLAUDE.md — persistent project context, used well

`CLAUDE.md` is auto-loaded into context every session. That makes it powerful and
also a **standing cost** — every token in it is paid on every turn of every
session forever. So:

- ✅ Put in CLAUDE.md: stable, high-value facts the model needs *often* — build/test
  commands, project conventions, "always do X" rules, architecture in one paragraph.
- ❌ Keep out of CLAUDE.md: things needed rarely, long reference dumps, anything
  that changes constantly. Put those in separate files and let the model read them
  *on demand* — pay for them only when relevant.

The skill is curation: CLAUDE.md should be short and dense, not a wiki.

`/init` generates a starter CLAUDE.md by scanning the project — a good first draft
to then trim.

---

## Tool 5: reference-on-demand instead of load-everything

A subtle but big one. Two ways to give the model a 3,000-line spec:

- **Load-everything:** paste/read the whole spec into context now. You pay for all
  3,000 lines on every remaining turn.
- **Reference-on-demand:** tell the model *where* the spec is ("see `docs/spec.md`,
  section 4 covers auth") and let it read only the part it needs, when it needs it.

The second is dramatically cheaper for large material. Same idea applies to
subagents (Module 3): a subagent can read a huge file and return only the 5-line
conclusion to the main context.

---

## 🔬 Hands-on: prove the lever to yourself

A 4-step exercise in this repo. ~5 minutes.

1. **Baseline.** In a fresh session run `/context`. Note roughly how full the
   window is (it'll be mostly system prompt + tools — quite empty).

2. **Bloat it.** Ask Claude to read every file in this project and summarize each.
   Run `/context` again. Watch the "messages/files" portion jump — those file
   contents are now resident and will be re-sent every turn.

3. **Measure the tax.** Run `/cost`, note it. Ask two short follow-up questions
   ("what's the title of module 0?"). Run `/cost` again. The per-question cost is
   inflated because every trivial question is dragging all those file contents along.

4. **Reset.** Run `/clear`, then re-run `/context`. Back to near-empty. Ask the
   same short question — far cheaper now. **That delta is the entire lesson.**

Bonus: open [CURRICULUM.md](../CURRICULUM.md) and imagine which facts belong in a
`CLAUDE.md` (stable, frequently needed) vs. which belong in an on-demand file
(this whole curriculum — read a module only when working on it).

---

## Takeaways

- Context = system prompt + tools + CLAUDE.md + memory + conversation + pulled-in
  files. You mostly control the last three.
- **Measure** with `/context` (where) and `/cost` (how much) before optimizing.
- **`/clear` between unrelated tasks** — the cheapest, highest-impact habit.
- **Compaction is a lossy safety net,** not a strategy; persist must-keep details
  to files/memory.
- **CLAUDE.md is a standing tax** — keep it short, dense, high-frequency only.
- Prefer **reference-on-demand** over load-everything for large material.

**Next:** Module 2 — Models & Effort Levels, the lever that changes the per-token rate.

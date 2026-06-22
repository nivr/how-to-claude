# Module 1 — Context Management

Context size is an input cost that compounds on every turn (Module 0), so managing
it is the highest-leverage habit. Get this right and the rest gets cheaper.

---

## What context contains

At any moment, the model's context window holds:

1. The system prompt — Claude Code's own instructions; fixed, you don't control it.
2. Tool definitions — schemas for every available tool; MCPs add to this (Module 5).
3. CLAUDE.md files — your project/user instructions, loaded every session.
4. Recalled memory — persistent facts, if you use the memory feature.
5. The conversation so far — every message and every tool result.
6. Files and command output you've pulled in this session.

You influence items 3–6. The largest variables are usually 5 and 6: conversation
history and the file or command output you pull in.

---

## The rule

Keep in context only what the current task needs. Every token that isn't helping is
still paid for, every turn. A 2,000-line file read "just in case" is a tax on all
subsequent turns.

---

## Measure first: `/context` and `/cost`

You can't manage what you can't see.

- `/context` shows what is filling the window right now — system prompt, tools,
  messages, files. It tells you where the bloat is.
- `/cost` shows tokens and spend so far. It tells you how much it costs.

Diagnose before optimizing.

---

## `/clear` between unrelated tasks

When you finish a task and start an unrelated one, the old conversation is dead
weight that rides along on every future turn for no benefit.

```
/clear
```

resets the conversation to empty while keeping CLAUDE.md. Starting a new task in a
fresh session is the most effective cost habit available. Rule of thumb: if your
next question doesn't depend on the last 20 messages, clear them.

---

## Compaction: a fallback, not a strategy

When context approaches full, Claude Code compacts — it summarizes the conversation
so far into a shorter form and continues. You can also trigger it with `/compact`.

The trade-off: compaction lets a long task continue without overflowing the window,
but the summary is lossy — exact strings and line numbers can be dropped or blurred.
Don't rely on it as your primary strategy. Prefer to `/clear` between tasks and keep
tasks scoped, and treat compaction as the safety net. If a detail must survive,
write it to a file or memory rather than trusting the summary.

---

## CLAUDE.md: persistent context, curated

`CLAUDE.md` is auto-loaded every session, which makes it useful and also a standing
cost — every token in it is paid on every turn of every session.

- Put in CLAUDE.md: stable, high-value facts the model needs often — build and test
  commands, project conventions, firm "always/never" rules, the architecture in a
  paragraph.
- Keep out: anything needed rarely, long reference material, anything that changes
  constantly. Put those in separate files and let the model read them on demand, so
  you pay for them only when relevant.

The work is curation: CLAUDE.md should be short and dense, not a wiki. `/init`
generates a starter file by scanning the project — a reasonable first draft to trim.

---

## Reference on demand instead of loading everything

Two ways to give the model a 3,000-line spec:

- Load everything: read the whole spec into context now, and pay for all 3,000 lines
  on every remaining turn.
- Reference on demand: tell the model where the spec is ("see `docs/spec.md`,
  section 4 covers auth") and let it read only the part it needs, when it needs it.

The second is far cheaper for large material. The same idea drives subagents
(Module 3): a subagent can read a large file and return only the conclusion to the
main context.

---

## Hands-on: prove the lever

A four-step exercise in this repo, about five minutes.

1. Baseline. In a fresh session, run `/context`. Note how empty it is — mostly
   system prompt and tools.

2. Bloat it. Ask Claude to read every file in this project and summarize each. Run
   `/context` again. The messages/files portion jumps; those file contents are now
   resident and will be re-sent every turn.

3. Measure the tax. Run `/cost` and note it. Ask two short follow-up questions
   ("what's the title of module 0?"). Run `/cost` again. The per-question cost is
   inflated because each trivial question drags all those file contents along.

4. Reset. Run `/clear`, then `/context` again — back to near-empty. Ask the same
   short question; it's far cheaper now. That difference is the lesson.

Bonus: skim the module files in this folder and decide which facts would belong in a
`CLAUDE.md` (stable, frequently needed) versus an on-demand file (the curriculum
itself — read a module only when working on it).

---

## Takeaways

- Context = system prompt + tools + CLAUDE.md + memory + conversation + pulled-in
  files. You mostly control the last three.
- Measure with `/context` (where) and `/cost` (how much) before optimizing.
- `/clear` between unrelated tasks is the cheapest, highest-impact habit.
- Compaction is a lossy safety net, not a strategy; persist must-keep details to
  files or memory.
- CLAUDE.md is a standing cost — keep it short, dense, and high-frequency only.
- Prefer reference-on-demand over loading everything for large material.

**Next:** Module 2 — Models & Effort Levels, the lever that changes the per-token rate.

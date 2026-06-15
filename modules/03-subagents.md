# Module 3 — Subagents

> A subagent is a **second Claude with its own fresh context window**, dispatched
> to do a job and report back. It's the single most powerful cost-orchestration
> tool you have — because it combines *both* levers from Modules 1 and 2: it keeps
> your main context lean **and** can run on a cheaper model.

---

## What a subagent actually is

When the main Claude (the "orchestrator") spawns a subagent, the subagent:

1. Starts with a **blank context** — it does *not* see your conversation history.
2. Gets a **task description** you (or the orchestrator) write for it.
3. Does the work in **its own context window** — all its file reads, tool calls,
   and reasoning happen over there, not in your main thread.
4. Returns **only a final report** to the orchestrator.

```
Main thread (orchestrator)                Subagent (isolated)
──────────────────────────                ───────────────────
"find every place we call the    ──spawn──▶  fresh context
 old auth API"                                reads 40 files
                                              greps, reasons
       main context stays small   ◀─report──  "Found 7 call sites:
       (just gets the 7-line list)             auth.py:12, ..."
```

The key word is **isolation**. The 40 files the subagent read never enter your
main context. Your orchestrator only pays — going forward — for the short report.

---

## Why this saves money (two levers at once)

**Lever 1 — context isolation (from Module 1).** Recall that context size
*compounds*: every token in your main thread is re-sent on every future turn. A
subagent that reads 40 files and returns a 10-line summary means those 40 files
are paid for **once, in the subagent**, instead of riding along in your main
context for the rest of the session. The expensive part (bulk reading/searching)
is quarantined.

**Lever 2 — model routing (from Module 2).** A subagent can run on a **different,
cheaper model** than your main session. Your Opus orchestrator can dispatch the
mechanical grunt work to a **Haiku** subagent. And unlike switching the main
model, this **doesn't bust your main cache** (Module 0) — the cheap model runs in
its own isolated context while your main cached prefix stays intact.

> "Opus thinks, Haiku fetches." That one sentence is the essence of cost
> orchestration, and subagents are how you express it.

---

## The catch: cold-start cost

A subagent isn't free. Because it starts blank, it has to **re-derive context** —
figure out the project layout, re-read files the main thread already knew about,
orient itself. That's real tokens spent on spin-up.

So subagents are a **trade**:

| | |
|---|---|
| **You save** | the bulk work staying out of main context (compounds) |
| **You spend** | the subagent's cold-start re-derivation (one-time) |

The trade pays off when the isolated job is **big** (lots of reading/searching) so
the isolation savings dwarf the spin-up cost. It's a **net loss** for trivial
jobs — spawning a subagent to rename one variable means paying a cold start to
save nothing.

> Rule: subagent for **broad or bulky** work (search a whole repo, read many
> files, an independent sub-task). Do it **inline** for anything small, sequential,
> or that needs your conversation's context.

---

## Built-in subagents you already have

Claude Code ships with ready-made agent types — no setup needed:

- **Explore** — read-only, broad fan-out search. "Where is X handled across the
  codebase?" It sweeps many files and returns the conclusion, not the file dumps.
  *This is the workhorse and your best first subagent.*
- **Plan** — designs an implementation plan for a task without executing it.
- **general-purpose** — a full-capability agent for open-ended multi-step jobs.

These cover most needs. You reach for *custom* agents when you want to **restrict
tools** or **pin a cheaper model** for a repeated kind of task.

---

## Custom subagents (where model routing gets real)

A custom subagent is just a markdown file in `.claude/agents/`. The frontmatter is
where the cost levers live:

```markdown
---
name: grunt-searcher
description: Use for broad read-only searches across the repo. Returns file:line hits only.
tools: Read, Grep, Glob       # ← restrict tools: can't edit, can't run commands
model: haiku                  # ← THE cost lever: cheap model for cheap work
---

You are a search specialist. Given a query, find all relevant locations in the
codebase and return them as a concise list of `file:line — one-line description`.
Do not read entire large files; grep first, then read only the matching regions.
Return only the list. No preamble.
```

Three cost-relevant knobs:

- **`model: haiku`** — runs this agent on the cheapest model. Mechanical search
  doesn't need Opus. This is Module 2's lever, made permanent for this agent.
- **`tools:`** — restricting tools keeps the agent focused and cheap (it can't
  wander into expensive actions) and is safer (a read-only searcher can't edit).
- **A tight system prompt** that says *"return only the list, no preamble"* — fewer
  output tokens (the 5×-priced ones).

---

## Orchestration patterns (the payoff)

Once you have subagents, you compose them:

1. **Fan-out search.** Spawn several read-only Explore/search agents *in parallel*,
   each on a different question, all on Haiku. They search simultaneously; you get
   back a handful of summaries. Fast and cheap.

2. **Plan → execute.** A Plan subagent (or Opus orchestrator) produces the plan;
   cheaper execution agents carry out the mechanical parts.

3. **Quarantine the bulky read.** Before a task needs a big file/spec, have a
   subagent read it and return just the relevant 10 lines — keeping the 2,000-line
   file out of your main context entirely (Module 1's reference-on-demand, enforced).

**Anti-patterns** (cold-start waste): spawning a subagent for a single-file read,
a one-line edit, or anything that needs the back-and-forth of your current
conversation.

---

## 🔬 Hands-on: watch isolation keep your context lean

The whole point — bulk work happens "over there." ~5 minutes.

1. **Baseline.** Fresh session, run `/context`. Note how empty it is.

2. **Dispatch a subagent.** Ask: *"Use a subagent to explore this project and
   report back what each module file covers, one line each."* Claude will spawn an
   Explore/Task agent. It reads all the module files **in its own context**.

3. **Check the damage.** When it reports back, run `/context` again. The main
   thread grew by roughly the size of the **summary** — not the size of all the
   files the subagent read. The bulk reading stayed quarantined.

4. **Contrast.** Compare this to the Module 1 exercise, where you had the *main*
   thread read every file and watched the context balloon. Same information
   gathered — but the subagent version left your main context lean, so every
   subsequent turn is cheaper. **That delta is the isolation lever.**

> If you want to also feel Lever 2: skim `.claude/agents/` (create the example
> above), note the `model: haiku` line, and recognize that this entire search just
> *could have* run at 1/5 the per-token rate.

---

## 💸 The cost move for this module

> **Quarantine broad/bulky work in a subagent — and run it on the cheapest model
> that can do the job.** Keep the main thread for reasoning and small edits.
> "Opus orchestrates, Haiku fetches." Don't spawn for trivial tasks (cold-start
> tax with no payoff).

---

## Takeaways

- A subagent is a **fresh-context Claude** that does a job and returns **only a
  report** — the bulk work never touches your main context.
- It stacks **both levers**: context isolation (Module 1) *and* cheap-model routing
  (Module 2), without busting the main cache.
- It costs a **cold start** (re-derivation), so it pays off for **broad/bulky**
  work and wastes money on **trivial** work.
- Use built-in **Explore / Plan / general-purpose** for most needs; write **custom
  agents** (`.claude/agents/*.md`) to pin `model: haiku` and restrict `tools:`.
- Compose them: **fan-out search**, **plan→execute**, **quarantine the bulky read**.

**Next:** Module 4 — Skills, reusable expertise that loads only when needed.

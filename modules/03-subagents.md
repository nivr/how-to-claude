# Module 3 — Subagents

A subagent is a second Claude with its own fresh context window, dispatched to do a
job and report back. It combines both levers from Modules 1 and 2: it keeps your
main context lean and can run on a cheaper model.

---

## What a subagent is

When the main Claude (the orchestrator) spawns a subagent, the subagent:

1. Starts with a blank context — it does not see your conversation history.
2. Gets a task description that you or the orchestrator write for it.
3. Does the work in its own context window — its file reads, tool calls, and
   reasoning happen there, not in your main thread.
4. Returns only a final report to the orchestrator.

```
Main thread (orchestrator)                Subagent (isolated)
──────────────────────────                ───────────────────
"find every place we call the    ──spawn──▶  fresh context
 old auth API"                                reads 40 files
                                              greps, reasons
       main context stays small   ◀─report──  "Found 7 call sites:
       (just gets the 7-line list)             auth.py:12, ..."
```

The point is isolation. The 40 files the subagent read never enter your main
context; the orchestrator pays only for the short report from then on.

---

## Why this saves money

Two levers apply at once.

Context isolation (Module 1). Context size compounds: every token in your main
thread is re-sent on every future turn. A subagent that reads 40 files and returns a
10-line summary means those 40 files are paid for once, in the subagent, instead of
riding along in your main context for the rest of the session. The bulk reading and
searching is quarantined.

Model routing (Module 2). A subagent can run on a cheaper model than your main
session — an Opus orchestrator dispatching mechanical work to a Haiku subagent. And
unlike switching the main model, this doesn't bust your main cache (Module 0): the
cheaper model runs in its own isolated context while your main cached prefix stays
intact.

The shorthand: Opus thinks, Haiku fetches. Subagents are how you express it.

---

## The catch: cold-start cost

A subagent isn't free. Because it starts blank, it has to re-derive context — figure
out the project layout, re-read files the main thread already knew about, orient
itself. That is real spin-up cost.

So a subagent is a trade:

| | |
|---|---|
| You save | the bulk work staying out of main context, which compounds |
| You spend | the subagent's cold-start re-derivation, once |

It pays off when the isolated job is large enough that the isolation saving exceeds
the spin-up cost. It's a net loss on trivial jobs — spawning a subagent to rename
one variable pays a cold start to save nothing. Use a subagent for broad or bulky
work (search a whole repo, read many files, an independent sub-task); work inline for
anything small, sequential, or dependent on your current conversation.

---

## Built-in subagents

Claude Code ships with ready-made agent types, no setup required:

- Explore — read-only, broad fan-out search. "Where is X handled across the
  codebase?" It sweeps many files and returns the conclusion, not the file contents.
  This is the one you'll reach for most.
- Plan — designs an implementation plan for a task without executing it.
- general-purpose — a full-capability agent for open-ended multi-step jobs.

These cover most needs. Reach for custom agents when you want to restrict tools or
pin a cheaper model for a repeated kind of task.

---

## Custom subagents

A custom subagent is a markdown file in `.claude/agents/`. The frontmatter holds the
cost levers:

```markdown
---
name: grunt-searcher
description: Use for broad read-only searches across the repo. Returns file:line hits only.
tools: Read, Grep, Glob       # restrict tools: can't edit, can't run commands
model: haiku                  # the cost lever: cheap model for cheap work
---

You are a search specialist. Given a query, find all relevant locations in the
codebase and return them as a concise list of `file:line — one-line description`.
Do not read entire large files; grep first, then read only the matching regions.
Return only the list. No preamble.
```

Three cost-relevant settings:

- `model: haiku` runs this agent on the cheapest model. Mechanical search doesn't
  need Opus. This is Module 2's lever, made permanent for the agent.
- `tools:` restricts what the agent can do, which keeps it focused and cheap and is
  safer — a read-only searcher can't edit.
- A tight system prompt ("return only the list, no preamble") reduces output tokens,
  the 5×-priced ones.

---

## Orchestration patterns

With subagents in place, you compose them:

1. Fan-out search. Spawn several read-only search agents in parallel, each on a
   different question, all on Haiku. They search simultaneously and return a handful
   of summaries.

2. Plan then execute. A Plan subagent or Opus orchestrator produces the plan;
   cheaper execution agents carry out the mechanical parts.

3. Quarantine the bulky read. Before a task needs a large file or spec, have a
   subagent read it and return only the relevant lines, keeping the full file out of
   your main context — Module 1's reference-on-demand, enforced.

Anti-patterns, all cold-start waste: spawning a subagent for a single-file read, a
one-line edit, or anything that needs the back-and-forth of your current
conversation.

---

## Hands-on: watch isolation keep your context lean

About five minutes.

1. Fresh session, run `/context`. Note how empty it is.

2. Ask: *"Use a subagent to explore this project and report back what each module
   file covers, one line each."* Claude spawns an Explore agent that reads the module
   files in its own context.

3. When it reports back, run `/context` again. The main thread grew by roughly the
   size of the summary, not the size of all the files the subagent read. The bulk
   reading stayed quarantined.

4. Compare this to the Module 1 exercise, where the main thread read every file and
   the context ballooned. Same information gathered, but the subagent version left
   your main context lean, so every subsequent turn is cheaper. That is the isolation
   lever.

To see the model lever too: create the example agent above, note the `model: haiku`
line, and recognize that the same search could have run at a fifth of the per-token
rate.

---

## The cost move

Quarantine broad or bulky work in a subagent, and run it on the cheapest model that
can do the job. Keep the main thread for reasoning and small edits. Opus
orchestrates, Haiku fetches. Don't spawn for trivial tasks — that's a cold-start tax
with no return.

---

## Takeaways

- A subagent is a fresh-context Claude that does a job and returns only a report; the
  bulk work never touches your main context.
- It stacks both levers — context isolation (Module 1) and cheap-model routing
  (Module 2) — without busting the main cache.
- It costs a cold start, so it pays off for broad or bulky work and wastes money on
  trivial work.
- Use built-in Explore, Plan, and general-purpose agents for most needs; write
  custom agents (`.claude/agents/*.md`) to pin `model: haiku` and restrict `tools:`.
- Compose them: fan-out search, plan then execute, quarantine the bulky read.

**Next:** Module 4 — Skills, reusable expertise that loads only when needed.

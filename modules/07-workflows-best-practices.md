# Module 7 — Workflows & Best Practices

The most expensive thing in Claude Code isn't tokens per turn — it's turns spent
going the wrong direction. Good workflows minimize wasted work.

---

## The third lever: wasted turns

Total cost sums over every turn (Module 0). Five modules went into reducing the cost
per turn — smaller context, cheaper models, subagents, on-demand skills and tools,
free harness work. This module attacks the number of turns, specifically the ones you
waste building the wrong thing and then unwinding it.

Doing the wrong thing efficiently is still the wrong thing: a 20-turn detour at Haiku
rates costs more than a 3-turn correct path at Opus rates. Direction beats speed.
Every habit below spends a few cheap turns up front to avoid many expensive ones.

---

## Plan before executing

For anything non-trivial, have Claude investigate and propose a plan before it
touches code. In Claude Code this is plan mode: Claude reads what it needs, writes a
step-by-step plan, makes no edits, and presents it for approval.

A plan is cheap to produce and cheap to correct. Fixing a wrong assumption in a
five-line plan costs one comment; fixing it after Claude has written 300 lines across
six files is a token-heavy unwind. Planning surfaces misunderstandings before they
become expensive, and the plan can come from a Plan subagent or a cheaper model
(Modules 2–3) — it doesn't always need your top model. Use plan mode whenever the
task is bigger than a one-liner, or you're not certain Claude shares your
understanding.

---

## Verify the work

Don't assume a change is correct; verify it — tests, a quick run, or the `/verify`
command, which runs the app and observes real behavior.

Verification costs some tokens now, but it catches errors in the loop where they're
cheap to fix. The alternative — discovering the bug three tasks later — means
re-loading context, re-deriving what happened, and fixing it cold, which costs far
more. So verification is cheap insurance against expensive rework. Verify in
proportion to the cost of being wrong: a throwaway script needs little, a change to
auth logic deserves a real check. A Stop hook that runs your tests (Module 6) makes
verification automatic and free of model effort.

---

## One task per session

This is Module 1's habit, raised to a workflow rule.

- Start each distinct task in a clean context with `/clear`. A focused session has a
  small, relevant context — every turn is cheaper, and the model isn't distracted by
  stale history.
- Scope tasks tightly. A sprawling "do everything" session accumulates context and
  wrong turns; a series of small, well-defined tasks stays lean and correctable.

This session is itself an example: it has carried a large resident document for many
turns. In real work, that's the cue to `/clear` and start the next task fresh.

---

## Git hygiene as cheap insurance

Commit at checkpoints, work on branches, keep the working tree clean.

Git makes wrong turns cheap to discard. If Claude goes down a bad path and you have a
clean commit to return to, throwing away the detour costs nothing — `git reset` and
retry. Without checkpoints, you pay the model to unpick a tangled change by hand.
Frequent commits turn an expensive unwind into a free one, and a clean tree also
makes Claude's reasoning easier, since it can see exactly what changed.

---

## Use the built-in review passes

Claude Code ships targeted review commands; use them instead of re-explaining what
"review" means each time:

- `/code-review` reviews the current diff for correctness bugs and cleanups at a
  chosen effort level — low or medium for a few high-confidence findings, high or max
  for broader coverage.
- `/security-review` runs a security-focused pass over pending changes.
- `/simplify` applies reuse and simplification cleanups.

These are scoped — they look at the diff, not the whole repo — and effort-tunable
(Module 2). Run a cheap `low` pass for routine changes and a thorough `max` pass when
correctness is critical. Right-size the review to the risk.

---

## Hands-on: direction beats speed

A two-way comparison, about ten minutes.

1. Pick a small but slightly ambiguous task in this repo — for example, *"add a short
   `README.md` that introduces the course and links the modules in order."* There's
   room to misread intent: tone, depth, which files to link.

2. Path A: in a fresh session, give the task and let Claude execute immediately. Note
   how many turns it takes and whether you correct direction after it's written
   things.

3. Path B: `/clear`, give the same task but ask for a plan first. Read the plan,
   correct anything off (*"link them as a numbered list, keep it under 200 words"*),
   then approve and let it execute once.

4. Compare. Path B usually reaches a better result in fewer corrective turns, because
   the cheap plan caught the wrong turn before it became code. Multiplied across every
   real task, that is the turn-count lever.

Then run `/code-review low` on the result and note that it's scoped to the diff —
cheap, targeted, and effort-tunable.

---

## The cost move

Spend a few cheap turns up front to avoid many expensive ones. Plan before executing
on anything non-trivial; verify in-loop; keep one tightly scoped task per clean
session; commit checkpoints so wrong turns are free to discard; and use scoped,
effort-tuned review passes. Direction beats speed.

---

## Takeaways

- The remaining lever is turn count, and the main waste is turns spent in the wrong
  direction. Doing the wrong thing efficiently is still waste.
- Plan before executing: a plan is cheap to produce and correct; rework after the
  fact is not.
- Verify in-loop (`/verify`, tests, a Stop hook): cheap insurance against expensive
  cold rework later.
- One tightly scoped task per clean session (`/clear`) keeps context lean and wrong
  turns contained.
- Git checkpoints make bad paths free to discard.
- Built-in reviews (`/code-review`, `/security-review`, `/simplify`) are scoped and
  effort-tunable — right-size to the risk.

**Next:** Module 8 — Measuring & Optimizing Cost: the consolidated checklist that
assembles every module's cost move.

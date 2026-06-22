# Module 9 — Putting It Together

The previous modules covered each lever in isolation. This one shows them combined
on realistic work, plus a reference setup and a "which tool when" guide.

---

## Worked example: a feature in an unfamiliar repo

Task: *add rate limiting to an API endpoint in a medium-sized codebase you don't
know well.* Here is a cost-aware way to run it, with the source module for each
decision.

1. **Start clean.** `/clear` if the previous task is unrelated. The session begins
   with a small context. (Module 1)

2. **Map the territory with a subagent.** Don't read the repo into your main
   context. Dispatch an Explore subagent: *"find where API routes are defined, where
   middleware is registered, and any existing throttling code."* It reads many files
   in its own context and returns a short list of locations. Your main context stays
   lean. (Module 3)

3. **Plan before editing.** Enter plan mode. Claude proposes: add a limiter
   middleware, register it on the target route, add config, add a test. You correct
   one assumption (the project already has a Redis client to reuse) before any code
   is written. Correcting the plan costs one sentence; correcting written code would
   not. (Module 7)

4. **Execute on the right model.** The core change needs judgment — keep it on
   Opus or Sonnet. If the plan includes a mechanical part (updating many call sites
   identically), that part can go to a Haiku subagent or a script. (Modules 2–3)

5. **Use a skill if one applies.** If your team has a skill for the project's
   testing conventions, it loads when the test-writing step triggers it — you don't
   re-explain the conventions. (Module 4)

6. **Verify in-loop.** Run the test. Better, have a Stop hook run the suite
   automatically so verification costs no model effort. A failure here is cheap to
   fix; the same bug found next week is not. (Modules 6–7)

7. **Review, scoped to risk.** `/code-review` at `low` for a routine change, higher
   if the path is sensitive. It looks at the diff, not the repo. (Module 7)

8. **Checkpoint.** Commit the working state so the next step can be discarded freely
   if it goes wrong. (Module 7)

No single step is dramatic. The savings come from not reading the whole repo into
main context, not running mechanical work on the top model, and not writing code
before the plan was right.

---

## Which tool when

A quick reference for the choices that get conflated:

| Situation | Use | Module |
|-----------|-----|--------|
| Need a section of a large file | Reference the path; read on demand | 1 |
| Broad search or bulky read | Subagent (isolates it from main context) | 3 |
| Mechanical, repeated task | Cheaper model / Haiku subagent | 2–3 |
| Reach an external system (browser, DB, desktop) | MCP — if a script can't do it and you use it often | 5 |
| One-off external action | Bash / a script (zero standing cost) | 5–6 |
| A workflow you repeat | Skill (invoke with `/name`) | 4–6 |
| A deterministic "always do X" rule | Hook | 6 |
| Non-trivial or ambiguous task | Plan first | 7 |
| Recurring scheduled work | `/loop` or `/schedule`, on a cheap model | 6 |

The mental shorthand from Module 4 still resolves most confusion: **skill = know-how,
subagent = workspace, MCP = new hands, hook = a rule the harness enforces.**

---

## A maintainable personal setup

Most of the value comes from configuring a few things once, then leaving them alone.

- **`CLAUDE.md`** — short and stable. Build/test commands, project conventions, the
  one-paragraph architecture, any firm "always/never" rules. Not a wiki; every line
  is paid every turn. (Module 1)
- **`.claude/skills/`** — a small set of skills for workflows you repeat, with
  specific descriptions so they trigger correctly. (Module 4)
- **`.claude/agents/`** — a couple of custom subagents for common jobs, pinned to
  cheaper models and restricted tool sets. A read-only search agent on Haiku is a
  good first one. (Module 3)
- **`.claude/settings.json`** — hooks for your deterministic rules (format on edit,
  test on stop), plus an allowlist for trusted commands to cut prompt friction. Edit
  it with the `update-config` skill. (Module 6)
- **Memory** — durable facts about you and the project that aren't in the code, so
  they survive across sessions.

Set these up, then most sessions are: `/clear`, state the task, let the
configuration do its work.

---

## Habits that carry the most

If you internalize only a few things from the course:

1. `/clear` between unrelated tasks. Cheapest, most frequent win.
2. Right-size the model before starting. Don't park on Opus for everything.
3. Quarantine bulky reads and searches in subagents.
4. Plan before executing on anything non-trivial.
5. Move deterministic rules to hooks; reach for Bash before MCP.

---

## Takeaways

- The levers compound: a real task combines clean context, a search subagent, plan
  mode, right-sized models, a skill, a verification hook, and a scoped review.
- The "which tool when" table resolves the common confusions; the
  know-how/workspace/hands/rule shorthand covers most of it.
- Configure `CLAUDE.md`, a few skills and agents, and `settings.json` once; then run
  lean sessions.
- The five habits above account for most of the achievable savings.

This is the end of the core curriculum. Revisit any module's "cost move" as a quick
refresher, or Module 8 for the consolidated checklist.

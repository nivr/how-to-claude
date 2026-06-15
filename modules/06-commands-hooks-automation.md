# Module 6 — Custom Commands, Hooks & Automation

> Module 0's deepest cost idea: work you push onto the **harness** is *free* (no
> model tokens); work you hand to the **model** is *paid*. This module is the
> toolkit for that shift — commands, hooks, and automation that get things done
> deterministically without burning a single token of reasoning.

---

## The principle, restated

Every time you ask the model to "remember to do X every time," you pay twice:

1. **Tokens** — the instruction sits in context, and the model spends output
   reasoning about it, every relevant turn.
2. **Reliability** — the model *might* forget, especially as context fills up.

If X is **deterministic** ("always format after editing", "never touch this
directory", "run the tests when you're done"), the harness can do it instead — for
**zero tokens and 100% reliability**. That's the trade this whole module makes.

> Litmus test: *"Is this a judgment call, or a rule?"* Judgment → the model. Rule →
> the harness.

---

## Tool 1: Custom commands (repeatable workflows)

A **slash command** is a workflow you invoke by name — in Claude Code these are
**skills** (Module 4) you trigger explicitly with `/name`. Instead of re-typing a
five-paragraph prompt ("review the diff for X, Y, Z, then…") every time, you write
it once as a command and invoke it.

Cost angle: a command keeps the long instructions **out of your everyday context**
and pays for them only when you run it (progressive disclosure, Module 4). It also
saves *you* time and keeps the workflow consistent. You've already used several —
`/code-review`, `/init`, `/verify` are all commands.

**When to make one:** any multi-step prompt you find yourself typing more than
twice. Package it; invoke it.

---

## Tool 2: Hooks (the big one — deterministic automation)

A **hook** is a shell command the **harness** runs automatically on a specific
event — *not* the model. This is the heart of the module.

Hooks fire on events like:

- **PreToolUse** — before Claude runs a tool (can *block* or modify it)
- **PostToolUse** — after a tool runs (e.g. auto-format an edited file)
- **UserPromptSubmit** — when you send a message
- **Stop** — when Claude finishes responding
- **SessionStart**, **PreCompact**, and more

Because a hook is a deterministic program, it's perfect for rules:

| Want | Hook approach | Why not just tell the model? |
|------|---------------|------------------------------|
| Auto-format code after every edit | **PostToolUse** hook running your formatter | Model might forget; costs tokens each time |
| Block edits to `secrets/` | **PreToolUse** hook that denies the call | A prompt rule can be overlooked; a hook *can't* be |
| Run tests when a task finishes | **Stop** hook running the test command | Deterministic + free vs. paid + unreliable |
| Inject project context at session start | **SessionStart** hook | Guaranteed, every session |

The payoff: the formatter, the guardrail, the test run — all happen for **zero
model tokens** and **every single time**. You've moved the work from the expensive,
fallible reasoner to the free, reliable harness.

> Key distinction: **memory/preferences cannot do this.** Asking Claude to
> "remember to always format" is a *model* behavior — fallible and token-costing.
> A hook is a *harness* behavior — enforced by the program. Automated "whenever
> X / before X / after X" rules **must** be hooks, not memory.

Hooks live in `settings.json`. The exact schema (event names, matchers, the env
vars passed to your command) is worth getting from the source rather than memory —
the **`update-config`** skill configures hooks and permissions for you and knows
the current format.

---

## Tool 3: Permissions & allowlists (cut the friction tax)

Every time Claude asks "allow this command?" you pay a small **interruption tax**.
For commands you trust and run constantly (`git status`, `npm test`, a read-only
query), you can **allowlist** them in `settings.json` so they run without prompting.

- Fewer prompts = smoother flow and fewer turns spent on approval round-trips.
- The **`fewer-permission-prompts`** skill scans your history and proposes a
  sensible allowlist automatically.
- Keep genuinely risky actions (deletes, pushes, sends) *behind* prompts — the goal
  is to remove friction on the safe 90%, not to disable judgment on the risky 10%.

`settings.json` also holds **environment variables** and other harness config —
again, the `update-config` skill is the right tool to edit it safely.

---

## Tool 4: Scheduled & recurring automation

For work that should happen **on a schedule** or **repeatedly**, the harness can
run Claude *for* you:

- **`/loop`** — run a prompt or command on an interval (e.g. "check the deploy
  every 5 minutes"), or let the model self-pace.
- **`/schedule`** — create scheduled cloud agents (cron-style) that run
  automatically — a daily report, a nightly cleanup, a one-time "remind me at 3pm."

Cost angle: these don't make a given run cheaper, but they remove *you* from the
loop for routine work — and let you put recurring jobs on **cheaper models / lower
effort** deliberately (Module 2) since they're unattended.

---

## 🔬 Hands-on: turn a repeated instruction into a free, reliable hook

The simplest possible demonstration of harness-vs-model. ~10 minutes.

1. **Find a rule you keep repeating.** Something deterministic you wish Claude
   always did — e.g. "format Python files after editing them," or "log every time
   you finish."

2. **Add a hook via the `update-config` skill.** Ask: *"Add a PostToolUse hook
   that runs after Edit/Write and appends the edited file path to `hooks.log`."*
   The skill writes the correct `settings.json` entry (it knows the exact schema
   and env-var names for your version — better than hand-editing).

3. **Trigger it.** Make any small edit to a file in this project. Then open
   `hooks.log` — there's your entry. **No model tokens were spent deciding to write
   it.** The harness did it deterministically.

4. **Feel the contrast.** Imagine instead instructing Claude in every session:
   *"remember to log every edit."* That's tokens every turn, and it'd eventually
   slip. The hook is free and never forgets. **That gap is the whole module.**

> Safe-and-minimal variant: a **Stop** hook that appends a timestamp to a file when
> Claude finishes. Zero risk, and you'll *see* it fire on every turn.

---

## 💸 The cost move for this module

> **Convert every "always do X" rule from a model instruction into a harness hook.**
> Rules are free and reliable on the harness; on the model they cost tokens and can
> slip. Allowlist trusted commands to cut prompt friction, and schedule routine
> jobs onto cheaper models. Reserve the model for judgment, not bookkeeping.

---

## Takeaways

- **Harness work is free and reliable; model work is paid and fallible.** Move
  deterministic rules to the harness.
- **Custom commands** package repeated multi-step prompts (invoke with `/name`),
  keeping long instructions out of everyday context.
- **Hooks** are the star: harness-run programs on events (**PreToolUse**,
  **PostToolUse**, **Stop**, …) that enforce rules — auto-format, guardrails, test
  runs — for **zero tokens, every time**. Automated "whenever/before/after" rules
  *must* be hooks, not memory.
- **Allowlists** cut the per-prompt friction tax (`fewer-permission-prompts`); keep
  risky actions gated.
- **`/loop` and `/schedule`** automate recurring work — a good place to deploy
  cheaper models (Module 2).
- Edit `settings.json` via the **`update-config`** skill rather than from memory.

**Next:** Module 7 — Workflows & Best Practices: plan mode, verification, and the
habits that tie the levers together.

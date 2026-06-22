# Module 6 — Custom Commands, Hooks & Automation

Work you push onto the harness is free — no model tokens — while work you hand to the
model is paid (Module 0). This module is the toolkit for that shift: commands, hooks,
and automation that get things done deterministically without spending reasoning
tokens.

---

## The principle

Asking the model to "remember to do X every time" costs you twice:

1. Tokens — the instruction sits in context and the model reasons about it on every
   relevant turn.
2. Reliability — the model might forget, especially as context fills.

If X is deterministic — "always format after editing", "never touch this directory",
"run the tests when you're done" — the harness can do it instead, for no tokens and
with full reliability. The litmus test: is this a judgment call or a rule? Judgment
goes to the model; rules go to the harness.

---

## Custom commands

A slash command is a workflow you invoke by name. In Claude Code these are skills
(Module 4) triggered with `/name`. Instead of re-typing a five-paragraph prompt
("review the diff for X, Y, Z, then…") each time, write it once and invoke it.

This keeps the long instructions out of your everyday context and pays for them only
when you run the command (progressive disclosure, Module 4); it also saves your time
and keeps the workflow consistent. You've used several already — `/code-review`,
`/init`, and `/verify` are all commands. Make one for any multi-step prompt you type
more than twice.

---

## Hooks

A hook is a shell command the harness runs automatically on a specific event, not
the model. Hooks fire on events such as:

- PreToolUse — before Claude runs a tool; can block or modify it.
- PostToolUse — after a tool runs, e.g. to auto-format an edited file.
- UserPromptSubmit — when you send a message.
- Stop — when Claude finishes responding.
- SessionStart, PreCompact, and others.

Because a hook is a deterministic program, it suits rules:

| Want | Hook approach | Why not just tell the model |
|------|---------------|-----------------------------|
| Auto-format code after every edit | PostToolUse hook running your formatter | The model might forget, and it costs tokens each time |
| Block edits to `secrets/` | PreToolUse hook that denies the call | A prompt rule can be overlooked; a hook can't be |
| Run tests when a task finishes | Stop hook running the test command | Deterministic and free, versus paid and unreliable |
| Inject project context at session start | SessionStart hook | Guaranteed, every session |

The formatter, the guardrail, and the test run all happen for no model tokens and on
every relevant event. The work moves from the fallible, token-costing model to the
reliable, free harness.

This is something memory and preferences cannot do. Asking Claude to "remember to
always format" is a model behavior — fallible and token-costing — whereas a hook is
enforced by the program. Automated "whenever X / before X / after X" rules have to be
hooks, not memory.

Hooks live in `settings.json`. Get the exact schema — event names, matchers, the env
vars passed to your command — from the source rather than memory; the `update-config`
skill configures hooks and permissions and knows the current format.

---

## Permissions and allowlists

Every "allow this command?" prompt is a small interruption. For commands you trust
and run constantly — `git status`, `npm test`, a read-only query — allowlist them in
`settings.json` so they run without prompting.

- Fewer prompts means smoother flow and fewer turns spent on approvals.
- The `fewer-permission-prompts` skill scans your history and proposes an allowlist.
- Keep genuinely risky actions — deletes, pushes, sends — behind prompts. The goal is
  to remove friction on the safe majority, not to disable judgment on the risky part.

`settings.json` also holds environment variables and other harness config; the
`update-config` skill is the right tool to edit it.

---

## Scheduled and recurring automation

For work that should happen on a schedule or repeatedly, the harness can run Claude
for you:

- `/loop` runs a prompt or command on an interval ("check the deploy every 5
  minutes"), or lets the model self-pace.
- `/schedule` creates cron-style cloud agents — a daily report, a nightly cleanup, a
  one-time "remind me at 3pm."

These don't make a given run cheaper, but they take you out of the loop for routine
work, and because the jobs are unattended they're a good place to use cheaper models
or lower effort deliberately (Module 2).

---

## Hands-on: turn a repeated instruction into a hook

About ten minutes.

1. Find a deterministic rule you keep repeating — "format Python files after editing
   them," or "log every time you finish."

2. Add a hook with the `update-config` skill: *"Add a PostToolUse hook that runs
   after Edit/Write and appends the edited file path to `hooks.log`."* The skill
   writes the correct `settings.json` entry, including the exact schema and env-var
   names for your version.

3. Make any small edit to a file in this project, then open `hooks.log` — the entry
   is there. No model tokens were spent deciding to write it; the harness did it.

4. Compare that to instructing Claude every session to "remember to log every edit":
   tokens every turn, and it would eventually slip. The hook is free and doesn't
   forget.

A lower-risk variant: a Stop hook that appends a timestamp to a file when Claude
finishes. No risk, and you'll see it fire on every turn.

---

## The cost move

Convert every "always do X" rule from a model instruction into a harness hook. On the
harness, rules are free and reliable; on the model they cost tokens and can slip.
Allowlist trusted commands to cut prompt friction, and schedule routine jobs onto
cheaper models. Reserve the model for judgment, not bookkeeping.

---

## Takeaways

- Harness work is free and reliable; model work is paid and fallible. Move
  deterministic rules to the harness.
- Custom commands package repeated multi-step prompts (invoke with `/name`), keeping
  long instructions out of everyday context.
- Hooks run harness programs on events (PreToolUse, PostToolUse, Stop, and others) to
  enforce rules — auto-format, guardrails, test runs — for no tokens, every time.
  Automated "whenever/before/after" rules must be hooks, not memory.
- Allowlists cut the per-prompt friction (`fewer-permission-prompts`); keep risky
  actions gated.
- `/loop` and `/schedule` automate recurring work, a good place to use cheaper
  models (Module 2).
- Edit `settings.json` with the `update-config` skill rather than from memory.

**Next:** Module 7 — Workflows & Best Practices: plan mode, verification, and the
habits that tie the levers together.

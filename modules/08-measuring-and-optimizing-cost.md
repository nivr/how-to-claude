# Module 8 — Measuring & Optimizing Cost

This module consolidates the cost moves from Modules 1–7 into one reference. Use it
two ways: measure where your tokens go, then apply the matching fix.

---

## Measure first

Two commands tell you everything you need before optimizing.

- `/cost` — tokens and spend for the current session. Run it periodically; watch
  the rate of change, not just the total.
- `/context` — where the context window is being spent right now, broken down by
  category (system prompt, tools, MCP tools, skills, messages, free space).

Diagnose before changing anything. Most "Claude Code is expensive" problems are one
identifiable category in `/context` being larger than it needs to be.

---

## Reading `/context`

Each category maps to a specific fix:

| Category | What it is | If it's large |
|----------|-----------|---------------|
| Messages | Conversation history + files/output pulled in | `/clear` between tasks; stop reading whole files (Module 1) |
| MCP tools | Schemas of connected MCP servers | Disconnect unused servers; scope per-project (Module 5) |
| Skills | Skill descriptions (always resident) | Usually fine — descriptions are cheap; only prune if you have many custom ones (Module 4) |
| System tools | Built-in tool schemas | Mostly fixed; not your lever |
| System prompt | Claude Code's own instructions | Fixed; not your lever |

The two you actually control are **Messages** and **MCP tools**. Those are where
runaway cost almost always lives.

---

## The playbook

Organized by the three levers from Module 0. When you sit down to work, run through
these.

### Lever 1 — Context size (cost per turn, compounds)

- `/clear` when starting an unrelated task. The highest-frequency habit.
- Don't read whole files when you need a section. Reference the path and let Claude
  read the relevant part on demand.
- Keep `CLAUDE.md` short and stable — it's paid every turn, and editing it
  mid-session invalidates the prompt cache.
- Quarantine bulky reads/searches in a subagent so they never enter main context
  (Module 3).
- Let compaction be a fallback, not a plan. Persist must-keep details to files or
  memory.

### Lever 2 — Model and effort (price per token)

- Default to Sonnet for everyday work; drop to Haiku for mechanical tasks; reserve
  Opus/Fable for hard reasoning (Module 2).
- Lower effort for simple tasks; keep it high only where correctness matters.
- Route grunt work to a cheaper-model subagent rather than switching the main model
  (which busts the cache).

### Lever 3 — Turn count (wasted work)

- Plan before executing on anything non-trivial. Correcting a plan is cheap;
  unwinding code is not (Module 7).
- Verify in-loop (`/verify`, tests) so errors are caught while they're cheap.
- Commit checkpoints so wrong turns are free to discard.
- Move deterministic rules to hooks — free and reliable — instead of repeated model
  instructions (Module 6).
- Right-size review passes with effort levels (`/code-review low` vs `max`).

---

## When cost spikes, check in this order

1. `/context` — is one category (usually Messages or MCP tools) oversized?
2. If Messages: did you carry an unrelated task or a large file across many turns?
   `/clear` or start fresh.
3. If MCP tools: are servers connected that you aren't using this session?
4. Are you on a heavier model than the task needs? `/model`.
5. Are you spending turns correcting direction? Switch to planning first.

---

## Hands-on: profile this session

1. Run `/context`. Identify the largest category you control (Messages or MCP
   tools).
2. Run `/cost` and note the current spend.
3. Apply the matching fix — most likely `/clear` if Messages is large, or
   disconnecting an unused MCP server.
4. In a fresh session, run `/context` again and compare. The reclaimed space is the
   per-turn saving, multiplied across every turn you would have paid it.

---

## Cost checklist

Before a task:
- Right model for the difficulty? (`/model`)
- Fresh context if this is a new task? (`/clear`)
- Plan first if the task is non-trivial or ambiguous?

During:
- Reading whole files when a section would do?
- Spawning subagents for bulky/parallel work, on cheap models?
- Spending turns correcting direction instead of replanning?

After:
- Verify proportional to the cost of being wrong.
- Commit the working state.
- Recurring rule worth a hook instead of a repeated instruction?

---

## Takeaways

- Measure with `/context` (where) and `/cost` (how much) before optimizing.
- The two categories you control are **Messages** and **MCP tools**; that's where
  runaway cost lives.
- The playbook is the three levers: shrink context, right-size model/effort, cut
  wasted turns.
- When cost spikes, diagnose top-down from `/context` rather than guessing.

**Next:** Module 9 — Putting It Together: worked examples that combine the levers.

# Module 2 — Models & Effort Levels

Module 1 reduced the number of tokens. This module changes the price per token and
how hard the model thinks. The same work costs a different amount depending on
whether the model and effort match the task.

---

## The lineup and what each costs

Claude Code can run on different models. They differ in capability and in price per
million tokens:

| Model | Input $/1M | Output $/1M | Use it for |
|-------|-----------:|------------:|------------|
| Haiku 4.5 | $1 | $5 | Fast, simple, mechanical work |
| Sonnet 4.6 | $3 | $15 | The balanced default — most coding |
| Opus 4.8 | $5 | $25 | Hardest reasoning, long agentic tasks |
| Fable 5 | $10 | $50 | Maximum capability, when it's warranted |

Two things follow from this table:

1. Output costs about 5× input on every model (Module 0), so a model that rambles
   is expensive twice over.
2. Opus input costs 5× Haiku input. Running every trivial task on the top model
   overpays by a multiple.

The move for this module: don't pay Opus rates for Haiku work.

---

## Match the model to the task's difficulty

Before a task, ask whether it actually needs deep reasoning.

| Task looks like | Right model |
|-----------------|-------------|
| Rename a variable across files, format JSON, run a script, summarize a file, simple lookups | Haiku |
| Write a feature, fix a normal bug, refactor, review a diff — the daily 80% | Sonnet |
| Subtle multi-file reasoning, architecture, a hard bug, long autonomous runs | Opus |
| The rare problem where you want the ceiling | Fable |

Most work is the middle row. The savings come from pushing the bottom row down to
Haiku instead of running it on Opus, and reserving Opus for work that earns its
rate.

---

## Effort: the second dial

Picking a model sets the price per token. Effort sets how many tokens the model
spends thinking before it acts, and therefore how thorough — and expensive — a
given model is on a given task.

Effort levels, cheapest to most thorough: `low`, `medium`, `high`, `xhigh`, `max`.

- Lower effort means fewer, more-consolidated tool calls, less preamble, and terser
  reasoning. Cheaper and faster, but can under-think hard problems.
- Higher effort means more exploration and self-checking before answering. Better on
  hard tasks, and it burns more tokens — though on hard agentic work it often takes
  fewer total turns, so it isn't always more expensive overall.

Claude Code defaults to a high effort setting (`xhigh`) because coding rewards it.
The adjustment you'll make is to drop effort for simple or mechanical tasks and keep
it high only where correctness matters. A classification or a rename does not need
`max`.

Model is how smart; effort is how hard it thinks. They multiply. A cheap model at
low effort is the floor and an expensive model at max effort is the ceiling; pick
the lowest point on that grid that still does the job.

---

## Controlling this in Claude Code

Three levers, simplest first:

1. `/model` switches the model for the current session. Use it to drop to a cheaper
   model before a batch of simple work, or move up to Opus before a hard task.

   ```
   /model        # see or choose the current model
   ```

2. `/fast` toggles fast mode — Opus with faster output. Same top model, lower
   latency, when you want Opus quality without the wait.

3. Per-subagent model is the largest lever, covered in Module 3: a subagent can run
   on a cheaper model than your main session. That is how you get "Opus
   orchestrates, Haiku does the grunt work."

Switching the main model mid-session also busts the prompt cache (Module 0).
Subagents avoid that — the cheaper model runs in its own isolated context while your
main cached prefix stays intact.

---

## Hands-on: the model dial

No code, about three minutes.

1. Run `/model` and note the current model, likely Opus.

2. Ask a trivial task on it: *"List the markdown files in this project and give each
   a one-line description."* Run `/cost` and note the number.

3. Run `/model`, switch to Haiku (or Sonnet), and ask the same question in the same
   session. Run `/cost` again.

4. Compare. The cheaper model handled the mechanical task at a fraction of the
   per-token rate. Multiplied across every simple task you'd otherwise run on Opus,
   that is the saving. Switch back to Opus when you return to hard work.

The cheaper model also tends to be terser — less output means fewer of the
5×-priced output tokens. A cheaper model and lower effort compound.

---

## The cost move

Right-size the model before starting a task. Default to Sonnet for everyday coding,
drop to Haiku for mechanical work, and reserve Opus or Fable for genuinely hard
reasoning. Don't leave every session parked on the top model. The larger version of
this — routing cheap models to subagents — is Module 3.

---

## Takeaways

- Models differ in price per token: Haiku $1/$5, Sonnet $3/$15, Opus $5/$25, Fable
  $10/$50 (input/output per 1M). Output is about 5× input everywhere.
- Match model to task difficulty. Most work is Sonnet; push trivial work to Haiku;
  reserve Opus and Fable for hard reasoning.
- Effort is a second dial — how hard a model thinks (`low` to `max`). Lower it for
  simple tasks, keep it high only where correctness matters.
- Control it with `/model` and `/fast`; the per-subagent lever is Module 3.
- Switching the main model mid-session busts the cache, so prefer subagents for
  cheap-model work.

**Next:** Module 3 — Subagents, where model routing becomes orchestration.

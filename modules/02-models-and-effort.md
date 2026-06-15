# Module 2 — Models & Effort Levels

> Module 1 shrank the *number of tokens*. This module changes the **price per
> token** and how *hard the model thinks*. Same work, different bill — if you
> match the model and effort to the task instead of using a sledgehammer on every
> nail.

---

## The lineup (and what each costs)

Claude Code can run on different models. They differ in capability **and** in
price per million tokens. Current rates:

| Model | Input $/1M | Output $/1M | Use it for |
|-------|-----------:|------------:|------------|
| **Haiku 4.5** | $1 | $5 | Fast, simple, mechanical work |
| **Sonnet 4.6** | $3 | $15 | The balanced default — most coding |
| **Opus 4.8** | $5 | $25 | Hardest reasoning, long agentic tasks |
| **Fable 5** | $10 | $50 | Maximum capability, when it's truly worth it |

Two things to notice:

1. **Output costs ~5× input** on every model (recall Module 0). So a model that
   rambles is doubly expensive.
2. **Opus input costs 5× Haiku input.** Running every trivial task on the top
   model is like taking a taxi to the mailbox — it works, but you're overpaying
   by a multiple.

> The core move of this module: **don't pay Opus rates for Haiku work.**

---

## The mental model: match the model to the task's *difficulty*

Ask one question before a task: *does this actually need deep reasoning?*

| Task looks like… | Right model |
|------------------|-------------|
| Rename a variable across files, format JSON, run a script, summarize a file, simple lookups | **Haiku** |
| Write a feature, fix a normal bug, refactor, review a diff — the daily 80% | **Sonnet** |
| Subtle multi-file reasoning, architecture, a gnarly bug, long autonomous runs | **Opus** |
| The rare problem where you want the absolute ceiling | **Fable** |

Most work is the middle row. The savings come from **pushing the bottom row down
to Haiku** instead of letting it ride on Opus, and **reserving Opus** for when it
earns its rate.

---

## Effort: the *other* dial

Picking a model sets the price per token. **Effort** sets how many tokens the
model spends *thinking* before it acts — and therefore how thorough (and
expensive) a given model is on a given task.

Effort levels, cheapest → most thorough: **`low` → `medium` → `high` → `xhigh` → `max`**

- **Lower effort** → fewer, more-consolidated tool calls, less preamble, terser
  reasoning. Cheaper and faster; can under-think hard problems.
- **Higher effort** → more exploration and self-checking before answering. Better
  on hard tasks; burns more tokens (and on truly hard agentic work, often *fewer
  total turns* — so not always more expensive overall).

Claude Code defaults to a high effort setting (`xhigh`) because coding rewards it.
The lever you'll actually reach for: **drop effort for simple/mechanical tasks**,
and **keep it high only where correctness genuinely matters**. A classification or
a rename does not need `max`.

Rule of thumb: **model = how smart; effort = how hard it thinks.** They multiply.
A cheap model at low effort is the floor; an expensive model at max effort is the
ceiling. Pick the lowest point on that grid that still does the job.

---

## How you actually control this in Claude Code

Three practical levers, simplest first:

1. **`/model`** — switch the model for the current session. Use it to drop to a
   cheaper model before a batch of simple work, or bump to Opus before a hard one.

   ```
   /model        # see/choose the current model
   ```

2. **`/fast`** — toggles **fast mode** (Opus with faster output). Same top-tier
   model, lower latency; handy when you want Opus quality without the wait.

3. **Per-subagent model** — the biggest lever, but it lives in **Module 3**: a
   subagent can run on a *different, cheaper* model than your main session. That's
   how you get "Opus orchestrates, Haiku does the grunt work" — the heart of cost
   orchestration. We build it next module.

> Why routing per-subagent matters: switching the *main* model mid-session also
> busts the prompt cache (Module 0). Subagents sidestep that — the cheap model
> runs in its own isolated context, and your main cached prefix stays intact.

---

## 🔬 Hands-on: feel the model dial

No code — just observe the difference. ~3 minutes.

1. **See where you are.** Run `/model`. Note the current model (likely Opus).

2. **Do a trivial task on the big model.** Ask: *"List the markdown files in this
   project and give each a one-line description."* Then run `/cost`, note the number.

3. **Drop to a cheaper model.** Run `/model` and switch to **Haiku** (or Sonnet).
   Ask the *same* question in the same session. Run `/cost` again.

4. **Compare.** The cheaper model handled this mechanical task fine — at a fraction
   of the per-token rate. That gap, multiplied across every simple task you'd
   otherwise run on Opus, is the entire lesson. Switch back to Opus when you return
   to hard work.

> Bonus intuition: notice the cheaper model also tends to be *terser*. Less output
> = fewer of those 5×-priced output tokens. Cheap model + low effort compounds.

---

## 💸 The cost move for this module

> **Right-size the model before you start a task.** Default to **Sonnet** for
> everyday coding; drop to **Haiku** for mechanical work; reserve **Opus/Fable**
> for genuinely hard reasoning. Don't leave every session parked on the top model.

(And the bigger version of this move — routing cheap models to subagents — is
Module 3.)

---

## Takeaways

- Models differ in **price per token**: Haiku $1/$5, Sonnet $3/$15, Opus $5/$25,
  Fable $10/$50 (input/output per 1M). Output is ~5× input everywhere.
- **Match model to task difficulty.** Most work is Sonnet; push trivial work to
  Haiku; reserve Opus/Fable for hard reasoning.
- **Effort** is a second dial — how hard a model thinks (`low`→`max`). Lower it for
  simple tasks; keep it high only where correctness matters.
- Control it with **`/model`** and **`/fast`**; the big win (per-subagent cheap
  models) is **Module 3**.
- Switching the *main* model mid-session busts the cache — prefer subagents for
  cheap-model work.

**Next:** Module 3 — Subagents, where model routing becomes real orchestration.

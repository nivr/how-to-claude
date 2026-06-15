# Module 4 — Skills

> A skill is **reusable expertise that loads only when it's relevant.** It's
> Module 1's "reference-on-demand" principle, packaged: the full instructions stay
> *out* of context until a task actually needs them, so you carry dozens of
> capabilities for almost no standing cost.

---

## What a skill is

A skill is a folder with a `SKILL.md` file (plus optional bundled scripts and
reference files). The `SKILL.md` has two parts:

```markdown
---
name: pdf-filler
description: Fill out PDF forms from structured data. Use when the user wants to
             populate a PDF form, flatten fields, or generate filled PDFs.
---

<the full instructions — how to do the task, step by step, with any gotchas.
 Can reference bundled scripts in this same folder.>
```

- The **frontmatter `description`** is short and *always* resident in context.
- The **body** is the full expertise — and it is **not** loaded until a task
  matches. Only then does Claude read the whole file.

This two-tier design is called **progressive disclosure**, and it's the entire
cost story.

---

## The cost mechanic: progressive disclosure

Here's the magic, and you already saw it live. When you ran `/context` earlier, the
**Skills** line showed roughly **2.7k tokens for ~20 skills** — about 130 tokens
each. That's *only the one-line descriptions*. The full bodies (which can be
hundreds or thousands of tokens each) were **not** loaded.

```
Always in context:   [desc][desc][desc][desc] ...   ← cheap, ~1 line each
Loaded on demand:    full SKILL.md body              ← only when a task matches
```

So you get to keep a large *library* of capabilities available at a tiny standing
cost. You pay for a skill's full instructions **only on the turns where you
actually use it** — exactly Module 1's "reference-on-demand instead of
load-everything," applied to instructions.

> Contrast with stuffing all that expertise into `CLAUDE.md`: that would be a
> **standing tax** paid every turn of every session (Module 1). Skills move it to
> pay-per-use.

---

## The description is load-bearing

Because the description is the *only* thing Claude sees by default, it's what
decides **whether the skill fires at all**. A vague description = the skill doesn't
trigger when it should (wasted capability) or fires when it shouldn't (wasted
tokens, wrong behavior).

Good descriptions are **specific about *when* to use the skill**, not just what it
does:

- ❌ "Works with documents."
- ✅ "Create, read, or edit Word `.docx` files — tables of contents, headings,
  letterheads, tracked changes. Use whenever the user mentions a Word doc or `.docx`."

If two skills have overlapping descriptions, they **collide** — the wrong one may
trigger. Keeping descriptions distinct is the main maintenance task.

---

## Skill vs. subagent vs. MCP (when to use which)

These three get confused. The distinction:

| Tool | What it gives you | Reach for it when… |
|------|-------------------|--------------------|
| **Skill** | *Instructions/expertise*, loaded on demand | You want Claude to **do a task a specific way** repeatedly (a workflow, a format, a procedure) |
| **Subagent** (Mod 3) | *A separate context* to run work in | You want to **isolate bulky work** or run it on a cheaper model |
| **MCP** (Mod 5) | *New tools/actions* (connect to a system) | Claude needs to **reach an external system** it otherwise can't |

Rough rule: **skill = know-how, subagent = workspace, MCP = new hands.** They
compose — a subagent can use a skill; a skill can tell Claude to use an MCP tool.

---

## Building and maintaining skills

- **Create:** the **`skill-creator`** skill walks you through scaffolding a new
  skill, and can run evals to measure how reliably it triggers. (Yes — a skill for
  making skills.)
- **Iterate:** the most common fix is **tuning the description** so the skill fires
  exactly when intended — no more, no less.
- **Keep bodies lean:** the body is paid for on every turn it's *active*, so apply
  Module 1 thinking inside it too — put rarely-needed detail in **bundled reference
  files** the body points to, not inline.
- **Avoid collisions:** as your library grows, keep descriptions distinct so the
  right skill triggers.

---

## 🔬 Hands-on: see progressive disclosure in the numbers

The cheapest possible demonstration — it's already in front of you. ~5 minutes.

1. **Measure the standing cost.** Run `/context`. Find the **Skills** line. Note
   it's small (a couple thousand tokens) *despite* there being ~20 skills available.
   That number is **descriptions only**. Divide: ~100–130 tokens per skill. That's
   the whole library's resting cost.

2. **Trigger one.** Do something that matches a skill — e.g. ask to *"create a
   simple Word document with a title and two bullet points."* That matches the
   `docx` skill's description, so Claude now reads its **full body**.

3. **Intuit the difference.** The full `docx` instructions are far longer than its
   one-line description — but you only paid for them on *this* task. Twenty other
   skills' bodies are still sitting unloaded at ~130 tokens each.

4. **(Optional) Build the simplest skill.** Create
   `.claude/skills/say-hello/SKILL.md`:

   ```markdown
   ---
   name: say-hello
   description: Greet the user warmly. Use only when the user explicitly says "hello skill test".
   ---

   Respond with exactly: "👋 The skill fired! This text lives in the body and was
   loaded only because the description matched."
   ```

   Now say `hello skill test`. The body text appears — proving the body loaded
   *only* on the trigger. Change the description to something vague and watch
   triggering get unreliable: that's why descriptions matter.

---

## 💸 The cost move for this module

> **Package repeated expertise as a skill, not as bloat in CLAUDE.md.** A skill's
> body is pay-per-use (progressive disclosure); CLAUDE.md is a standing tax. Write
> tight, specific **descriptions** so skills trigger exactly when needed, and push
> rarely-used detail into bundled reference files rather than the always-active body.

---

## Takeaways

- A skill = `SKILL.md` (always-resident **description** + on-demand **body**) plus
  optional bundled files.
- **Progressive disclosure** is the cost win: you carry a big library for ~1 line
  each, paying for full instructions only when a task triggers them — reference-on-
  demand for instructions.
- The **description is load-bearing** — it's the sole trigger signal; make it
  specific about *when* to use the skill, and keep descriptions distinct to avoid
  collisions.
- **Skill = know-how, subagent = workspace, MCP = new hands.** They compose.
- Build/iterate with **`skill-creator`**; keep bodies lean (bundle rarely-used
  detail).

**Next:** Module 5 — MCP Servers, and the "just use a script?" test for when an
MCP is actually worth its cost.

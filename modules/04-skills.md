# Module 4 — Skills

A skill is reusable expertise that loads only when it's relevant. It's Module 1's
reference-on-demand principle, packaged: the full instructions stay out of context
until a task needs them, so you can carry dozens of capabilities for almost no
standing cost.

---

## What a skill is

A skill is a folder with a `SKILL.md` file, plus optional bundled scripts and
reference files. The `SKILL.md` has two parts:

```markdown
---
name: pdf-filler
description: Fill out PDF forms from structured data. Use when the user wants to
             populate a PDF form, flatten fields, or generate filled PDFs.
---

<the full instructions — how to do the task, step by step, with any gotchas.
 Can reference bundled scripts in this same folder.>
```

- The frontmatter description is short and always resident in context.
- The body is the full expertise, and it is not loaded until a task matches. Only
  then does Claude read the whole file.

This two-tier design is called progressive disclosure, and it's where the cost
saving comes from.

---

## How progressive disclosure saves money

You saw this in your own `/context`. The Skills line showed roughly 2.7k tokens for
about 20 skills — around 130 tokens each. That is only the one-line descriptions;
the full bodies, which can run to hundreds or thousands of tokens, were not loaded.

```
Always in context:   [desc][desc][desc][desc] ...   cheap, ~1 line each
Loaded on demand:    full SKILL.md body              only when a task matches
```

So you keep a large library of capabilities available at a small standing cost, and
pay for a skill's full instructions only on the turns you use it — reference-on-
demand from Module 1, applied to instructions. The alternative, putting that
expertise in `CLAUDE.md`, would be a standing cost paid every turn of every session.
Skills move it to pay-per-use.

---

## The description decides whether a skill fires

The description is the only part Claude sees by default, so it determines whether the
skill triggers at all. A vague description means the skill won't fire when it should,
or fires when it shouldn't. Good descriptions are specific about when to use the
skill, not just what it does:

- Weak: "Works with documents."
- Better: "Create, read, or edit Word `.docx` files — tables of contents, headings,
  letterheads, tracked changes. Use whenever the user mentions a Word doc or `.docx`."

If two skills have overlapping descriptions, the wrong one can trigger. Keeping
descriptions distinct is the main maintenance task.

---

## Skill vs. subagent vs. MCP

These three get conflated. The distinction:

| Tool | What it gives you | Reach for it when |
|------|-------------------|-------------------|
| Skill | Instructions/expertise, loaded on demand | You want Claude to do a task a specific way repeatedly — a workflow, a format, a procedure |
| Subagent (Mod 3) | A separate context to run work in | You want to isolate bulky work or run it on a cheaper model |
| MCP (Mod 5) | New tools/actions to connect to a system | Claude needs to reach an external system it otherwise can't |

The shorthand: skill = know-how, subagent = workspace, MCP = new hands. They compose
— a subagent can use a skill, and a skill can tell Claude to use an MCP tool.

---

## Building and maintaining skills

- Create: the `skill-creator` skill scaffolds a new skill and can run evals to
  measure how reliably it triggers.
- Iterate: the most common fix is tuning the description so the skill fires when
  intended and not otherwise.
- Keep bodies lean: the body is paid for on every turn it's active, so apply Module 1
  thinking inside it — put rarely-needed detail in bundled reference files the body
  points to, not inline.
- Avoid collisions: as the library grows, keep descriptions distinct so the right
  skill triggers.

---

## Hands-on: see progressive disclosure in the numbers

About five minutes, mostly using data you already have.

1. Run `/context` and find the Skills line. Note it's a couple thousand tokens
   despite around 20 skills being available — that's descriptions only, roughly
   100–130 tokens per skill. That is the whole library's resting cost.

2. Trigger one. Ask to *"create a simple Word document with a title and two bullet
   points."* That matches the `docx` skill's description, so Claude reads its full
   body.

3. The full `docx` instructions are far longer than its one-line description, but
   you paid for them only on this task. The other skills' bodies remain unloaded at
   ~130 tokens each.

4. Optional: build the simplest skill. Create `.claude/skills/say-hello/SKILL.md`:

   ```markdown
   ---
   name: say-hello
   description: Greet the user warmly. Use only when the user explicitly says "hello skill test".
   ---

   Respond with exactly: "The skill fired! This text lives in the body and was
   loaded only because the description matched."
   ```

   Say `hello skill test`; the body text appears, confirming the body loaded only on
   the trigger. Change the description to something vague and triggering becomes
   unreliable — which is why descriptions matter.

---

## The cost move

Package repeated expertise as a skill rather than as bloat in CLAUDE.md. A skill's
body is pay-per-use; CLAUDE.md is a standing cost. Write specific descriptions so
skills trigger exactly when needed, and push rarely-used detail into bundled
reference files rather than the always-active body.

---

## Takeaways

- A skill is a `SKILL.md` with an always-resident description and an on-demand body,
  plus optional bundled files.
- Progressive disclosure is the saving: you carry a large library for about one line
  each, paying for full instructions only when a task triggers them.
- The description is the sole trigger signal — make it specific about when to use the
  skill, and keep descriptions distinct to avoid collisions.
- Skill = know-how, subagent = workspace, MCP = new hands. They compose.
- Build and iterate with `skill-creator`; keep bodies lean by bundling rarely-used
  detail.

**Next:** Module 5 — MCP Servers, and the "just use a script" test for when an MCP is
worth its cost.

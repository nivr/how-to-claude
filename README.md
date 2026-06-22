# Mastering Claude Code

Using Claude Code effectively, for the best outcomes at the lowest cost.

Each module is a self-contained file in [`modules/`](modules/): a concept, a
worked example you run yourself, and the concrete cost move it implies. Work
through them in order, or jump to one by topic below.

## Modules

0. [Orientation & Mental Model](modules/00-orientation.md) — the agentic loop, the cost equation, and the three levers: model choice, context size, turn count.
1. [Context Management](modules/01-context-management.md) — why context size compounds, and how `/clear`, a lean `CLAUDE.md`, and reference-on-demand control it.
2. [Models & Effort Levels](modules/02-models-and-effort.md) — current pricing, and matching model and effort to task difficulty.
3. [Subagents](modules/03-subagents.md) — isolated-context agents that quarantine bulky work and run on cheaper models.
4. [Skills](modules/04-skills.md) — reusable expertise that loads only when a task triggers it.
5. [MCP Servers](modules/05-mcp-servers.md) — when external tools are worth their standing context cost, and the "just use a script" test.
6. [Commands, Hooks & Automation](modules/06-commands-hooks-automation.md) — moving deterministic rules onto the harness instead of the model.
7. [Workflows & Best Practices](modules/07-workflows-best-practices.md) — plan mode, verification, scoping, and git as ways to cut wasted turns.
8. [Measuring & Optimizing Cost](modules/08-measuring-and-optimizing-cost.md) — `/cost` and `/context`, and the consolidated playbook.
9. [Putting It Together](modules/09-putting-it-together.md) — a worked example, a "which tool when" guide, and a reference setup.

## How to use it

The modules build on each other — context and cost mechanics (0–2) come first
because the later topics (subagents, skills, MCP, automation) are applications of
them. The hands-on exercises use this repository itself, so clone it and run them
as you read. Module 8 is the reference checklist once you know the material.

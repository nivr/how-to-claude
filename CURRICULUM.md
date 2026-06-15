# Mastering Claude Code — Curriculum Skeleton

> Goal: achieve the best possible outcomes at the lowest possible cost.
> Status: **Approved skeleton — expanding module by module.**
>
> **Decisions locked in:**
> - Ordering as below (context + cost levers front-loaded).
> - Format: **hands-on, simplest-possible examples** that still illustrate the learning. Each module = concept → minimal worked example you run yourself → cost note.
> - Each module lives in `modules/` as its own file.
> - Open to new topic suggestions as we go.

---

## 0. Orientation & Mental Model
- What Claude Code actually is (agentic loop, tools, harness vs. model)
- The cost equation: tokens in × tokens out × model rate × number of turns
- The three levers for cost/quality: **model choice**, **context size**, **turn count**
- How to think about "outcome quality per dollar"

## 1. Context Management (the highest-leverage skill)
- Why context size is the dominant cost and quality driver
- CLAUDE.md: project memory, what belongs there vs. not
- Persistent file-based memory vs. in-session context
- Compaction / summarization: how it works, when it hurts
- Keeping context lean: scoping tasks, clearing, `/init`
- Reference files vs. loading everything

## 2. Models & Effort Levels (cost orchestration)
- The model lineup (Opus / Sonnet / Haiku) and when each is correct
- Effort/reasoning levels and their cost impact
- Matching model → task difficulty (cheap models for mechanical work)
- Fast mode and when it pays off
- Practical routing rules / heuristics

## 3. Subagents
- What subagents are and the isolation boundary (fresh context)
- When a subagent saves money vs. when it wastes it (cold-start cost)
- Built-in agent types (Explore, Plan, general-purpose) vs. custom
- Defining custom agents (frontmatter, tool restrictions, model override)
- Orchestration patterns: fan-out search, parallel work, plan→execute
- Anti-patterns (spawning for trivial tasks)

## 4. Skills
- What a skill is and how it differs from a prompt or an MCP
- Anatomy: SKILL.md, description-driven triggering, bundled scripts
- Building a skill (skill-creator), iterating, and measuring/eval
- Maintaining skills: versioning, descriptions, avoiding trigger collisions
- When a skill is the right tool vs. a subagent vs. an MCP

## 5. MCP Servers
- What MCP is, the client/server model, and the tool-deferral cost problem
- When an MCP is worth it vs. a CLI/script (the "just use bash" test)
- Connecting, scoping, and securing MCP servers
- Cost implications: tool-schema bloat, deferred loading
- Common useful MCPs and how to evaluate new ones

## 6. Custom Commands, Hooks & Automation
- Slash commands (skills) for repeatable workflows
- Hooks: deterministic automation the harness runs (not the model)
- Scheduled tasks / loops / cron for recurring work
- Settings.json: permissions, env vars, allowlists

## 7. Workflows & Best Practices
- Plan mode and when to use it
- Test-driven and verification-driven development
- Permission modes and reducing prompt friction
- Git hygiene with Claude Code
- Code review / security review built-ins

## 8. Measuring & Optimizing Cost
- How to observe token usage and spend
- Profiling where the tokens go
- Concrete cost-reduction tactics (recap across modules)
- Building a personal "cost playbook"

## 9. Putting It Together — Reference Architectures
- End-to-end example: a feature built with model routing + subagents + skills
- A maintainable personal setup (CLAUDE.md + skills + commands + settings)
- Checklists and decision trees

---

## Proposed format for each module (to decide)
- Concept explanation → worked example → hands-on exercise → cost notes
- Each module as its own file under `modules/`

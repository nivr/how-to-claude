# Module 5 — MCP Servers

> An MCP server gives Claude **new hands** — tools to reach systems it otherwise
> can't (a browser, a database, your desktop, an API). Powerful, but every
> connected server has a **standing context cost**: its tool definitions take up
> room. This module is about when that cost is worth paying.

---

## What MCP is

**MCP (Model Context Protocol)** is a standard way to plug external tools into
Claude. An MCP *server* exposes a set of tools (e.g. "navigate to URL", "click
element", "run SQL query"); Claude Code is the *client* that can call them.

Recall the framing from Module 4: **skill = know-how, subagent = workspace,
MCP = new hands.** MCP is how Claude gains *capabilities it doesn't natively have*
— controlling a browser, driving your desktop, querying a specific service.

---

## The cost problem: tool schemas live in context

Here's the catch, and your own `/context` shows it vividly. Each tool an MCP
server exposes has a **schema** (name, description, parameters) that has to be
described to the model. Those schemas take up context. From the `/context` you
just ran:

```
MCP tools             3.6k     ← active tool schemas
MCP tools (deferred)  28.7k    ← schemas held out of the active prompt
```

That's **~32k tokens** of tool definitions across your connected servers (Chrome,
computer-use, Preview, scheduled-tasks, and more) — roughly **80 tools**. Every one
is a small standing cost. Connect a lot of servers and you can silently spend tens
of thousands of tokens before you've asked a single question.

> Worse, tool schemas sit at the **front** of the context (Module 0's caching
> note): they're part of the stable prefix. Changing your tool set *invalidates the
> cache*. So a churning set of MCP servers is doubly costly.

---

## The harness's defense: deferred tools

Notice that most of that 32k is **deferred**, not active. This is the harness
applying Module 1's **reference-on-demand** principle *to tool definitions*:

- Only a small set of tool schemas stay **active** in the prompt.
- The rest are **deferred** — their names are known, but the full schema isn't
  loaded until Claude actually needs that tool (via a tool-search step).

This is why your 80 tools cost 3.6k active instead of 32k active. The harness is
quietly doing for tools what you learned to do for files: don't load everything,
reference on demand. **But deferred ≠ free** — deferred schemas still count toward
context, and resolving one mid-task costs a lookup.

---

## The key decision: MCP vs. "just use a script?"

The most important MCP skill is knowing when *not* to use one. Before adding a
server, apply the **"just use a script" test**:

> Could a one-off Bash command or a small script do this instead?

- A script (or the built-in **Bash** tool) costs **zero standing context** — it
  exists only when you run it.
- An MCP server costs context **every turn it's connected**, whether you use it or
  not.

So an MCP earns its standing cost only when **all** of these hold:

1. **You use it repeatedly** — the per-turn cost amortizes over many uses.
2. **A script can't easily do it** — it needs a persistent connection, managed
   auth, an interactive session (a live browser, your desktop), or structured
   access a shell one-liner can't replicate.
3. **The tool surface is reasonably small** — a server exposing 60 tools you'll
   never call is mostly dead weight.

| Need | Better choice |
|------|---------------|
| Run `git log`, parse JSON, hit a REST endpoint once | **Bash / a script** (zero standing cost) |
| Drive a real browser through many steps | **MCP** (Chrome) — a script can't hold the session |
| Query a database you hit constantly all session | **MCP** — amortizes, manages the connection |
| A capability you'll use *once* this month | **Script**, or connect the MCP *only when needed* |

> Rule of thumb: **reach for Bash first; add an MCP only when a script genuinely
> can't do the job and you'll use it often.**

---

## Connecting, scoping, and securing

- **Scope deliberately.** Connect servers per-project when you can, not globally —
  a server you need for one project shouldn't tax every other session. Audit your
  active set periodically (your `/context` is the audit tool).
- **Fewer tools = leaner + safer.** A server with a focused tool set costs less
  context *and* gives Claude a smaller surface to misuse. Prefer narrow servers.
- **Security matters more with MCP** because you're granting *real capabilities* —
  desktop control, browser control, data access. Treat an MCP server like granting
  an app permissions: only connect ones you trust, and be aware that a tool with
  side effects (sending, deleting, paying) can act. (Claude Code gates many of
  these behind permission prompts — Module 6.)

---

## 🔬 Hands-on: audit your MCP cost

The exercise is reading the bill you already have. ~5 minutes.

1. **Read the standing cost.** In the `/context` you just ran, find the two MCP
   lines: **`MCP tools` (~3.6k active)** and **`MCP tools (deferred)` (~28.7k)**.
   That ~32k is the tax for having all those servers connected — paid on top of
   everything else, before any work.

2. **Scan the per-server breakdown.** Look at the MCP tools table. Notice how
   `computer-use` alone is ~15 tools and `Claude_in_Chrome` ~20. Ask yourself, for
   each server: *have I used this in this session? Will I?*

3. **Apply the test.** Pick one server in that list and ask the Module 5 question:
   *could a script do what it does?* For `scheduled-tasks` — maybe. For
   `computer-use` (driving the live desktop) — no, a script can't. That's the
   instinct to build.

4. **Imagine the savings.** If half those servers are unused this session,
   disconnecting them reclaims a chunk of that 32k from your prefix — and stops
   churning the cache. You don't have to do it now; the point is to *see* the cost
   you were paying invisibly.

---

## 💸 The cost move for this module

> **Treat every connected MCP server as a standing tax and justify it.** Reach for
> **Bash/scripts first** (zero standing cost); add an MCP only when a script
> genuinely can't do the job *and* you'll use it repeatedly. Connect servers
> per-project, prefer narrow tool sets, and audit your active set with `/context`.

---

## Takeaways

- MCP gives Claude **new hands** — tools to reach external systems (browser,
  desktop, databases, APIs).
- Each server's **tool schemas cost context every turn** it's connected (your live
  example: ~32k across all servers), and they sit in the **cached prefix** — a
  churning tool set busts the cache.
- The harness **defers** most schemas (reference-on-demand for tools), which is why
  active cost is far below total — but deferred isn't free.
- **The "just use a script" test:** prefer **Bash** (zero standing cost); justify
  an MCP only when a script can't do it (persistent connection/auth/interactive
  session) *and* you'll use it often.
- **Scope per-project, prefer narrow servers, audit with `/context`,** and treat
  granting an MCP like granting app permissions.

**Next:** Module 6 — Custom Commands, Hooks & Automation: pushing work onto the
*harness* (free) instead of the model (paid).

# Module 5 — MCP Servers

An MCP server gives Claude new hands — tools to reach systems it otherwise can't: a
browser, a database, your desktop, an API. The trade-off is that every connected
server has a standing context cost, because its tool definitions take up room. This
module is about when that cost is worth paying.

---

## What MCP is

MCP (Model Context Protocol) is a standard way to plug external tools into Claude. An
MCP server exposes a set of tools — "navigate to URL", "click element", "run SQL
query" — and Claude Code is the client that calls them.

From Module 4: skill = know-how, subagent = workspace, MCP = new hands. MCP is how
Claude gains capabilities it doesn't have natively: controlling a browser, driving
your desktop, querying a specific service.

---

## The cost: tool schemas live in context

Each tool an MCP server exposes has a schema — name, description, parameters — that
must be described to the model, and those schemas take up context. Your own
`/context` shows it:

```
MCP tools             3.6k     active tool schemas
MCP tools (deferred)  28.7k    schemas held out of the active prompt
```

That is about 32k tokens of tool definitions across your connected servers (Chrome,
computer-use, Preview, scheduled-tasks, and more) — roughly 80 tools. Each is a small
standing cost, and connecting many servers can quietly spend tens of thousands of
tokens before you ask a single question.

Tool schemas also sit at the front of the context, in the stable prefix (Module 0).
Changing your tool set invalidates the cache, so a churning set of MCP servers costs
on both fronts.

---

## The harness's defense: deferred tools

Most of that 32k is deferred, not active. This is the harness applying Module 1's
reference-on-demand principle to tool definitions:

- A small set of tool schemas stay active in the prompt.
- The rest are deferred — their names are known, but the full schema isn't loaded
  until Claude needs that tool, via a tool-search step.

That is why your 80 tools cost 3.6k active instead of 32k. The harness does for tools
what you learned to do for files: don't load everything, reference on demand. But
deferred isn't free — deferred schemas still count toward context, and resolving one
mid-task costs a lookup.

---

## The decision: MCP vs. a script

Knowing when not to use an MCP matters more than knowing how to add one. Before
adding a server, ask whether a one-off Bash command or a small script could do the
same thing.

- A script, or the built-in Bash tool, costs zero standing context — it exists only
  when you run it.
- An MCP server costs context every turn it's connected, whether you use it or not.

An MCP earns its standing cost only when all of these hold:

1. You use it repeatedly, so the per-turn cost amortizes.
2. A script can't easily do it — it needs a persistent connection, managed auth, an
   interactive session (a live browser, your desktop), or structured access a shell
   one-liner can't replicate.
3. The tool surface is reasonably small — a server exposing 60 tools you'll never
   call is mostly dead weight.

| Need | Better choice |
|------|---------------|
| Run `git log`, parse JSON, hit a REST endpoint once | Bash or a script — zero standing cost |
| Drive a real browser through many steps | MCP (Chrome) — a script can't hold the session |
| Query a database you hit constantly all session | MCP — amortizes, manages the connection |
| A capability you'll use once this month | Script, or connect the MCP only when needed |

Reach for Bash first; add an MCP only when a script genuinely can't do the job and
you'll use it often.

---

## Connecting, scoping, and securing

- Scope deliberately. Connect servers per-project where you can, not globally — a
  server you need for one project shouldn't tax every other session. Audit your
  active set periodically with `/context`.
- Prefer narrow servers. A focused tool set costs less context and gives Claude a
  smaller surface to misuse.
- Treat security seriously. An MCP grants real capabilities — desktop control,
  browser control, data access — so treat connecting one like granting an app
  permissions: only connect servers you trust, and be aware that a tool with side
  effects (sending, deleting, paying) can act. Claude Code gates many of these
  behind permission prompts (Module 6).

---

## Hands-on: audit your MCP cost

This exercise is reading a bill you already have, about five minutes.

1. In the `/context` you ran, find the two MCP lines: `MCP tools` (~3.6k active) and
   `MCP tools (deferred)` (~28.7k). That ~32k is the cost of having those servers
   connected, paid before any work.

2. Scan the per-server breakdown. `computer-use` alone is about 15 tools and
   `Claude_in_Chrome` about 20. For each server, ask whether you've used it this
   session and whether you will.

3. Apply the test. Pick one server and ask whether a script could do its job.
   `scheduled-tasks`, maybe. `computer-use`, driving the live desktop, no.

4. If half those servers are unused this session, disconnecting them reclaims a chunk
   of that 32k from your prefix and stops churning the cache. The point is to see the
   cost that was otherwise invisible.

---

## The cost move

Treat every connected MCP server as a standing cost and justify it. Reach for Bash
and scripts first; add an MCP only when a script genuinely can't do the job and
you'll use it repeatedly. Connect servers per-project, prefer narrow tool sets, and
audit your active set with `/context`.

---

## Takeaways

- MCP gives Claude new hands — tools to reach external systems (browser, desktop,
  databases, APIs).
- Each server's tool schemas cost context every turn it's connected (your example:
  ~32k across all servers), and they sit in the cached prefix, so a churning tool set
  busts the cache.
- The harness defers most schemas — reference-on-demand for tools — which is why
  active cost is far below the total, but deferred isn't free.
- Prefer Bash (zero standing cost); justify an MCP only when a script can't do the
  job (persistent connection, auth, interactive session) and you'll use it often.
- Scope per-project, prefer narrow servers, audit with `/context`, and treat
  granting an MCP like granting app permissions.

**Next:** Module 6 — Custom Commands, Hooks & Automation: pushing work onto the
harness instead of the model.

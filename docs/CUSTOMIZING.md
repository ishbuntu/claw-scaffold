# Customization guide: MCPs, models, apps, and everything else

How to plug your own stuff into the assistant without forking anything.
Written for the OpenClaw base (`../openclaw/`); the patterns themselves
are distribution-agnostic.

## Adding your own MCP servers

MCP is the universal socket: any tool you want the agent to have —
smart-home bridge, bank export, custom scraper — ships as an MCP server
and plugs into config.

### The pattern

1. **One config entry per server**, command + args, nothing global:

   ```jsonc
   // OpenClaw: openclaw.json MCP/plugin tool block
   { "my-tool": { "command": "uvx", "args": ["my-mcp-server"] } }
   ```

2. **Probe before trusting**: after adding, verify the server actually
   handshakes and lists its tools before relying on it in a persona or
   cron. A server that fails probe should be disabled, not left flapping.

3. **Scope tools per agent**: main agent gets the full set; subagents
   and cron sessions get deny-lists. A board-sync cron does not need the
   messaging tools. OpenClaw supports per-agent/per-session tool
   policy — use it.

4. **Disable, don't delete**: keep unused servers in config but toggled
   off. Every enabled server costs context (tool descriptions) and RAM
   (a child process); 3–6 enabled is the sweet spot.

### Writing your own MCP server

For anything with an API you want the agent to use safely: wrap it in a
thin MCP server (Python `mcp` package or TypeScript SDK, ~50 lines for
CRUD on one API) instead of letting the agent curl raw endpoints. You
get: typed arguments, a stable tool name the persona can reference,
credential isolation (the server reads the keychain; the agent never
sees the token), and a natural choke point for rate limits and audits.

The board-adapter drivers (`../skills/board-adapter/`) can graduate to
this: driver module → MCP server exposing `list_items`/`update_item`/...
when you want board access from every session without shell access.

### MCP hygiene rules

- Credentials via env/keychain read by the SERVER process — never in
  config JSON, never as tool arguments.
- Third-party MCP servers are code you're running with your data: read
  them before enabling, pin versions, treat updates like dependency
  bumps (pattern: security baseline "update discipline").
- One server = one domain. A do-everything server can't be scoped.

## Custom models

Keep three slots and route by judgment required, not by task name:

| Slot | Used by | Example |
|---|---|---|
| primary | main chat, planning, anything user-facing | top-tier model |
| cheap | subagent fan-out, classification, triage | small/fast model |
| free | zero-token script crons | no model at all |

The adapter exposes primary/cheap in config. Per-cron model override
(pattern 16) is the lever that keeps the bill flat as you add jobs.

## Custom skills

Drop a directory with a SKILL.md (frontmatter: name, description,
triggers, optional always) into the workspace skills dir. Rules that
must never be violated → tiny `always: true` skills (pattern 9).
Everything else → trigger-loaded. Skills are the customization surface
you'll touch most; keep each one small enough to audit at a glance.

## Custom apps / dashboards

When a workflow outgrows chat (you want a screen, not a conversation):

1. **Cheapest**: agent-built local web app — one page served on
   127.0.0.1 reading the board cache / vault. Rebuild on request.
2. **Structured**: OpenClaw plugins + Canvas. Use when the thing needs
   lifecycle (install, update, its own tools and hooks) rather than
   just a view.

Either way the app consumes the same substrate — board cache, vault,
memory — never its own private copy of state.

## Custom automations

Order of preference, cheapest first:

1. Shell command cron (zero tokens) — polling, syncing.
2. Script cron with the notify/skip/done contract (pattern 3).
3. LLM cron from a prompt template (`../crons/`) — judgment on schedule.
4. Heartbeat entry (`../heartbeat/SPEC.md`) — monitor-until-done.
5. Autonomous task runner (pattern 18) — multi-step spec with resume.
6. Webhook → session: external systems (CI, email rules) POST to the
   gateway to wake the agent with context. OpenClaw exposes a webhook/hook
   endpoint; treat inbound payloads as untrusted input.

Pick the lowest number that does the job.

# Obsidian vault as the knowledge base

Architecture for a brand-new vault shared between the always-on agent (Mac
mini) and human editing (laptop/phone). Based on OpenClaw's native Obsidian
support plus community patterns (Aug 2026).

## Decisions

| Question | Decision | Why |
|---|---|---|
| Agent access | Direct filesystem (Filesystem MCP / built-in fs tools) + OpenClaw's official `obsidian` skill (Obsidian CLI 1.12+) | Filesystem works headless even when Obsidian is closed; the CLI skill adds native ops (daily notes, templates, properties) when the app runs |
| Device sync | Obsidian Sync (or Syncthing) | git-backed vaults conflict with an active external writer (obsidian-git doesn't see agent writes; mobile git is unstable). Do NOT use obsidian-git for sync here |
| Search | ripgrep first | Fine below ~5k notes. Graduate to a local semantic layer (qmd/engraph) only when keyword search stops cutting it |
| Provenance | `agent: sisyphus` frontmatter on every machine-written note | Instantly distinguishable, queryable in Dataview, filterable in search |

## Vault structure

Human notes use whatever structure you like (PARA works well). The agent
owns exactly one subtree and never writes outside it:

```
Vault/
├── agent/
│   ├── daily/YYYY-MM-DD.md       # morning rundown + EOD handoff (crons)
│   ├── weekly/YYYY-Wnn.md        # weekly retro
│   ├── lessons/                  # corrections, one note each, #lesson
│   ├── people/                   # notes on people (prefs, context, dates)
│   ├── projects/                 # agent-maintained project state notes
│   ├── career/impact-log.md      # running impact log; weekly-retro cron appends candidates, human curates
│   └── inbox/                    # "remember this" captures from chat
├── ...your PARA/notes...
```

Rules:

- Agent writes only under `agent/`. It may READ the whole vault (that's the
  point of a knowledge base) but human notes are read-only to it.
- Wikilinks from agent notes into human notes are encouraged — that's how
  the two halves knit into one graph.
- Stable filenames, no renames: links must not rot.

## Frontmatter for machine-written notes

```yaml
---
type: daily | lesson | person | project | capture
created: 2026-08-11
agent: sisyphus
tags: []
related: []
---
```

## Write patterns

- **Daily note**: morning-rundown cron creates `agent/daily/{date}.md`;
  EOD handoff appends. One file per day tells the whole story.
- **Lesson capture**: on correction, one small note in `agent/lessons/`
  with the rule, the trigger context, and date. The persona's
  {lesson_tool} maps here when no native memory store is used.
- **Inbox capture**: "remember this" in chat → note in `agent/inbox/` with
  a link to source context. You process the inbox like any Obsidian inbox.
- **Weekly retro**: writes `agent/weekly/{ISO-week}.md`, linking the daily
  notes it summarized.

## Known pitfalls (verified against community reports)

1. Obsidian's file watcher may lag on external writes — the app can hold a
   stale metadata cache until re-scan. Harmless for our append-mostly use.
2. Never have the agent and Obsidian "normalize" the same file
   concurrently: agent writes whole files or appends; it does not fight
   plugins that rewrite frontmatter on save.
3. Keep `.obsidian/workspace*.json` out of any backup/versioning.
4. If the vault syncs to a cloud service, remember the security baseline:
   nothing goes in `agent/` you wouldn't accept on that service; sensitive
   drafts live outside the vault.

## Semantic search, later

When the vault outgrows grep: qmd (github.com/tobi/qmd) or engraph
(hybrid semantic+FTS5+graph) — both local, both expose MCP. Add as one
more MCP server; don't replace filesystem access.

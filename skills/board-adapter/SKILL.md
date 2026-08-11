---
name: board-adapter
description: Query and update connected kanban/work boards (Trello, GitHub Projects, Jira, Linear, Todoist, vault markdown board) through the local cache + driver interface.
triggers: board, kanban, ticket, task, backlog, sprint, my items, whats blocked, move card, close item
---

# Board adapter

## Reading (always from cache — never hit the API for reads)

The sync cron mirrors every connected board to
`{WORKSPACE}/board-cache/{board}.json` every ~5 min. Answer all "what's on
my board / what's blocked / what changed" questions from the cache.
Include the cache timestamp when staleness could matter (>15 min old).

Cache item shape: `id, board, title, state, assignee, priority, due,
last_activity, url, labels`.

## Writing (through the driver, with confirmation tiers)

Drivers live in `{WORKSPACE}/board-drivers/{board}.py` and expose:
`update_item(id, fields)`, `comment(id, text)`, `move(id, state)`,
`create_item(fields)`, `close(id, reason)`.

- Comment, move within active states, create → do it, show what changed.
- Close, bulk operations (3+ items), reassign to another person → list
  exactly what will happen and confirm first.
- After any write, re-sync that item into the cache so views stay honest.

## Adding a new board

1. Copy `board-drivers/_template.py`, implement the five methods against
   the new API.
2. Register a zero-token sync cron for it (see patterns 3/15).
3. Store API credentials via the platform keychain — never in the driver
   file, never in the cache.

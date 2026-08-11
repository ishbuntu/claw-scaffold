# OpenClaw adapter

How this scaffold maps onto OpenClaw (docs.openclaw.ai, verified Aug 2026).
OpenClaw natively supports almost every pattern in `../docs/PATTERNS.md` —
this layer is mostly workspace files + config + registration, no core
patching.

## Concept map

| Scaffold concept | OpenClaw mechanism |
|---|---|
| Sisyphus persona | Workspace `SOUL.md` (persona/tone) + `AGENTS.md` (operating rules) + `IDENTITY.md` (name/emoji) |
| `{spawn_tool}` | `sessions_spawn` (set `subagents.maxSpawnDepth: 2`, `maxConcurrent`, per-agent `allowAgents`) |
| `{lesson_tool}` | `memory_write` (built-in SQLite memory) or a lesson note in the vault (`obsidian/DESIGN.md`) — pick ONE |
| `{notify_tool}` | cron `--announce` delivery / channel message |
| `{wait_tool}` | in-session wait; heartbeat for long waits |
| LLM crons | `openclaw cron add --message "..."` |
| Zero-token script crons | `openclaw automations create --command "..."` or `--script file.js` (headless, no agent turn) |
| Skip semantics | `--pre-check` gate: non-zero exit or empty stdout skips the run |
| Done/Report semantics | script return `{ notify: "...", state: {...} }`; one-shot jobs remove themselves |
| Heartbeat | native: workspace `HEARTBEAT.md` + heartbeat config (OpenClaw's own convention matches `../heartbeat/SPEC.md`) |
| Skills | `<workspace>/skills/<name>/SKILL.md` — same format this repo uses |
| Obsidian | official `obsidian` skill (Obsidian CLI 1.12+) + filesystem access to the vault |
| Board adapter drivers | workspace scripts + `board-adapter` skill; sync via command-payload automations |
| Security baseline [verify] items | `gateway.bind: "loopback"` + token auth, channel `dmPolicy: "pairing"`, `tools.deny`, sandbox `mode: "non-main"`, `openclaw security audit --deep` |

## Applying the layer

```bash
# 1. Base install + onboarding (Anthropic auth, no channels yet)
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon

# 2. Workspace files
WS=~/.openclaw/workspace
mkdir -p "$WS/skills"
#   personas/sisyphus.md  -> split: persona/tone sections into $WS/SOUL.md,
#                            operating rules into $WS/AGENTS.md
#   skills/*              -> $WS/skills/ (fill {PLACEHOLDERS})
#   heartbeat: OpenClaw reads $WS/HEARTBEAT.md natively — start it empty

# 3. Crons (fill {braces} in ../crons/*.md first)
openclaw cron add --name morning-rundown --cron "0 8 * * *" \
  --session isolated --message "$(cat crons/morning-rundown.md)" --announce
openclaw cron add --name midmorning-sweep --cron "0 11 * * MON-FRI" \
  --session isolated --message "$(cat crons/midmorning-sweep.md)" --announce
openclaw cron add --name eod-handoff --cron "30 17 * * MON-FRI" \
  --session isolated --message "$(cat crons/eod-handoff.md)" --announce
openclaw cron add --name aging-digest --cron "0 9 * * MON,WED,FRI" \
  --session isolated --message "$(cat crons/aging-digest.md)" --announce
openclaw cron add --name review-queue-nudge --cron "0 14 * * MON-FRI" \
  --session isolated --message "$(cat crons/review-queue-nudge.md)" --announce
openclaw cron add --name weekly-retro --cron "0 16 * * FRI" \
  --session isolated --message "$(cat crons/weekly-retro.md)" --announce
openclaw cron add --name inbox-triage --every 20m \
  --session isolated --message "$(cat crons/inbox-triage-draft.md)" --announce

# Board sync (zero-token, per connected board)
openclaw automations create "*/5 * * * *" --name board-sync-trello \
  --command "python3 $WS/board-drivers/trello.py sync"

# 4. Harden BEFORE connecting channels
openclaw security audit --deep --fix
```

## Config template

`openclaw.json.example` in this directory: loopback bind, token auth via
env, Anthropic provider, sandboxed non-main sessions, subagent limits,
restrictive tool policy, pairing-mode DMs. Copy to
`~/.openclaw/openclaw.json`, set env vars, run `openclaw doctor`.

## Gotchas found in research

- `agents.defaults.subagents.allowAgents` is silently ignored — set
  `allowAgents` per-agent in `agents.list` (upstream issue #59938).
- Bootstrap files are truncated past ~20k chars each — keep SOUL.md and
  AGENTS.md tight; details belong in skills, loaded on demand.
- Pin the version; update deliberately. Re-run `openclaw security audit
  --deep` after every update.
- Sub-agents inherit a deny-list, not your intent: review
  `tools.subagents.tools` explicitly.

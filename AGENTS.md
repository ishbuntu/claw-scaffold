# AGENTS.md — setup runbook

You are an agent setting up this scaffold on the owner's personal machine
(target: always-on Mac mini). Follow this file top to bottom. Everything
you need is in this repo; do not improvise architecture. Where a step says
INTERVIEW, ask the owner and wait — do not guess.

## Ground rules

- Read `security/BASELINE.md` first. It wins over convenience, every time.
- Personal machine, personal accounts, personal API keys only. If anything
  employer-related appears (credentials, URLs, code), stop and tell the
  owner.
- Verify each phase before starting the next. Evidence = command output.
- The base assistant is OpenClaw — installed, never forked. This repo only
  layers workspace files, skills, crons, and config on top.

## Phase 0 — interview the owner

Collect and write answers to `setup-answers.md` (gitignored is fine):

1. `{OWNER}` name and how the assistant should address them.
2. `{PRIMARY_MODEL}` / `{CHEAP_MODEL}` — Anthropic model IDs to use (and
   whether auth is API key or subscription).
3. Vault path (create if new): default `~/Obsidian/Main`.
4. First chat channel (recommend exactly one to start; Telegram is the
   least-friction) and the owner's account ID on it.
5. Boards to connect (any of: Trello, GitHub Projects, Jira, Linear,
   Todoist, vault markdown board) + where their API creds will come from.
6. Email + calendar: provider (Gmail/Google Workspace, iCloud, IMAP) and
   scope of access — read-only, or read + draft. Recommend read + draft
   (sending stays human). This powers morning-rundown, eod-handoff, and
   inbox-triage. Owner may defer to a later session; note it and move on.
7. `{INBOX}` source for triage (usually the email above; platform DMs
   also work) — or skip the inbox-triage cron for now.
8. Which crons to enable (default: all seven in `crons/`), and whether
   the owner wants any custom summaries beyond them (describe → you
   write the cron template from the existing ones as a pattern).
9. Timezone → adjust cron expressions if not local.

## Phase 1 — base install

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon   # Anthropic auth, NO channels yet
openclaw gateway status              # verify: running
```

Always-on host prep (macOS — sleep kills schedules; a mini that naps
misses crons):

```bash
sudo pmset -a sleep 0 displaysleep 10   # never sleep; display may
pmset -g | grep -E "^\s*(sleep|displaysleep)"   # verify: sleep 0
```

Also confirm with the owner:

- Auto-login enabled for the gateway's macOS user (System Settings →
  Users & Groups → Automatically log in) so launchd + keychain come up
  unattended after a power cut.
- System Settings → Energy: "Start up automatically after a power
  failure" ON, "Prevent automatic sleeping" ON if the pmset route is
  declined.

## Phase 2 — workspace layer

1. `WS=~/.openclaw/workspace`
2. From `personas/sisyphus.md`: persona/tone/identity sections →
   `$WS/SOUL.md` and `$WS/IDENTITY.md`; operating rules (intent gate,
   todos, delegation, verification, corrections, safety rails, scheduled
   -job conduct, long-running work, output) → `$WS/AGENTS.md`.
   Fill every `{braces}` slot using the OpenClaw mapping table in
   `openclaw/README.md` and Phase 0 answers. No unfilled braces may
   remain — grep to prove it: `grep -rn "{[A-Z_]*}" $WS` returns nothing.
3. Copy `skills/*` → `$WS/skills/`, filling placeholders the same way.
4. Create `$WS/HEARTBEAT.md` empty, per `heartbeat/SPEC.md`.
5. Restart the gateway; send a test message via `openclaw dashboard`;
   confirm the agent answers AS Sisyphus (ask it "who are you").

## Phase 3 — Obsidian vault

Follow `obsidian/DESIGN.md`:

1. Create the vault (or open existing) and the `agent/` subtree:
   `daily/ weekly/ lessons/ people/ projects/ inbox/`.
2. If Obsidian app is installed: enable the CLI (Settings → General →
   Command line interface), then install OpenClaw's official obsidian
   skill. If not installed yet: filesystem access alone is fine.
3. Device sync: Obsidian Sync or Syncthing. NOT obsidian-git. Confirm the
   owner's choice.
4. Verify: have the agent create `agent/daily/<today>.md` with correct
   frontmatter (`agent: sisyphus`), and confirm it can READ a human note
   but knows it must not write outside `agent/`.

## Phase 4 — crons

Register the enabled crons exactly as shown in `openclaw/README.md`
(message payloads from `crons/*.md`, placeholders filled). Then:

- `openclaw cron list` — every job present, correct schedule.
- Trigger morning-rundown once manually; verify it delivers AND writes the
  vault daily note.
- Verify inbox-triage cannot send: its session must have outbound message
  tools denied (config), not just the prompt rule.

## Phase 5 — boards (if any)

Per board, following `skills/board-adapter/SKILL.md`:

1. Write the driver (`$WS/board-drivers/<board>.py`) implementing
   list_items/get_item/update_item/comment/move/create_item/close against
   the board's API. Credentials via macOS keychain (`security
   add-generic-password` / `find-generic-password`) — never in files.
2. Register the zero-token sync automation (every 5 min, command payload).
3. Verify: cache file appears, agent answers "what's on my board" from
   cache, a test comment round-trips.

## Phase 6 — harden, then connect channels

Order matters: harden FIRST.

1. Apply `openclaw/openclaw.json.example` values into the live config
   (merge, don't clobber onboarding results). `openclaw doctor` clean.
2. `openclaw security audit --deep --fix` — resolve every finding.
3. Walk `security/BASELINE.md` checkbox by checkbox; report each item's
   status to the owner. [verify] items: check against current OpenClaw
   docs, versions move fast.
4. Only now enable the one channel from Phase 0, `dmPolicy: "pairing"`,
   complete pairing with the owner's account, send the test message.
5. Email + calendar (if chosen in Phase 0) — LAST, after everything
   above is green:
   - Gmail/Google Workspace: install the gog skill (Gmail + Calendar
     CLI); walk the owner through its OAuth consent, requesting the
     narrowest scopes matching Phase 0 (read-only vs read + draft —
     never grant send unless the owner explicitly insists).
   - iCloud/IMAP: an IMAP-based skill or Apple Mail via AppleScript on
     this host.
   - Verify: agent reads today's calendar and latest 5 subjects; then
     prove the negative — inbox-triage drafts but CANNOT send (tool
     denial in config, not just the prompt rule).

## Phase 7 — acceptance

Demonstrate to the owner, live:

- [ ] "who are you" → Sisyphus persona
- [ ] morning rundown arrives on schedule (or manual trigger) + vault note
  — including calendar + email sections if Phase 6.5 was done
- [ ] a correction ("always call me X") persists into a new session
- [ ] "watch <thing> until done" → heartbeat entry appears; completes and
      self-removes when the condition is met
- [ ] board query answered from cache in <5s
- [ ] inbox triage produces a draft and does NOT send
- [ ] `openclaw security audit --deep` clean
- [ ] gateway unreachable from another device except via Tailscale (if
      configured)

Then write a summary of what was configured (versions, channels, boards,
crons, deviations from this runbook) to `setup-report.md` and give it to
the owner.

## When stuck

- Config semantics: docs.openclaw.ai/reference/configuration and the
  gotchas list at the bottom of `openclaw/README.md`.
- 3 failed attempts at any step → stop, report exactly what was tried,
  ask the owner. Do not leave the gateway in a half-configured state:
  revert to the last working config.

# Pattern library

Every reusable idea from running a persistent orchestrating assistant,
distilled to its portable form. Each pattern lists what it is, why it earns
its place, and how to implement it on any claw-style distribution.

---

## 1. Orchestrator persona with an intent gate

A single named agent (Sisyphus) fronts everything. Before acting on any
message it classifies intent: trivial, explicit, exploratory, open-ended, or
ambiguous. Questions get answers, not implementations. Ambiguity with a 2x+
effort difference between readings gets ONE clarifying question. Everything
else proceeds without ceremony.

Why: a persistent assistant receives wildly mixed input (questions, tasks,
vents, forwarded messages). The gate stops the two failure modes that erode
trust: implementing when you were only asked a question, and asking
permission for trivia.

Implement: system prompt section. See `personas/sisyphus.md`.

## 2. Delegation-first with completion events

The orchestrator never does parallelizable work inline. Independent subtasks
spawn as parallel subagents in one call; the orchestrator's own turn is
reserved for synthesis, decisions, and verification. Results arrive as
injected completion events — no polling. Hard anti-pattern: starting the
delegated work yourself after spawning.

Why: a chat-facing agent that grinds through serial work feels dead. Spawn,
acknowledge, synthesize when events land.

Implement: whatever subagent mechanism the distribution has (OpenClaw
`sessions_spawn`, sub-sessions, task tools). The prompt rules are the
portable part.

## 3. Zero-token script crons

Two tiers of scheduled jobs:

- **LLM crons**: a prompt runs as an agent turn on schedule. For anything
  needing judgment (summaries, triage, drafting).
- **Script crons**: a deterministic script (Python/shell) runs directly, no
  model call, zero tokens. For polling, syncing, and condition checks where
  reasoning adds nothing. The script signals outcomes through a tiny
  contract: notify(message), Skip (silent, retry next cycle), Done(message)
  (deliver and remove job), Report(message) (deliver and keep job).

Why: an always-on assistant that polls every 5 minutes with LLM turns burns
money doing regex work. Push determinism down to scripts; reserve the model
for judgment. The Skip/Done/Report contract keeps scripts composable with
the notification system.

Implement: OpenClaw cron supports non-agent command execution; wrap the
Skip/Done/Report semantics in a small runner. Scripts live in the repo,
registered by a bootstrap script.

## 4. The daily rhythm cron suite

A fixed cadence of scheduled agent jobs that turn the assistant into a chief
of staff:

| Job | Schedule | Purpose |
|---|---|---|
| Morning rundown | 08:00 daily | Everything on my plate today, ranked |
| Midmorning sweep | 11:00 weekdays | New urgent items since the rundown |
| End-of-day handoff | 17:30 weekdays | What I did, what's open, what tomorrow-me needs |
| Aging digest | Mon/Wed/Fri 09:00 | Items I own with no activity for N days |
| Review-queue nudge | 14:00 weekdays | Things waiting on ME >24h (reviews, replies, decisions) |
| Weekly retro | Fri 16:00 | Everything I touched this week, grouped, with patterns |

Why: pull-based assistants only help when you remember to ask. The rhythm
makes the assistant push the right context at the right time. The
aging digest and review-queue nudge specifically target the silent killers:
things you own that stall, and things blocked on you.

Implement: cron prompt templates in `crons/`, parameterized by data source
(calendar, task manager, email, ticket system).

## 5. Draft-never-send triage

A frequent cron (every ~20 min) reads new inbound messages and drafts
replies, but is hard-forbidden from sending. Drafts land in a review
location; the human sends.

Why: automated outbound is the highest-blast-radius thing a personal agent
can do. Drafting captures 90% of the value at 0% of the risk.

Implement: cron prompt with an explicit MUST NOT SEND rule, plus channel/tool
permissions that deny outbound message tools to that job (defense in depth:
prompt rule AND permission rule).

## 6. Heartbeat: monitor-until-done task queue

A self-cleaning checklist file the agent processes every few minutes.
Each entry is a task with a completion condition. On each cycle the agent
processes entries; a task is retained only if the response contains an
explicit KEEP marker (or the task errored). Complete tasks disappear without
manual cleanup. Survives gateway restarts because state is a plain file.

Why: "keep an eye on it" is the most common request an assistant fumbles.
Sessions end; the heartbeat file persists. The KEEP-marker retention default
(forget unless explicitly kept) means the queue can't silently grow stale.

Implement: `heartbeat/SPEC.md`. Needs only a recurring cron + a markdown file.

## 7. Wait-and-poll vs heartbeat split

Short waits (user present, <30 min): stay in-session, use a wait tool, poll,
iterate with a hard cap (stop after 3 non-improving iterations or 5 total).
Long or unattended waits: write a heartbeat entry and end the session.

Why: the iteration cap prevents the infinite fix-check-fix loop; the split
keeps sessions short-lived and state durable.

## 8. Lessons loop (correct once, never again)

When corrected, the agent saves the lesson immediately — not at session end.
Save only behavior-changing corrections; never one-time facts, changelog
noise, or things already covered by standing docs.

Why: requiring the same correction twice is the fastest way to make an
assistant feel worthless. The save-filter matters as much as the saving —
a lessons store full of trivia is dead weight.

Implement: distribution memory tool if present; otherwise Obsidian notes
(see `obsidian/DESIGN.md`) with a `lesson` tag.

## 9. Always-on micro-skills for hard rules

Tiny skills marked always-loaded that encode routing rules the agent must
never get wrong: resolve identities fresh before messaging anyone (no
cached aliases), which tool handles outbound messages, mandatory prefixes
or formats. Kept separate from the persona so they're auditable one rule
per file.

Why: system prompts drift and get compacted; a 20-line always-on skill with
ONE rule survives. Identity resolution specifically prevents the worst
personal-assistant failure: messaging the wrong human.

## 10. Evidence-based verification

A task is complete only with evidence: a diff proves an edit, exit 0 proves
a build, command output proves a claim. Three consecutive failed fixes →
stop, revert, report what was tried. Fix root causes; never refactor while
bugfixing.

## 11. Safety confirmations at the tool boundary

Standing rules independent of any one task:

- Destructive commands (recursive delete, table drops, instance
  termination): confirm with the human first.
- Reading credential files (SSH keys, cloud creds): confirm first.
- Any local HTTP server: bind 127.0.0.1 explicitly, never a default bind.
- Outbound messages from scheduled jobs: draft-only unless a job is
  explicitly whitelisted to send.

Why: a persistent agent with shell access WILL eventually be asked (or
decide) to do something irreversible. Cheap friction at the four
highest-blast-radius boundaries.

## 12. Session-restore context with staleness marking

When a webhook or restart resumes a prior workflow, restored context is
injected with an explicit staleness warning when old. The agent treats
stale context as low-confidence and verifies before acting on it.

Why: acting confidently on a week-old snapshot is worse than starting fresh.

## 13. Transient-failure retry discipline

Tool disconnect followed by reconnect within the same turn = transient.
Retry silently; only report unavailability after 2+ failed retries.

Why: always-on gateways see constant MCP churn; an agent that announces
every blip trains the human to ignore it.

## 14. Diff-visible file changes

Every file mutation is shown as a unified diff with real paths. Makes the
agent's writes auditable at a glance from any chat surface, and doubles as
the evidence artifact for pattern 10.

## 15. Board adapter: connect to any kanban, work it, update it

A generic work-board layer with three parts:

- **Sync (script cron, zero tokens)**: a deterministic script polls the
  board's API on a short interval (~5 min) and mirrors items to a local
  cache (JSON or markdown). Sync never calls the model.
- **View**: the agent answers "what's on my board / what's blocked / what
  changed today" from the local cache instantly — no API round-trips, no
  rate limits, works even when the board is slow.
- **Act**: moving cards, commenting, assigning, closing go through a small
  set of write tools against the board API. Writes above a blast-radius
  threshold (closing items, bulk moves) confirm first per pattern 11.

The adapter is an interface, not a product integration: `list_items`,
`get_item`, `update_item`, `comment`, `move`. Each backend (Trello, GitHub
Projects, Jira, Linear, Todoist, a plain markdown board in the Obsidian
vault) is a thin driver behind it. The cron suite (pattern 4) reads the
same cache: the morning rundown, aging digest, and review-queue nudge all
become board queries plus judgment.

Why: every job and life system eventually resolves to "a board with items
in states". Building the sync/view/act split once means switching employers,
tools, or projects only means writing a new driver — the agent workflows,
crons, and digests stay identical. Local cache plus zero-token sync is what
makes a 5-minute poll affordable.

Implement: one script-cron per board (pattern 3), cache under the
workspace, drivers as small Python modules, board queries exposed to the
agent as a skill.

## 16. Cron hygiene

Scheduled jobs at scale need four controls beyond the schedule itself:

- **Jitter**: recurring jobs get a random delay (minutes for hourly, up
  to a couple hours for daily) unless marked strict — prevents every job
  slamming the machine and APIs at the same instant.
- **Skip dates**: recurring jobs can exclude specific dates (holidays,
  vacation) with a per-job timezone.
- **Per-job timeout**: a runaway job is killed and reported, never left
  spinning.
- **Per-job model override**: judgment-light jobs run on the cheap model;
  only jobs needing real reasoning get the expensive one.

Why: the daily-rhythm suite (pattern 4) is 7+ jobs before you add your
own. Without jitter and timeouts, one bad morning takes the whole rhythm
down with it.

## 17. Artifacts: versioned outputs that outlive chat

Anything substantial the agent produces (a document, a report, a script)
becomes an artifact: stable identity, deliberate version snapshots (not
every keystroke), revert, and an activity timeline (created / edited /
iterated / referenced). Artifacts are file-backed — the artifact points at
a real path on disk, so external edits and agent edits converge on one
source of truth. Iterating happens by talking to the agent with the
artifact in context.

Why: chat scrollback is where good outputs go to die. The vault (see
`../obsidian/DESIGN.md`) covers notes; artifacts cover working documents
that go through drafts. In this scaffold the two merge: artifacts live as
files under the vault's `agent/` subtree with version history via
snapshot copies or the distribution's artifact store when it has one.

## 18. Autonomous task runner: spec → steps → verify → resume

For multi-step delegated work ("do this migration", "build this app"):

1. Decompose the spec into ordered steps with explicit completion checks.
2. Execute step by step, writing progress to a durable file after each.
3. Failed step → diagnose, retry with the fix; extract a lesson from the
   failure (pattern 8) so the next task doesn't repeat it.
4. Crash or restart → resume from the progress file, not from zero.
5. After all steps pass, an acceptance pass reviews the RESULT against
   the ORIGINAL spec — steps passing is not the same as the spec being
   satisfied.
6. A watchdog alerts on stalls (no progress in ~30 min) and enforces a
   global timeout.

For bigger jobs, gate stages on human approval: present the plan, let the
owner approve stage-by-stage or all at once, stop after repeated
escalations rather than thrashing.

Why: this is the difference between an assistant that chats about work
and one you hand a spec at 11pm and review at breakfast. The durable
progress file is the load-bearing piece — everything else degrades
gracefully if that survives.

## 19. Layered memory with decay

Memory is not one store. The layers that earn their keep:

| Layer | Content | Lifecycle |
|---|---|---|
| Preferences | how the owner likes things | rewritten wholesale periodically |
| Projects | current state per project/area | rewritten wholesale periodically |
| History | what happened, by day | decays: full → summary → date-only → pruned |
| Lessons | corrections (pattern 8) | permanent until contradicted |
| Episodic/semantic search | everything, embedded | searched on demand, never injected wholesale |

Two rules make it work: **confidence gating** (the agent only writes a
"fact" to memory when confident; user-stated facts always qualify) and
**budgeted injection** (always-injected layers have a hard char budget;
the rest is reachable only by search). Time-decay keeps history honest —
last week in full, last year as dates.

Why: inject-everything assistants drown in their own past; search-only
assistants forget your name. The split — small curated always-on core,
large searchable archive — is the only shape that scales.

Implement: distributions ship most of this (memory files + embeddings).
The vault mirrors the human-readable layers (`agent/lessons/`, daily
notes as history). Let the distribution own embeddings.

## 20. Inbox secretary

A step past draft-never-send triage (pattern 5): the inbox itself becomes
a managed surface.

- **Discovery**: the agent enumerates conversations (DMs, group threads)
  itself — no manual list to maintain.
- **Classification**: each new human message → needs-reply / needs-action
  / FYI / noise. Bot noise auto-classified without an LLM call.
- **Drafts on demand**: replies drafted when you ask (button/command),
  not speculatively for everything — cheaper and less presumptuous.
- **Investigate**: any inbox item can be handed to a full session with
  the thread context attached.
- **Keyword hooks**: configurable rules — "message contains X → notify
  me immediately / spawn a session / draft a reply". The only path to an
  auto-sent reply is an explicit per-hook whitelist (pattern 11 applies).

Why: triage crons (pattern 5) poll on a schedule; a secretary owns the
queue between polls — nothing sits unclassified, and urgent keywords
break through the batch cycle.

## 21. Snapshot and restore

One command exports the assistant's whole state — config, memory,
lessons, crons, skills, vault agent-subtree — to a portable archive;
one command restores it, with replace and merge modes and a dry-run
preview. Merge dedups (memory by content, crons by name).

Why: the always-on machine WILL be rebuilt someday. If state restore
isn't tested, the "self-learning" half of the assistant is a single
hardware failure away from amnesia. Restore-test quarterly: snapshot,
restore into a scratch home dir, boot, ask it something it should know.

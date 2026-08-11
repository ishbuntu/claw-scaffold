# Heartbeat: monitor-until-done task queue

A self-cleaning checklist the agent processes on a fixed cycle. Solves
"keep an eye on this" durably: sessions end, the file persists, the cycle
continues after restarts.

## State

One markdown file: `{WORKSPACE}/HEARTBEAT.md`. Each task is a checklist
line:

```markdown
- [ ] Check PR #142 for new review comments. If found, address them and
  push; respond HEARTBEAT_KEEP. If merged, notify "PR #142 merged ✅".
- [ ] Passport renewal: check status page with ref ABC123. Not approved →
  HEARTBEAT_KEEP. Approved → notify me and add a vault note.
```

A good entry contains: what to check, what to do on each outcome, and the
completion condition. Entries the agent can't act on autonomously should
say "notify owner" as the action.

## Cycle

A recurring cron (every 5–15 min) runs an agent turn:

1. Read `HEARTBEAT.md`. Empty → end immediately (zero further cost).
2. Process each unchecked task independently.
3. Retention contract, per task:
   - Response contains `HEARTBEAT_KEEP` → task is retained for next cycle.
   - Response omits it → task is complete; remove the line.
   - Task raised an error → retained automatically (transient failures
     must not silently drop a monitor).
4. Deliver any notifications produced.

Default-forget is the point: the queue cannot accumulate stale entries,
because keeping a task requires an explicit, per-cycle decision.

## Writing entries (agent-side rules)

- One condition per entry. "Watch the deploy AND the ticket" is two lines.
- Include enough context to act from a cold start — the processing session
  has no memory of the conversation that created the entry.
- Add a `(since YYYY-MM-DD)` stamp; entries older than {MAX_AGE, default
  14 days} get surfaced to the owner as possibly-stale instead of blindly
  re-processed.

## When to use heartbeat vs in-session wait

| Situation | Mechanism |
|---|---|
| Owner is waiting, expected <30 min | in-session wait + poll (cap: 3 non-improving iterations or 5 total) |
| Longer, unattended, or "let me know when" | heartbeat entry, end session |

## Cost note

Each cycle with N tasks is one agent turn reading a small file. Checks
that are pure polling (HTTP status, string match) should graduate to
zero-token script crons (pattern 3) — heartbeat is for checks needing
judgment on what to do next.

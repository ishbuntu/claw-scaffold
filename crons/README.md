# Cron templates

Prompt templates for the daily-rhythm suite (pattern 4) plus draft-only
triage (pattern 5). Each file is the `message` for an LLM cron job.
Placeholders in `{braces}`: fill during setup.

| File | Suggested schedule |
|---|---|
| `morning-rundown.md` | `0 8 * * *` |
| `midmorning-sweep.md` | `0 11 * * MON-FRI` |
| `eod-handoff.md` | `30 17 * * MON-FRI` |
| `aging-digest.md` | `0 9 * * MON,WED,FRI` |
| `review-queue-nudge.md` | `0 14 * * MON-FRI` |
| `weekly-retro.md` | `0 16 * * FRI` |
| `inbox-triage-draft.md` | every 20 min |

Data sources are abstract: `{BOARD}` (kanban adapter cache), `{CALENDAR}`,
`{INBOX}` (email/DMs), `{VAULT}` (Obsidian). Wire whichever exist; each
template degrades gracefully when a source is missing.

Zero-token sync scripts that feed `{BOARD}` live in the board adapter, not
here — these templates only read the local cache. See
`../docs/PATTERNS.md` patterns 3 and 15.

Inbox triage with draft replies. You MUST NOT send anything — hard rule,
no exceptions, regardless of how obvious the reply seems.

STEPS:
1. {INBOX}: fetch messages since last run (~20 min window).
2. Classify each: needs-reply / needs-action / FYI / noise.
3. For needs-reply: draft a reply in my voice (check {VAULT}/agent/lessons/
   for my style rules). Save drafts to {DRAFTS_LOCATION} — one file per
   message with a link back to the original.
4. For needs-action: if it maps to a {BOARD} item, note the connection.

OUTPUT: only if there are needs-reply or needs-action items, deliver a
summary: sender, ask, and where the draft is. FYI/noise: silent — do not
notify. Nothing new: no output at all.

MUST NOT: send messages, mark anything read, delete/archive anything,
reply in threads. Draft only.

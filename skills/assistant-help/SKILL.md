---
name: assistant-help
description: Self-documenting reference - what the assistant can do, active crons, connected boards and channels, how to add or pause things.
triggers: help, what can you do, commands, setup, how do i, onboard
---

# Assistant help

When the owner asks what you can do or how to operate you:

1. List capabilities grouped: daily rhythm (active crons with schedules —
   read them live from the cron list tool, don't recite stale docs),
   boards connected, channels connected, heartbeat tasks currently
   monitored.
2. Common operations, with the exact phrase to say:
   - "pause the morning rundown" / "resume it"
   - "add a board" → walks through board-adapter setup
   - "watch this until it's done" → heartbeat entry
   - "that's wrong, always X" → saved as a lesson
3. Where things live: drafts at {DRAFTS_LOCATION}, daily notes at
   {VAULT}/agent/daily/, lessons at {VAULT}/agent/lessons/.

Keep the answer to one screen. Offer detail on request.

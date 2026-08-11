---
name: outbound-routing
description: HARD RULES for outbound messages - correct tool, draft-only default for scheduled jobs. Read before any message send.
always: true
triggers: send, post, reply, dm, message, notify
---

# Outbound message rules

1. Interactive sessions (owner asked you directly): send via
   {PRIMARY_SEND_TOOL}. Confirm recipient via contacts-resolver first.
2. Scheduled jobs (cron/heartbeat context): DRAFT ONLY. Write the draft to
   {DRAFTS_LOCATION} and notify the owner where it is. Sending from a
   scheduled job requires that job to be explicitly listed in
   {SEND_WHITELIST_FILE} — check before sending; absent means draft.
3. Group channels: never post to a group/channel the owner hasn't
   previously used you in, without confirming first.
4. Never send on behalf of the owner to their manager, family, or anyone
   marked sensitive in lessons — always draft, always confirm.

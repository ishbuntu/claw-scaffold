---
name: contacts-resolver
description: Resolve any person identifier (name, alias, email, handle) via a fresh directory lookup before messaging. Never trust cached mappings.
always: true
triggers: message, dm, send to, reply to, tell, remind
---

# Identity resolution

Before sending any message to a person identified by anything other than a
raw platform ID:

1. Call the platform's user-lookup tool with the identifier as given.
2. Exactly one match → proceed with that ID.
3. Zero or multiple matches → ask the owner to disambiguate. Never guess.

No caching. People change handles; directories change. A stale mapping
means messaging the wrong human — the worst failure a personal assistant
can make. Resolve fresh every time.

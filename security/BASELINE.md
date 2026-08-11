# Security baseline — single-operator, always-on assistant

An always-on agent with shell access, messaging reach, and your credentials
is the most dangerous software in your house. Apply this before connecting
any channel. Items marked [verify] have distribution-specific mechanics —
confirm against your distribution's current security docs.

## Identity and access

- [ ] Dedicated OS user for the gateway; your personal account stays
      separate. The agent's user owns only its workspace, vault agent
      folders, and cache.
- [ ] Gateway binds loopback only. Remote access via Tailscale (or
      WireGuard) — never a port-forward, never 0.0.0.0. [verify]
- [ ] Gateway auth token set even on loopback (defense against local
      SSRF/browser attacks). [verify]
- [ ] Channel allowlist: exactly your own account IDs may talk to the
      agent. Group chats: mention-only, allowlisted groups. [verify]
- [ ] Treat inbound messages from anyone else as untrusted input — pairing
      / DM approval on. [verify]

## Secrets

- [ ] API keys in the OS keychain or an env file readable only by the
      gateway user (`chmod 600`) — never in config committed to git, never
      in the vault, never in board caches.
- [ ] The scaffold repo carries `{PLACEHOLDERS}` only. `.gitignore` covers
      `*.env`, `*token*`, `*secret*`, `board-cache/`.
- [ ] Agent rule (already in persona): reading credential files requires
      confirmation. Enforce it also at tool-permission level where the
      distribution supports deny-lists. [verify]

## Blast-radius controls

- [ ] Destructive shell patterns (recursive force deletes, disk writes,
      package publishes) on the confirm-first list.
- [ ] Outbound messaging: draft-only default for all scheduled jobs;
      explicit whitelist file for jobs allowed to send (see
      `skills/outbound-routing`).
- [ ] Board writes: close/bulk/reassign confirm first (see
      `skills/board-adapter`).
- [ ] Sandbox non-main sessions (Docker or the distribution's sandboxing)
      so spawned subagents can't touch the host by default. [verify]
- [ ] Financial, medical, legal actions: never autonomous. Draft and
      confirm.

## Data hygiene

- [ ] Vault sync: if the vault syncs to a cloud service, the `agent/`
      subtree contains only what you'd accept leaking to that service.
      Drafts of sensitive replies go to a local-only drafts dir, not the
      synced vault.
- [ ] Logs: gateway and session logs rotate; they contain message content.
      Same protection level as the vault.
- [ ] Backups: workspace + vault in your normal backup; secrets excluded.

## Update discipline

- [ ] Pin the distribution version; update deliberately (read release
      notes), not automatically. An always-on agent that self-updates is a
      supply-chain risk with hands.
- [ ] Third-party skills: read every line before installing. Skills are
      prompt-injection with filesystem access. Prefer authoring your own
      (this repo) over installing from registries.

## Employer boundary

- [ ] This system runs on personal hardware, personal accounts, personal
      API keys. No employer credentials, code, internal URLs, or data ever
      enter the workspace, vault, boards, or prompts. If a work task needs
      doing, do it on work systems.

# claw-scaffold

A portable pattern library and configuration layer for running a personal,
always-on AI assistant ("claw"-style: OpenClaw or any compatible agent
gateway) with a Sisyphus orchestrator persona, life/job automation crons,
heartbeat monitoring, and an Obsidian vault as the long-term knowledge base.

This repo contains **ideas, prompts, templates, and configuration** — not a
fork of any assistant. You install the base assistant (e.g. OpenClaw)
separately and apply this layer on top.

## Provenance and content policy

Every file in this repo was written fresh for personal use. It contains:

- No employer-internal code, prompts, tool names, service names, or data.
- Patterns expressed generically (ticket system, code review, chat platform)
  so they apply to any personal or professional context.
- A Sisyphus persona derived conceptually from the public OhMyOpenCode
  project's orchestrator philosophy, rewritten from scratch.

Do not add employer-internal content to this repo. Ever.

## Layout

| Path | What it is |
|---|---|
| `docs/PATTERNS.md` | The distilled pattern library: every reusable idea, with rationale |
| `personas/sisyphus.md` | Sisyphus orchestrator system prompt (tool-agnostic, with adapter notes) |
| `crons/` | Scheduled-job prompt templates for daily life/job management |
| `skills/` | Portable skill templates (SKILL.md format) |
| `heartbeat/SPEC.md` | Monitor-until-done task queue spec |
| `security/BASELINE.md` | Single-operator hardening checklist |
| `obsidian/DESIGN.md` | Obsidian vault as agent memory: architecture and conventions |
| `openclaw/` | OpenClaw-specific adapter: config template + Mac mini bootstrap |

## Quick start (Mac mini)

1. Install the base assistant: `curl -fsSL https://openclaw.ai/install.sh | bash`
2. Run onboarding with your Anthropic auth: `openclaw onboard --install-daemon`
3. Apply this layer: see `openclaw/README.md`
4. Read `security/BASELINE.md` **before** connecting any messaging channel.

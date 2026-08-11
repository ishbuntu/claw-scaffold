# Skills

Portable skill templates. Format follows the common SKILL.md convention
(frontmatter: name, description, triggers; body: instructions). `always:
true` skills encode hard rules that must survive context compaction — one
rule per skill, kept tiny (pattern 9).

| Skill | Type | Purpose |
|---|---|---|
| `contacts-resolver/` | always-on | Resolve identity fresh before messaging anyone |
| `outbound-routing/` | always-on | Which tool sends messages + draft-only default for crons |
| `board-adapter/` | triggered | Query/update any connected kanban board via the local cache |
| `image-authoring/` | triggered | Author images as code (SVG/matplotlib/PIL) when no image model exists |
| `assistant-help/` | triggered | Self-documenting command reference for the owner |

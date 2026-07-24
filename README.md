# GuidedTrack AI Coding Agent Skill

A skill that teaches AI coding agents (Claude Code, Codex, and similar) to write, edit, review, push, and debug [GuidedTrack](https://www.guidedtrack.com/) programs correctly.

GuidedTrack is a strict DSL with many undocumented sharp edges — collection methods that mutate in place, HTML sanitization rules, a call-stack bug in long loops, shared variable scope across subprograms, and a push CLI with several traps. This skill encodes all of that so agents get it right the first time instead of rediscovering each pitfall.

## Contents

| Path | Purpose |
|---|---|
| `SKILL.md` | The skill entry point: workflow, validation checklist, authoring rules, pushing to GuidedTrack.com (Mac/Linux/Windows), downloading run data |
| `references/complete_guide.md` | The GuidedTrack language guide: syntax rules, core patterns, keyword inventory, language notes, common misconceptions |
| `bin/gt` | The GuidedTrack CLI (bash). This copy includes portable fixes (works with Windows jq.exe; program names with spaces) and adds a `pull` subcommand for downloading program source |
| `agents/openai.yaml` | Metadata for Codex-style agent frameworks |

## Installation

**Claude Code (per-project):** copy this repo's contents into `<project>/skills/skill_guidedtrack-builder/` (or clone it there).

**Claude Code (global):** clone into `~/.claude/skills/guidedtrack-builder/`.

The `bin/gt` CLI is optional but recommended — install it with:

```bash
mkdir -p ~/bin && cp bin/gt ~/bin/gt && chmod +x ~/bin/gt
```

`gt` requires `jq` (installation instructions for each OS are in SKILL.md).

## Provenance

Merged 2026-07-24 from the original skill, study-recreation additions (Isaac), a Windows adaptation (Greg), and field-tested improvements from building and launching a production research study with ~330 stimuli across seven GuidedTrack programs. Every non-obvious claim in the guide was either verified against the [official documentation](https://docs.guidedtrack.com/) or observed empirically in production.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

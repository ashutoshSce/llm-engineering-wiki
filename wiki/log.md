# Wiki Log

Append-only activity log. Every ingest, query, and lint pass is recorded here.

Format: `## [YYYY-MM-DD] <operation> | <title>`

Greppable: `grep "^## \[" wiki/log.md | tail -10`

---

## [YYYY-MM-DD] create | Wiki initialized

Wiki structure scaffolded.

Files created:
- `CLAUDE.md` — schema and operating manual
- `wiki/index.md` — master catalog
- `wiki/log.md` — this file
- `wiki/overview.md` — architecture synthesis
- `wiki/glossary.md` — terminology reference
- `wiki/sources/` — per-document summaries
- `wiki/analyses/` — saved query answers
- `raw/` — source documents (immutable)
- `.obsidian/` — pre-configured Obsidian vault

---

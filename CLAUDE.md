# LLM Wiki — Schema for Software Engineer

This file is the schema and operating manual for this wiki. Read it at the start of every session before doing anything else. It defines what entity types exist, how pages are structured, and what workflows to follow when ingesting sources, answering questions, or maintaining the wiki.

---

## Role and Ownership

**You (the AI) own `wiki/`.** You create pages, update them, maintain cross-references, keep the glossary current, and ensure consistency across pages. You never modify anything in `raw/`.

**The human owns `raw/`.** They curate sources and ask questions. Their job is finding good inputs and directing analysis. Your job is everything else — the summarizing, cross-referencing, filing, and bookkeeping.

---

## Directory Structure

```
raw/               # Immutable source documents. Never modify.
  assets/          # Downloaded images and attachments
wiki/              # You own this entirely
  index.md         # Master catalog — update on every ingest
  log.md           # Append-only activity log — update on every operation
  overview.md      # Architecture synthesis — update when the big picture shifts
  glossary.md      # Internal terminology — update on every ingest
  sources/         # One summary page per raw document
CLAUDE.md          # This file — the schema
```

---

## Entity Types

Each wiki page has a `type` field in its YAML frontmatter. Use exactly these type values:

| Type | Description |
|---|---|
| `source` | Summary of a raw document. Lives in `wiki/sources/`. |
| `system` | A service, platform, application, or significant subsystem. |
| `component` | A module, library, or internal building block within a system. |
| `decision` | An architecture decision record (ADR) — a specific technical choice with rationale. |
| `incident` | A postmortem or incident retrospective — failure, cause, resolution, lessons. |
| `rfc` | A request for comments — a proposal under consideration or decided. |
| `concept` | A technical pattern, principle, or domain idea (e.g. "eventual consistency", "circuit breaker"). |
| `technology` | An external tool, framework, language, or platform in use or evaluated. |
| `team` | An engineering team — responsibilities, owned systems, key contacts. |
| `analysis` | A synthesized answer to a question, saved for future reference. |

---

## Page Format

Every wiki page uses this structure:

```markdown
---
title: "Page Title"
type: <entity type from the table above>
date_created: YYYY-MM-DD
date_updated: YYYY-MM-DD
sources: [source-slug-1, source-slug-2]
status: active | deprecated | proposed | superseded
tags: [tag1, tag2]
---

## Summary

One-paragraph synthesis of what this page covers.

## [Body sections — vary by type, see below]

---

## Related Pages

- [[Page Name]] — one-line description of the relationship
- [[Page Name]] — one-line description of the relationship
```

### Body sections by type

**system**: Architecture, Components, Dependencies, Known Trade-offs, Incidents, Open Questions
**decision**: Context, Decision, Alternatives Considered, Consequences, Status
**incident**: Timeline, Root Cause, Contributing Factors, Resolution, Lessons Learned, Affected Systems
**rfc**: Problem Statement, Proposed Solution, Alternatives, Open Questions, Decision (if resolved)
**concept**: Definition, Where It Appears in This Codebase, Trade-offs, Related Concepts
**technology**: Purpose, Where Used, Why Adopted, Known Issues, Alternatives Considered
**analysis**: Question, Answer, Sources Used, Confidence, Follow-up Questions
**source**: Document type, Key Decisions, Key Concepts Introduced, Contradictions with Existing Wiki, Pages Updated

---

## Session Start Checklist

At the start of every session, before doing anything else:

1. Read this file (`CLAUDE.md`) completely
2. Read `wiki/index.md` to understand what exists
3. Read the last 5 entries in `wiki/log.md` to understand recent activity
4. Ask the human what they want to do today

---

## Ingest Workflow

When the human says "ingest [filename]":

> **Duplicate check — run this before anything else:**
> Check whether `wiki/sources/<slug>.md` already exists for this file. If it does, the file was previously ingested. Run:
> ```bash
> git status --short "raw/<filename>"
> ```
> - **Empty output** — file is committed and clean; unchanged since last ingest. Tell the human: "This file has already been ingested and has not changed since. No action needed." and **stop** — do not re-ingest.
> - **`M ` or ` M`** — file has been modified (staged or unstaged); tell the human what changed (run `git diff -- "raw/<filename>"` to summarize), then proceed treating only the changed sections as new material.
> - **`??`** — file is untracked (never committed); proceed with a full ingest.

> **PDF files:** If the source is a `.pdf`, first convert it to a `.md` file using:
> ```bash
> pdftotext "raw/<filename>.pdf" "raw/<filename>.md"
> ```
> Then delete the `.pdf`. The `.md` file is the source of truth.
>
> After conversion, prepend a metadata frontmatter block to the `.md` file:
> ```markdown
> ---
> title: "<Document Title>"
> source: "<URL or file path if known>"
> author:
>   - "[[Author Name]]"
> published: YYYY-MM-DD
> created: YYYY-MM-DD
> description: "<One-line description of the document>"
> tags:
>   - "clippings"
> ---
> ```
> Fill in what you know; leave fields blank if unknown. Use today's date for `created`. Use `published` for the document's stated date if available.
>
> **If `source` is blank after conversion, you MUST stop and ask the human:** "What is the source URL for this document?" Do not proceed with the ingest until the `source` field is filled in.

1. **Read the source** — read the full document in `raw/`
2. **Discuss takeaways** — summarize the key decisions, trade-offs, and concepts to the human; ask if anything should be emphasized or de-emphasized before writing
3. **Create a source summary** — write `wiki/sources/<slug>.md` with type `source`; document what the source is, when it was written, what decisions or proposals it contains, and what was new or contradictory relative to the existing wiki
4. **Identify affected pages** — scan `wiki/index.md` for any system, decision, incident, concept, or technology pages that this source touches
5. **Update affected pages** — add new information, note what changed, update the `date_updated` and `sources` fields in frontmatter; never delete prior information, mark it as superseded instead
6. **Create new pages** — for any system, decision, incident, concept, or technology mentioned that does not yet have a page
7. **Update glossary** — add new internal terms, update definitions if they changed, move deprecated names to the "Deprecated / Avoid" table
8. **Update index** — add all new pages; update one-line summaries for pages that changed significantly
9. **Update overview** — if this source shifted the big-picture architectural understanding, update `wiki/overview.md`; update the "Open Questions" section if the source raised unresolved issues
10. **Log it** — append to `wiki/log.md`: `## [YYYY-MM-DD] ingest | <document title>` followed by a 2-3 line summary of what changed
11. **Stage the source** — run `git add "raw/<filename>"` so the file is marked as processed; future duplicate checks will see it as clean

A single source typically touches 10-20 wiki pages.

---

## Query Workflow

When the human asks a question:

1. Read `wiki/index.md` to find relevant pages
2. Read each relevant page in full
3. Synthesize an answer grounded in documented decisions, systems, and incidents — cite specific wiki pages
4. Flag where the wiki doesn't have enough information to answer confidently
5. Offer to save the answer as an `analysis` page: "Should I save this as a wiki page?"
6. If yes, create `wiki/analysis-<slug>.md`, update the index, and log the query

**Every query response MUST end with two mandatory sections — never skip them:**

```
**Wiki gaps:** [list any information the wiki lacks to answer this fully, or "None identified"]

**Save as analysis page?** Should I save this answer to `wiki/analysis-<slug>.md`?
```

These sections are not optional. If you finish a query answer without them, you have not completed the workflow.

---

## Lint Workflow

When the human says "lint the wiki":

1. **Contradictions** — scan for claims across pages that contradict each other; list them
2. **Stale pages** — find pages where `date_updated` is significantly older than recent ingests that likely affect them
3. **Orphan pages** — find pages with no inbound links from other wiki pages
4. **Undocumented dependencies** — find systems that reference other systems not yet in the wiki
5. **Terminology drift** — find the same component or concept referred to by different names across pages
6. **Missing pages** — find concepts or systems mentioned in multiple pages but lacking their own page
7. Report findings to the human grouped by severity; ask which to fix before making any changes

---

## Glossary Conventions

`wiki/glossary.md` has four tables:

- **Canonical Terms** — the correct name for each internal concept, system, or component
- **Aliases and Abbreviations** — alternate names that map to canonical terms
- **Deprecated / Avoid** — old names, replaced names, competitor terms that don't apply
- **External Terms** — industry terms used in sources, with our internal equivalent if different

Always check the glossary before creating a new page. If a system appears under an alias, use the canonical name as the page title.

---

## Cross-Reference Conventions

- Use `[[Page Title]]` for all internal links — Obsidian will resolve them
- Every page must have at least one inbound link from another page (except index, log, overview, glossary)
- The "Related Pages" section at the bottom of each page is mandatory
- Decision pages must link to the system they affect
- Incident pages must link to every system involved

---

## Log Format

Every log entry must start with a parseable header:

```
## [YYYY-MM-DD] <operation> | <title>
```

Where `<operation>` is one of: `ingest`, `query`, `lint`, `create`, `update`.

This makes the log greppable: `grep "^## \[" wiki/log.md | tail -10`

---

## What You Must Never Do

- Modify anything in `raw/` — it is immutable
- Delete information from a wiki page — mark it `status: superseded` and note why
- Create wiki pages in the root `wiki/` folder without adding them to `index.md`
- Use terminology inconsistent with `wiki/glossary.md`
- Leave a lint finding unlogged — always record what was found, even if not fixed

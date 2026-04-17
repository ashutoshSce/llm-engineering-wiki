# LLM Wiki (Karpathy Pattern)

A self-maintaining personal knowledge base powered by LLMs, based on [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md).

Instead of re-searching raw documents on every question (like RAG), the LLM **reads your sources once and builds a persistent, interlinked wiki** that compounds over time. Feed it your Request for Comments(RFCs), Architecture Decision Records(ADRs), Incident Retrospective (Post-Incident Review), Design docs, Slack Discussions, Email Threads, and Transcripts — the wiki keeps everything structured, cross-referenced, and up to date.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) (recommended) or any LLM agent that reads a schema file (Cursor, Codex, Gemini CLI)
- [Obsidian](https://obsidian.md/) (free) for browsing the wiki in real time

---

## Getting Started

### 1. Clone the repo

```bash
git clone [YOUR-REPO-URL]
cd llm-wiki
```

### 2. Open in Claude Code

Open the project folder. Claude reads `CLAUDE.md` automatically at the start of every session and understands the wiki structure, entity types, and all workflows.

If you use a different AI agent (Cursor, Codex, Gemini CLI), paste the contents of `CLAUDE.md` into your agent's context at the start of each session.

### 3. Open in Obsidian

```bash
brew install --cask obsidian
```

Open the project folder as an Obsidian vault. Everything is pre-configured — graph view, hotkeys, sidebar layout. No manual setup needed.

You'll have two windows side by side: Claude Code on the left where you talk to the AI, Obsidian on the right where you watch wiki pages appear in real time.

### 4. Install poppler (for PDF ingestion)

```bash
brew install poppler
```

This enables Claude to read PDF files directly. Without it, PDFs will need to be converted to text before ingesting.

### 5. Drop a source into `raw/`

Anything works:

- An RFC or ADR from your repo
- A postmortem or incident report
- A system design doc or architecture review
- A meeting transcript (engineering sync, incident debrief, 1:1 notes)
- A Confluence page exported as markdown
- A vendor evaluation or benchmark report
- An engineering blog post or deep-dive you want to reference

### 5. Say "ingest"

```
Ingest raw/kafka-vs-sqs-adr-2024.md
```

The AI will:

1. Read the document
2. Surface key decisions and trade-offs with you
3. Create a source summary page in `wiki/sources/`
4. Create or update pages for every system, component, and concept it finds
5. Update the glossary with internal terminology
6. Update the index
7. Update the overview if the big picture shifted
8. Log everything in `wiki/log.md` with a timestamp

A single source can touch 10-20 wiki pages. Watch them appear in Obsidian as it works.

### 6. Ask questions

```
What were the trade-offs we accepted when moving to event-driven architecture?
```

The AI reads the wiki — not the raw documents — and composes a grounded answer with citations. When the answer is useful:

```
Save this as an analysis page.
```

Your questions become permanent pages. The knowledge base grows with every query.

### 7. Lint the wiki

After major architectural changes, or every 10 ingests:

```
Lint the wiki
```

The AI checks for contradictions between pages, stale descriptions superseded by newer decisions, orphan pages with no inbound links, and terminology drift. It reports what it found and asks which fixes to apply.

---

## Repo Structure

```
llm-wiki/
├── CLAUDE.md          # Schema — the AI's operating manual
├── llm-wiki.md        # Karpathy's original idea document
├── engineers-nightmare.md  # Walkthrough article explaining this project
│
├── raw/               # Your source documents (AI reads, never writes)
│   └── .gitkeep
│
├── wiki/              # AI-generated knowledge base (AI owns this layer)
│   ├── index.md       # Master catalog — the AI reads this first on every query
│   ├── overview.md    # Architecture synthesis (evolves with each ingest)
│   ├── glossary.md    # Internal terminology, aliases, deprecated names
│   ├── log.md         # Append-only record of all activity
│   └── sources/       # One summary page per raw document
│
└── .obsidian/         # Pre-configured Obsidian vault settings
```

### How the layers work

| Layer | Folder | Who owns it | Purpose |
|-------|--------|-------------|---------|
| Raw sources | `raw/` | You | Immutable source documents. The AI reads from here but never modifies anything. |
| The wiki | `wiki/` | The AI | Structured, interlinked markdown pages. The AI creates, updates, and maintains everything here. |
| The schema | `CLAUDE.md` | You + AI | Defines entity types, workflows, and page formats. Edit this to customise the AI's behaviour for your domain. |

### Wiki page types

The AI creates pages based on what it finds in your sources:

| Type | What it captures |
|------|-----------------|
| `source` | Summary of a raw document — key decisions, concepts, contradictions |
| `system` | A service, platform, or significant subsystem |
| `component` | A module, library, or internal building block |
| `decision` | An ADR — a specific technical choice with rationale and alternatives |
| `incident` | A postmortem — failure, root cause, resolution, lessons learned |
| `rfc` | A proposal — problem, proposed solution, open questions, decision |
| `concept` | A technical pattern or principle (e.g. "circuit breaker", "eventual consistency") |
| `technology` | An external tool, framework, or platform — why adopted, known issues |
| `team` | An engineering team — responsibilities, owned systems |
| `analysis` | A synthesised answer to a question, saved for future reference |

---

## Customising for Your Domain

The schema file `CLAUDE.md` is not set in stone. Edit it to fit your needs:

- **Add new entity types.** If your domain needs `runbook`, `slo-definition`, `api-contract`, or `oncall-playbook`, add them to the entity types table and tell the AI.
- **Change the ingest workflow.** If you want the AI to skip the discussion step and process silently, update the workflow section.
- **Adjust the session start checklist.** Add or remove steps based on what context the AI needs at the start of each session.

---

## Tips

**Ingest decisions with their context.** An ADR without the RFC that motivated it is a conclusion without reasoning. When you ingest an ADR, also ingest the RFC it came from and any postmortems that shaped it.

**Save useful queries as pages.** When you ask a question and get a good answer, save it. That synthesis — built from multiple source documents — would take time to reconstruct manually. Now it lives in the wiki and gets referenced by future ingests.

**Use graph view.** Press `Cmd+G` in Obsidian. Pages with no inbound links are orphans — things that exist but nothing references. Pages with many inbound links are hubs — high blast-radius components worth extra attention.

**Lint after significant changes.** After a major refactor or team restructure, run a lint pass. The AI finds every page that still references the old state and every term that has drifted in meaning.

**Stay out of `wiki/`.** Your job is finding good sources and asking precise questions. The AI handles the writing, cross-referencing, and consistency checking. That division is what keeps the system trustworthy.

---

## Use Cases

- **Architecture decisions** — Ingest every ADR and build a living decision landscape with rationale, alternatives, and status. Lint flags decisions that rest on stale assumptions.
- **Incident retrospectives** — Ingest postmortems and build pattern pages. When a new incident looks familiar, the wiki tells you why.
- **RFC tracking** — Ingest the proposal and the outcome. The wiki tracks what changed, what was accepted, and what was deferred.
- **Onboarding** — Feed the wiki every design doc and ADR for a system. A new engineer reads the wiki instead of pairing with you for two weeks.
- **Tech debt mapping** — Ingest retros and health check documents. The wiki builds a living debt map — what exists, why it exists, which systems are affected.
- **Cross-team context** — Ingest design docs from teams you depend on. Track their decisions and contracts without attending all their meetings.

---

## Credits

- Pattern by [Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md)
- Referenced Technical Writer Implementation by [Balu Kosuri](https://github.com/balukosuri/llm-wiki-karpathy)
- Referenced Youtube, Karpathy's LLM Wiki - Full Beginner Setup Guide by [Teacher's Tech](https://www.youtube.com/watch?v=iXd0t60YmMw)
- Software Engineering LLM Wiki Implementation by [Ashutosh Jha](https://github.com/ashutoshSce)

# The Engineer's Nightmare: Knowledge Scattered Everywhere — Setup a Knowledge Base Quickly

*How Andrej Karpathy's LLM Wiki pattern, Claude Code, and Obsidian turned a scattered mess of RFCs, ADRs, postmortems, and design docs into a self-maintaining engineering knowledge base — without me writing a single wiki page.*

---

Here is a scenario every engineer knows too well.

You have documents scattered across a dozen places. An RFC from 18 months ago. An ADR buried in a repo nobody remembers. A postmortem PDF in someone's Drive. A Confluence page that hasn't been touched since the original author left. Three Slack threads where the real decision happened, now impossible to search.

Six months later, a new engineer joins. They ask: "Why did we choose Kafka over SQS here?" And you spend an hour digging through git blame, Confluence search, and your own memory trying to reconstruct a decision that was obvious to everyone at the time.

The knowledge exists. The reasoning is in there somewhere. But it is scattered, disconnected, and buried — and the cost of recovering it falls entirely on whoever gets asked.

I found a way to fix this. It involves a one-page idea document by Andrej Karpathy, an AI agent called Claude Code, and a free note-taking app called Obsidian. In about 30 minutes, I went from that one-page idea to a fully working personal knowledge base — ready to accept any document I throw at it and turn it into structured, interlinked wiki pages covering systems, decisions, trade-offs, and context.

And I didn't write a single wiki page myself.

This post tells the story of how it happened and links to a repo you can clone and start using today.

---

## Who Is Andrej Karpathy?

Andrej Karpathy is one of the most recognized names in AI engineering. Co-founder of OpenAI, former Director of AI at Tesla where he led the Autopilot vision team, and the person who coined the term "vibe coding." When Karpathy shares a systems thinking idea, the engineering community pays attention.

In early April 2026, he published a short GitHub Gist called `llm-wiki.md`. It was not a product. Not a library. Just a plain markdown document describing a pattern — a way to use AI agents to build and maintain personal knowledge bases that compound over time.

The document was designed to be dropped into any capable AI agent (Claude, Codex, Gemini). The agent reads it, understands the pattern, and builds a working implementation tailored to your domain.

**Original document:** [Karpathy's llm-wiki.md](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md)

That one-page idea is the foundation of everything in this repo.

---

## What Is LLM Wiki and How Does It Work?

The core idea maps cleanly to how engineers already think about systems.

Most AI tools work like query engines: you give them documents, ask a question, the AI searches and answers. It works — but it forgets. The next query starts from scratch. Nothing accumulates. Nothing builds on prior synthesis. It is stateless retrieval.

Karpathy's analogy is compilation. When you write source code, you don't execute the source directly every time — you compile it into an optimized artifact and run that. The compilation is expensive once. Everything after is fast.

**LLM Wiki applies the same logic to knowledge.** Instead of re-searching raw documents on every query, the AI reads your sources once and compiles them into a structured wiki — summary pages, system pages, concept pages, ADR pages, decision rationale pages — all interlinked. When you add a new document, the AI doesn't re-process everything. It reads the new source, updates affected pages, creates new ones, resolves contradictions, and keeps the whole structure current.

**The wiki is the persistent artifact.** It compounds. The more you feed it, the richer and more connected it becomes.

Karpathy ran this pattern on a personal research topic over several months. His wiki grew to approximately [**100 articles and 400,000 words**](https://venturebeat.com/data/karpathy-shares-llm-knowledge-base-architecture-that-bypasses-rag-with-an) — longer than most PhD dissertations — without him directly writing any of it. That is not a one-week result; it is what the pattern produces when you keep feeding it over time.

### The three layers

**1. Raw sources — `raw/`**
This is where your documents live: RFCs, ADRs, postmortems, architecture diagrams exported as markdown, meeting transcripts, PR descriptions, system design docs, tech specs, incident reports. The AI reads from this folder but never modifies it. Your originals are untouched.

**2. The wiki — `wiki/`**
The AI creates and owns everything here. System pages, trade-off pages, decision records, concept pages, component catalogs, cross-references, a glossary of internal terminology, and an index. You browse it. The AI writes it.

**3. The schema — `CLAUDE.md`**
This is the rulebook that turns a general-purpose AI agent into a disciplined engineering wiki maintainer. It defines entity types (systems, components, decisions, incidents, concepts), page formats, ingestion workflows, and consistency rules. Edit this file to change how the AI handles your specific domain.

### Three operations

**Ingest:** Drop a document into `raw/` and tell the AI to process it. It reads the source, creates summary and entity pages, updates existing pages with new information, flags contradictions with prior sources, updates the glossary, and logs everything with a timestamp. A single RFC can touch 10-20 wiki pages.

**Query:** Ask a question. The AI reads the wiki — not the raw files — to compose an answer grounded in your documented decisions and systems. Good answers can be saved back as analysis pages, making your questions compound into the knowledge base.

**Lint:** Health-check the wiki. The AI finds contradictions between pages, stale claims superseded by newer decisions, orphan pages with no inbound links, undocumented components, and terminology inconsistencies. Think of it as `go vet` for your engineering knowledge.

---

## What I Did: Two Prompts in Claude Code

Here is exactly what happened. I opened Claude Code, dropped Karpathy's `llm-wiki.md` into an empty project folder, and started a conversation.

### Prompt 1: "What is this and how can I make use of this as an engineer?"

*Claude's response, unprompted on the role-specific breakdown:*

> **LLM Wiki** is a pattern for building a personal knowledge base that an AI agent maintains for you.
>
> Most AI tools work like RAG: you upload documents, ask a question, the AI searches and answers. It works — but it forgets. Every query starts from scratch. Nothing accumulates.
>
> This is different. The AI reads your sources once and **compiles** them into a structured, interlinked wiki. When you add a new source, the AI updates existing pages, creates new ones, flags contradictions, and keeps everything current. The wiki is the persistent artifact. It compounds over time.
>
> For an engineer, this maps directly to your biggest day-to-day knowledge problem: **context is scattered across systems, teams, tools, and time.** Here is how the pattern addresses it:
>
> | Your pain point | How LLM Wiki solves it |
> |---|---|
> | Decisions live in people's heads, Slack, and stale ADRs | Ingest them all — the AI builds a living decision record with rationale and trade-offs |
> | New engineers ask "why did we do X?" and nobody has a fast answer | The wiki maintains reasoning and context for every decision |
> | Postmortems get written and never referenced again | Ingest them — the AI extracts failure patterns and links them to affected components |
> | RFC context gets lost between proposal and implementation | Ingest both — the AI tracks what changed, what was accepted, what was deferred |
> | Tech stack drift — nobody remembers why you adopted a tool | The wiki maintains a living technology inventory with adoption rationale |
> | Onboarding a new engineer costs weeks of your time | Feed the wiki first — it becomes the context layer that doesn't require you |

The AI mapped this to my role without me asking. I hadn't mentioned any of those pain points.

### Prompt 2: "Can you make a plan and create it?"

Five words. Claude planned the project structure and built it in one pass:

```
project-root/
│
├── llm-wiki.md               # Karpathy's original idea document
├── CLAUDE.md                 # Schema — entity types, page formats, all workflows
│
├── raw/                      # Your source documents (AI reads, never writes)
│   └── .gitkeep
│
├── wiki/                     # AI-generated knowledge base
│   ├── index.md              # Master catalog of all pages
│   ├── log.md                # Ingest and query history with timestamps
│   ├── overview.md           # Synthesis page — evolves over time
│   ├── glossary.md           # Internal terminology, aliases, deprecated names
│   └── sources/              # One summary per raw document
│
└── .obsidian/                # Pre-configured Obsidian vault
    ├── app.json              # File paths, link behavior
    ├── appearance.json       # Theme, font size
    ├── core-plugins.json     # Which plugins are active
    ├── graph.json            # Graph colors by entity type
    ├── hotkeys.json          # Keyboard shortcuts
    └── workspace.json        # Default tabs and sidebar layout
```

The schema was the impressive part. It defined:

- **10 entity types** — `system`, `component`, `decision`, `incident`, `rfc`, `concept`, `technology`, `team`, `analysis`, `source`
- **Page format** — YAML frontmatter for every page (title, type, date, sources, status, tags), structured body, "Related Pages" section with wiki-style links
- **Ingest workflow** — 10 steps:
  1. **Read the source** — read the full document in `raw/`
  2. **Discuss takeaways** — surface key decisions and trade-offs; ask what to emphasize before writing anything
  3. **Create a source summary** — write `wiki/sources/<slug>.md` documenting what the source contains and what was new or contradictory
  4. **Identify affected pages** — scan `wiki/index.md` for systems, decisions, incidents, and concepts the source touches
  5. **Update affected pages** — add new information, mark superseded claims; never delete prior content
  6. **Create new pages** — for any system, decision, incident, concept, or technology not yet in the wiki
  7. **Update the glossary** — add new internal terms, update definitions, move deprecated names to the "Avoid" table
  8. **Update the index** — add all new pages; refresh one-line summaries for pages that changed significantly
  9. **Update the overview** — if the source shifted the big-picture understanding, update `wiki/overview.md` and flag new open questions
  10. **Log it** — append `## [YYYY-MM-DD] ingest | <title>` to `wiki/log.md` with a 2-3 line summary of what changed
- **Query workflow** — read the index first, find relevant pages, synthesize an answer, offer to save it as an analysis page
- **Lint workflow** — contradictions, staleness, orphan pages, terminology drift
- **Session start checklist** — read schema → read index → read last 5 log entries → ask
- **Log format** — greppable with `grep "^## \[" wiki/log.md | tail -10`

All of this from a five-word prompt.

**The infrastructure was done.** A clean, empty knowledge base ready for the first source.

---

## Why This Structure Works

- **Clear separation of ownership.** `raw/` is yours — the AI never touches it. `wiki/` is the AI's — you browse it but don't write in it. This boundary is what makes the system reliable over time.
- **The schema is the contract.** `CLAUDE.md` defines entity types, page formats, and workflows. Edit it to add types specific to your domain (API contracts, runbooks, service catalogs). The AI reads this file first and follows it precisely.
- **The index replaces embeddings.** When you ask a question, the AI reads `index.md` to find relevant pages, then drills into them. No vector databases, no RAG infrastructure — works well up to hundreds of pages.
- **The log is the audit trail.** Every ingest, query, and lint pass is timestamped. You know what was added, when, and from what source.
- **Obsidian is the interface.** The graph view makes the knowledge structure visible — which systems are hubs, which decisions are most referenced, which areas are sparse.

---

## How to Use This Repo

### Step 1: Clone it

```bash
git clone [YOUR-REPO-URL]
cd llm-wiki
```

### Step 2: Open in Claude Code

Open the project folder. Claude reads `CLAUDE.md` automatically at the start of every session.

If you use a different AI agent (Cursor, Codex, Gemini CLI), paste the contents of `CLAUDE.md` into your agent's context at the start of each session.

### Step 3: Open in Obsidian

```bash
brew install --cask obsidian
```

Open the project folder as an Obsidian vault. Everything is pre-configured — no manual setup needed.

### Step 4: Drop a source into raw/

Anything works:
- An RFC or ADR from your repo
- A postmortem or incident report
- A design doc or architecture review
- A meeting transcript (engineering sync, incident debrief, 1:1 notes)
- A vendor evaluation or benchmark report
- A Confluence page exported as markdown

### Step 5: Say "ingest"

```
Ingest raw/kafka-vs-sqs-adr-2024.md
```

The AI reads the document, surfaces key decisions with you, creates and updates wiki pages, updates the glossary and index, and logs everything. You watch pages appear in Obsidian as it works.

### Step 6: Ask questions

```
What were the trade-offs we accepted when moving to event-driven architecture?
```

The AI reads the wiki and composes a grounded answer. When it is useful:

```
Save this as an analysis page.
```

Your questions become permanent pages in the knowledge base.

### Step 7: Keep feeding it

Every new source builds on what came before. After 10-15 sources, the wiki starts surfacing connections you hadn't consciously registered — recurring trade-offs, technical debt patterns, gaps between what was planned and what was built.

### Step 8: Lint occasionally

After major changes, or every 10 ingests:

```
Lint the wiki
```

The AI checks for contradictions between pages, stale descriptions, orphan pages with no inbound links, and terminology drift. It reports what it found and asks which fixes to apply.

---

## What This Pattern Is Good For

### Architecture decisions (ADRs)

Stop letting ADRs die in a `docs/` folder nobody opens. Ingest them and the wiki builds a living decision landscape — what was decided, why, what alternatives were rejected, and what assumptions the decision rested on. When an assumption changes, lint flags every downstream decision that may now be invalid.

### Incident retrospectives

Every postmortem contains lessons. Most get written and never referenced again. Ingest your postmortems and the wiki builds pattern pages — recurring failure modes, systemic causes, components with a history of incidents. When a new incident looks familiar, the wiki tells you why.

### RFC tracking

Ingest the proposal, then ingest the outcome document weeks later. The wiki tracks what changed between proposal and decision, what was accepted, what was deferred, what concerns were raised and resolved.

### Onboarding

Joining a new system or onboarding a new teammate? Feed the wiki every design doc, ADR, and architecture review you can find. Within a day you have a synthesized knowledge base covering the system's history, its decisions, its trade-offs, and its internal terminology. The new engineer reads the wiki instead of pairing with you for two weeks.

### Tech debt visibility

Ingest sprint retros, tech debt tickets, and engineering health documents. The wiki builds a living debt map — what exists, why it exists, which systems are affected. Debt stops being invisible.

### Cross-team context

Ingest design docs from teams you depend on. The wiki tracks what contracts exist and what decisions those teams have made that affect you. When something breaks at a team boundary, the context is already documented.

---

## Example: An Engineer's First Week with LLM Wiki

A concrete walkthrough. You have just taken ownership of a legacy distributed system. Three teams have contributed to it over four years. Nobody has a complete mental model. The original architects have mostly moved on. Here is how LLM Wiki turns your first week into a knowledge base.

### Day 1 — Document archaeology

You collect every document you can find: the original design doc, two ADRs, a postmortem from 18 months ago, and an architecture review from last year.

Drop them into `raw/` and ingest them one at a time.

After the first document — the design doc — the wiki has a source summary, system pages for the major components, a decision page for each trade-off mentioned, and a glossary with the system's internal terminology. You spot something in Obsidian: a component is called "event bus" in the design doc but your team now calls it "message broker." You tell the AI. It updates the glossary and any affected pages.

The ADRs, postmortem, and architecture review each add context to the existing pages rather than creating duplicates. The postmortem creates an incident page linked to the components it affected. The architecture review flags two technical concerns that were raised and never resolved.

**End of day 1:** You have 12-15 wiki pages, a glossary, and an overview that synthesizes the system's history. You haven't written a single page. More importantly — you now know about two unresolved concerns from a year ago that are probably still problems.

### Day 2 — Talking to engineers

You schedule 30-minute conversations with engineers who have worked on the system. Record them. Run them through any transcription tool (Whisper, Otter, or `whisper` CLI).

Ingest the transcripts.

The AI extracts every technical opinion, tribal knowledge, and undocumented assumption it finds. It creates a "Known Issues" concept page from the problems everyone mentioned but nobody filed. It flags three places where the engineers' mental model contradicts what the design doc says — which means either the doc is outdated or there is a genuine knowledge gap.

You now have a concrete list of things to investigate, without having taken a single note during the conversations.

### Day 3 — Current state vs. design

You walk through the codebase. Export the service dependency graph as a markdown description. Document the current topology. Capture the infrastructure configuration.

Ingest these "current state" documents. The AI compares them against the design docs and surfaces every delta — places where what exists diverges from what was planned.

```
Based on the gap between the design docs and the current state, what are the highest-risk inconsistencies?
```

The AI reads the wiki and produces a ranked list. You save that as an analysis page. Now it is part of the knowledge base and will be referenced by future ingests and queries.

### Day 4 — Writing an RFC

You need to propose a change to address one of the unresolved concerns from Day 1. Before writing, you open `wiki/glossary.md` — every internal term, component name, and acronym with the correct definition. You check the decision pages to make sure your proposal doesn't quietly contradict a prior decision. You check the incident pages to confirm your change doesn't reintroduce a failure mode that was previously resolved.

You write the RFC informed by everything the wiki knows. No hunting through docs. The wiki is the single source of truth.

### Day 5 — Onboarding a new teammate

A new engineer joins. Instead of blocking your week:

```
Read wiki/overview.md first. Then come to me with questions.
```

Their first questions are things the wiki already covers. They go deeper faster. Your time goes toward the judgment calls and context that the wiki genuinely cannot provide — the things that actually require you.

### What you have after one week

- **12-20 wiki pages** covering the system's components, decisions, incidents, and terminology — realistic for 5-6 ingested documents
- **A living decision record** with rationale and status for every ADR
- **An overview page** summarising what the system is, what it was supposed to be, and where the gaps are
- **A risk analysis page** from Day 3, saved and linked in the overview
- **A log** showing exactly what was ingested and when — useful for the next engineer who joins
- **A graph view** in Obsidian showing how everything connects

When the next document lands — a new RFC, a postmortem from last night — you know what to do. Drop it in `raw/`. Say "ingest." The wiki grows.

---

## Tips That Helped

**Ingest decisions with their context.** An ADR without the RFC that motivated it is a conclusion without reasoning. When you ingest an ADR, also ingest the RFC it came from and any postmortems that shaped the decision.

**Save useful queries as pages.** When you ask a question and get a good answer, save it. That synthesis — built from multiple source documents — would take time to reconstruct. Now it lives in the wiki permanently and gets referenced by future ingests.

**Use the graph view.** Pages with no inbound links are orphans — things that exist but nothing depends on. Pages with many inbound links are hubs — high blast-radius components worth extra attention. The graph shows you things about system health that no linear document can.

**Lint after significant changes.** After a major refactor or team restructure, run a lint pass. The AI finds every page that still references the old state, every ADR that assumed something that is no longer true, and every glossary term that has drifted in meaning.

**Edit the schema for your domain.** `CLAUDE.md` ships with common engineering entity types. Add the types your team actually uses — `runbook`, `slo-definition`, `api-contract`, `oncall-playbook`. The AI adapts immediately.

**Stay out of `wiki/`.** It is tempting to edit pages directly. Resist it. Your job is finding good sources and asking precise questions. The AI's job is the writing, cross-referencing, and consistency checking. That division is what keeps the system trustworthy.

---

## Why This Pattern Works

The reason engineering knowledge bases fail is not that engineers stop caring. It is that maintenance always loses to shipping.

Think about what maintenance actually means: updating cross-references when a service is renamed, keeping docs current after a refactor, checking that ADR-023 doesn't quietly contradict ADR-011, making sure the postmortem from six months ago is reflected in the component pages. This work is important and nobody does it consistently.

AI changes the equation. The AI never gets tired of maintenance. It cross-references automatically. It flags contradictions. It keeps terminology consistent across 50 pages. The maintenance cost drops to near zero — and the knowledge base stays useful instead of going stale.

That is Karpathy's core insight. The hard part of a knowledge base was never the initial capture. It was always the ongoing maintenance. And maintenance is exactly the kind of systematic, repetitive, high-precision work that AI is built for.

Your job becomes the interesting part: finding good sources, asking the right questions, and deciding what matters. The knowledge base grows. The next engineer who joins has context from day one. Decisions stop being re-derived from scratch. The team spends less time reconstructing the past and more time building the future.

---

## Get Started

The repo is ready to clone:

**GitHub:** [YOUR-REPO-URL]

What's included:

- Karpathy's original `llm-wiki.md` idea document
- `CLAUDE.md` — schema for engineering workflows (systems, ADRs, incidents, RFCs, concepts — edit it for your domain)
- Pre-configured Obsidian vault (graph view, hotkeys, sidebar layout)
- An empty `raw/` folder ready for your first source
- A `wiki/` folder with four starter pages: `index.md`, `log.md`, `overview.md`, `glossary.md`

Drop your first document. Say "ingest." The wiki writes itself.

---

*This repo is a starting point — clone it, adapt the schema to your domain, and start feeding it your team's documents.*

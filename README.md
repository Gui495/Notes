English | [中文](README.zh.md)
# Notes

An Obsidian knowledge base maintained jointly by humans and AI agents.

It uses the **LLM Wiki** model: raw materials go into `raw/`, agents compile them into structured `wiki/` pages, and `areas/` serves as the human's daily entry point. Knowledge grows **compounding** with each ingestion.

---
## Core Idea

The problem with traditional note tools is that the more notes you write, the more notes you have, but the knowledge does not get smarter. Every query requires searching again, old conclusions are not updated by new understanding, and cross-topic connections rely entirely on the human brain.

This repository treats the knowledge base as a **codebase**:

- **Raw materials = source code**, append-only, serving as traceable evidence
- **AI agent = compiler**, continuously compiling source into structured knowledge pages
- **wiki = build artifacts**, including concept pages, entity pages, summaries, comparisons, and syntheses
- **areas = user interface**, the place you enter the knowledge base from every day

With each new material ingestion, the agent does not merely index it; it **reads through the existing wiki and integrates it into the existing knowledge network**—updating concept pages, revising summaries, flagging contradictions, and creating bidirectional links. This is the essence of "self-growing": **the knowledge base gets smarter with use, not messier**.

---
## Three-Layer Architecture

| Layer | Maintained by | What goes here | Purpose |
|---|---|---|---|
| `raw/` | Humans write only, no edits | Web clippings, PDFs, class notes, code dumps, chat logs | Evidence warehouse, traceable |
| `wiki/` | Agent-owned | Concept pages, entity pages, summaries, comparisons, syntheses | Build artifacts, knowledge network |
| `areas/` | Human-led, agent drafts | Subject MOCs, learning maps, progress, todos | Daily entry point, cockpit |

Relationship among the three:

```
raw/  ──compile──▶  wiki/  ──aggregate──▶  areas/
raw material        knowledge             entry point
```

- `raw/` is an append-only archive; you do not need to open it every day
- `wiki/` is the agent's workspace; review it periodically
- `areas/` is the first place you enter after opening Obsidian every day

---
## Directory Structure

```
Notes/
├── AGENTS.md                  # Agent operation contract, must read first
├── README.md                  # This file
├── raw/                       # Raw material layer (read-only)
│   ├── College_Math/
│   ├── Linux/
│   └── Microcomputer_Principles/
├── wiki/                      # Compiled knowledge layer (agent-owned)
│   ├── concepts/              # Concept pages: one concept per page
│   ├── entities/              # Entity pages: people, tools, papers, courses
│   ├── summaries/             # Summary pages: distillation of a single raw material
│   ├── comparisons/           # Comparative analysis: side-by-side comparison of multiple concepts
│   ├── synthesis/             # Synthesis: cross-topic integration
│   └── _Index_Wiki.md         # Wiki master index
├── areas/                     # Human entry layer
│   ├── College_Math.md        # One MOC per subject
│   ├── Linux.md
│   ├── Microcomputer_Principles.md
│   └── _drafts/               # Agent-generated drafts, moved out after human review
└── _Index_All.md              # Full index (human-maintained)
```

Legacy subject folders (`College_Math/`, etc.) remain temporarily in the root directory and will be gradually triaged into `raw/` or `areas/`.

---
## Workflow

### Ingestion

Put learning materials into the corresponding subject directory under `raw/`. This can be:

- Markdown from web clippings
- Class notes
- Text extracted from PDFs
- Code snippets and comments
- Parts of AI conversations worth keeping

### Compilation

Tell the agent:

```
Process new materials in raw/
```

The agent will:

1. Scan `raw/` for uncompiled files
2. Create a summary page in `wiki/summaries/` for each material
3. Update or create relevant `wiki/concepts/` and `wiki/entities/` pages
4. Flag contradictions with existing pages
5. Update `wiki/_Index_Wiki.md`
6. Output a change report

### Query

Ask the agent directly. It should search `wiki/` first, fall back to `raw/` when not covered, and provide bidirectional-link citations.

### Review

Periodically tell the agent:

```
review
```

The agent will report: orphan pages, stalled seedlings, unresolved contradictions, and uncompiled raw materials.

### Proofreading

Open the agent-generated wiki pages and check:

- Whether the distillation is accurate (no content fabricated that does not exist in `raw/`)
- Whether bidirectional links point to pages that actually exist
- Whether frontmatter is complete
- Whether any duplicate pages should be merged

Proofreading feedback can be written back into `AGENTS.md` so the agent does better next time.

---
## Page Conventions

Each wiki page has frontmatter:

---
type: concept | entity | summary | comparison | synthesis
status: seedling | budding | evergreen
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - raw/College_Math/xxx.md
tags:
  - math
---

**Maturity**:

- `seedling` — only definitions and scattered points, links < 2
- `budding` — structured explanations, links 2–5
- `evergreen` — complete, cross-validated, links ≥ 5, readable independently

**Naming**: primarily Chinese, keep English for proper nouns. Do not use spaces or special symbols.

---
## Usage

### Obsidian

Open the repository root as a vault in Obsidian.

Recommended plugins:

- **Smart Connections** — passive semantic recall; automatically surfaces related pages while writing notes
- **Copilot for Obsidian** — vault-level chat and semantic search

### AI Agent

Open the repository root with dsh (or Claude Code / Codex). After startup, the agent must first read `AGENTS.md`, then operate according to permissions:

- `raw/` read-only
- `wiki/` readable and writable
- `areas/` read-only; can write drafts to `_drafts/`

---
## Permission Rules

| Directory | Agent Permission |
|---|---|
| `raw/` | Read-only; modification, moving, and deletion are prohibited |
| `wiki/` | Read/write; agent-owned |
| `areas/` | Read-only; can write to `_drafts/` |
| `_Index_All.md` | Read-only |
| Legacy subject folders in root | Read-only |

Operations beyond permissions are considered errors. Ask first when unsure.

---
## References

- [Andrej Karpathy's discussion of LLM Wiki](https://github.com/karpathy)
- [obsidian-llm-wiki-kit](https://github.com/) — out-of-the-box LLM Wiki template
- [NOUZ](https://github.com/) — Obsidian semantic MCP server
- [Tiago Forte, Building a Second Brain](https://www.buildingasecondbrain.com/) — PARA and CODE

---
## Status

Early setup stage. See `AGENTS.md` for core rules.
```
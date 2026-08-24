# LLM Wiki — Schema & Workflow

This is the schema document for this knowledge base. It defines the structure, conventions, and workflows for maintaining this wiki.

---

## Directory Structure

```
stock/                        ← Obsidian vault root
├── CLAUDE.md                 ← This file: LLM schema & instructions
├── sources/                  ← Raw sources (immutable, LLM reads only)
│   └── README.md
└── wiki/                     ← LLM-maintained knowledge pages
    ├── index.md              ← Catalog of all wiki pages
    ├── log.md                ← Append-only activity log
    ├── entities/             ← Pages for people, companies, products
    ├── concepts/             ← Pages for ideas, frameworks, theories
    ├── topics/               ← Broader topic overviews & syntheses
    └── sources/              ← One summary page per ingested source
```

---

## Core Rules

1. **Sources are immutable.** Files under `sources/` are never modified.
2. **The LLM owns `wiki/`.** All files under `wiki/` are created and maintained by the LLM.
3. **The user never writes wiki pages.** The user curates sources and asks questions.
4. **Every operation ends with updating `wiki/index.md` and `wiki/log.md`.**

---

## Page Conventions

### Frontmatter (all wiki pages)
```yaml
---
title: Page Title
type: entity | concept | topic | source-summary
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [source-filename.md]
---
```

### Cross-references
- Use Obsidian wiki links: `[[Page Name]]`
- Every page should link to at least 2–3 related pages
- When creating a new page, update existing related pages to link back

### Source Summary Pages (`wiki/sources/`)
Each ingested source gets one summary page with:
- **Origin**: title, author, date, URL/file
- **Key Takeaways**: 5–10 bullet points
- **Entities mentioned**: links to entity pages
- **Concepts mentioned**: links to concept pages
- **Contradictions/tensions**: anything that conflicts with existing wiki knowledge
- **Questions raised**: things worth following up

---

## Workflows

### INGEST — Adding a new source
When the user says "ingest [source]" or drops a file in `sources/`:

1. Read the source fully
2. Discuss key takeaways with the user (ask if anything should be emphasized)
3. Create `wiki/sources/<slug>.md` summary page
4. For each key entity (person, company, product): create or update `wiki/entities/<name>.md`
5. For each key concept: create or update `wiki/concepts/<name>.md`
6. Update relevant topic pages in `wiki/topics/`
7. Update `wiki/index.md` — add new pages, update touched pages
8. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | <Source Title>
   - Created: list of new pages
   - Updated: list of updated pages
   - Key additions: 1–2 sentence summary
   ```

### QUERY — Answering questions
When the user asks a question:

1. Read `wiki/index.md` to find relevant pages
2. Read the relevant pages
3. Synthesize an answer with citations (`[[Page Name]]`)
4. If the answer is substantial and reusable, offer to file it as a new wiki page

### LINT — Health check
When the user says "lint the wiki":

1. Check for contradictions between pages
2. Find orphan pages (no inbound links)
3. Find stale claims (superseded by newer sources)
4. Find missing cross-references
5. Find important concepts mentioned on many pages but lacking their own page
6. Report findings; ask user how to proceed

---

## index.md Format

```markdown
# Wiki Index
_Last updated: YYYY-MM-DD — N pages total_

## Entities
| Page | Summary | Sources |
|------|---------|---------|

## Concepts
| Page | Summary | Sources |

## Topics
| Page | Summary | Sources |

## Source Summaries
| Page | Original Source | Date Ingested |
```

## log.md Format

Append-only. Never edit existing entries.
Each entry starts with `## [YYYY-MM-DD] <operation> | <title>` for easy grep/parsing.

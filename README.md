# LLM Wiki: Building a Protein-Centered Knowledge Base for Papers with AI Agents

A methodology for using Claude Code or OpenAI Codex to build and maintain a structured, searchable wiki from origin PDFs and internal research reports. This version is adapted for protein and target research: papers are still organized by method/category, but the main navigation layer is now protein-centered.

This setup is based on joonan30's LLM Wiki guide:

https://gist.github.com/joonan30/cbce305684d079dbe9a3fbaefe4e3959

The original guide provides the general LLM Wiki pattern and `CLAUDE.md.template`. To use it here, customize that template for protein research and save it as `CLAUDE.md` in the project root. In this repository, `CLAUDE.md` is the agent's operating guide: it defines the Four Rules, folder structure, paper ingestion workflow, internal report workflow, naming convention, and protein-hub behavior.

> This is a starter template. Fork the structure, swap in your own protein targets and categories. The wiki only becomes useful once it reflects your domain, not someone else's.

## The Four Rules

The point of this wiki is to prevent hallucination by forcing every answer to be traceable to papers you actually have. Without these rules, the wiki turns into a dressed-up web search.

1. **No unsupported web filling.** Do not use web search to fill gaps in scientific answers.
2. **Answer from the wiki first.** `sources/` and `wiki/` are the primary sources of truth.
3. **If the wiki is insufficient, re-read the original PDF** in `papers/`. Then update the wiki.
4. **If no paper exists on the topic, say so.** Tell the user: "I don't have a paper on this; please give me the PDF." Do not improvise.

Apply these to every response, including overview pages: cite only papers that exist in the wiki.

The only exception is literature-location mode. In that mode, the agent may search for candidate papers and return titles, identifiers, and locations, but it should not create wiki claims until PDFs are ingested.

## The Concept

Inspired by Karpathy's LLM Wiki pattern and joonan30's biology-oriented implementation:

```text
Original PDF -> LLM markdown summary (sources/) -> Structured wiki page (wiki/) -> Protein hub and overview synthesis
Internal report -> normalized markdown (reports/) -> split report notes (sources/reports/) -> strategy wiki pages
```

Each paper goes through a 3-tier pipeline:

1. `papers/`: Original PDF, copied into the repository as an immutable archive.
2. `sources/`: LLM-generated structured summary with standard sections.
3. `wiki/{category}/`: Structured wiki page with cross-references using `[[wikilinks]]`.

This protein version adds a fourth navigation layer:

4. `wiki/proteins/`: Protein-level hub pages that collect all papers, disease evidence, mechanisms, drug development notes, biomarkers, genetics, and overview pages for one target.

Overview pages synthesize across papers. Protein hubs keep target-level knowledge easy to browse.

Internal reports use a separate report-derived pipeline:

1. `reports/`: Normalized markdown report, plus raw report files when useful.
2. `sources/reports/`: Topic-specific report notes.
3. `wiki/drug-design/` or `wiki/overviews/`: Report-derived strategy pages.
4. `wiki/proteins/`: The relevant protein hub links the report-derived section.

Report-derived claims are planning hypotheses unless supported by ingested academic PDFs.

## What Changed In The Protein Version

The original LLM Wiki is category-first. This version is protein-first.

The category folders still exist because they are useful for classifying evidence:

- `biochemistry`
- `structural-biology`
- `proteomics`
- `therapeutics`
- `concepts`
- `other`
- `overviews`

But the main reading path is:

```text
index.md -> wiki/proteins/{target}.md -> related paper pages and overviews
```

Protein hub pages are added for each target:

```text
wiki/proteins/ask1.md
wiki/proteins/pparg.md
```

Each protein hub should include:

- Protein snapshot
- Gene/protein aliases
- Core mechanism
- Mechanism papers
- Disease areas
- Therapeutic development notes
- Biomarkers or genetics
- Best overview pages
- Open questions
``
## Repository Structure

```text
protein-llm-wiki/
|-- CLAUDE.md               # Agent schema, workflow, and Four Rules
|-- README.md               # This file
|-- index.md                # Page catalog
|-- papers/                 # Original PDFs copied into the repository
|   `-- {author}-{year}-{title-5-words}.pdf
|-- reports/                # Internal reports normalized to markdown, plus raw report files
|   `-- {source}-{year}-{target}-{report-topic}.md
|-- sources/                # Structured PDF summaries
|   |-- {author}-{year}-{title-5-words}.md
|   `-- reports/            # Split source notes from internal reports
|-- wiki/                   # Final wiki pages
|   |-- proteins/           # Protein-level hub pages
|   |-- drug-design/         # Report-derived or paper-supported strategy pages
|   |-- biochemistry/
|   |-- structural-biology/
|   |-- proteomics/
|   |-- therapeutics/
|   |-- concepts/
|   |-- overviews/          # Cross-paper synthesis pages
|   `-- other/
|-- templates/
|-- scripts/
`-- requirements.txt
```

Keep the structure boring. The value comes from consistent papers, summaries, links, and overviews.

## Paper Naming Convention

All three paper tiers share the same stem:

```text
{first-author-lastname}-{year}-{first-5-title-words}.{ext}
```

Rules:

- Lowercase
- Strip special characters
- Replace spaces with `-`
- Use a four-digit year
- For consortium papers, use the consortium name when clearest

Example:

```text
sanyal-2010-pioglitazone-vitamin-e-or-placebo.pdf
sanyal-2010-pioglitazone-vitamin-e-or-placebo.md
```

## Adding A Paper

### Step 1: Copy PDF To `papers/`

Always copy the PDF into `papers/`. Do not symlink. Do not point `pdf_path` to `Downloads/`.

### Step 2: Extract Text

Use `pypdf`. In this repository, the bundled Codex Python runtime can run the helper script:

```powershell
& "$env:USERPROFILE\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe" scripts\extract_pdf_text.py papers\{stem}.pdf --max-pages 15 --max-chars 12000
```

The first 15 pages and about 12,000 characters are usually enough for a high-quality source summary.

### Step 3: Write `sources/{stem}.md`

Use the source template:

```text
templates/source-template.md
```

Each source page uses consistent YAML front matter:

```yaml
---
title: "Paper Title"
authors: Author List
year: YYYY
doi: DOI
category: your-category
pdf_path: C:/full/path/to/papers/{stem}.pdf
pdf_filename: {stem}.pdf
source_collection: external
---
```

Source pages use seven standard sections:

- One-line Summary
- Document Information
- Key Contributions
- Methodology and Architecture
- Key Results and Benchmarks
- Limitations and Future Work
- Related Work
- Glossary

### Step 4: Write `wiki/{category}/{stem}.md`

Use the wiki template:

```text
templates/wiki-page-template.md
```

Wiki pages use the same metadata plus:

```yaml
source: {stem}.md
tags: []
```

Standard sections:

- Summary
- Key Contributions
- Methodology and Architecture
- Results
- Related Papers

### Step 5: Update The Protein Hub

For protein-centered use, this is the key extra step.

Update:

```text
wiki/proteins/{target}.md
```

Add the new paper under the right section:

- Mechanism papers
- Disease areas
- Therapeutic development
- Genetics
- Biomarkers
- Overview links

If the target is new, create a new protein hub.

### Step 6: Update `index.md`

Add a one-line entry under the right category and make sure the protein hub is listed under `Proteins`.

## Knowledge Tree Method

The wiki grows by branching outward from real questions, not by dumping thousands of papers at once.

```text
Root target or question (e.g., "ASK1 in fibrosis")
|-- 1st wave: core mechanism papers
|-- 2nd wave: disease-specific papers
|-- 3rd wave: drug development and biomarker papers
`-- overview pages that synthesize across the branches
```

In practice:

1. Ask a question.
2. The agent searches `sources/` and `wiki/`.
3. If the wiki is insufficient, the agent re-reads local PDFs.
4. If no paper exists, the agent asks for the PDF.
5. Useful answers become overview pages in `wiki/overviews/`.
6. Protein hubs are updated so the target-level story stays current.

Over time, each target becomes a compact, cross-linked research memory.

## Working Modes

### Mode A: PDF Ingestion Mode

Use when PDFs are available.

```text
PDF -> papers/ -> sources/ -> wiki/{category}/ -> wiki/proteins/ -> index.md
```

This is the normal mode for building durable knowledge.

### Mode B: Literature Location Mode

Use when the user asks for papers but no PDF has been provided.

Return:

- Title
- Authors
- Year
- Journal
- PMID, DOI, or URL
- Source location
- Short reason for relevance

Do not create wiki pages in this mode. Once PDFs are provided, switch to PDF ingestion mode.

### Mode C: Internal Report Ingestion Mode

Use when the user provides an internal report, AI-generated target review, feasibility report, drug design report, competitive assessment, or strategy document.

```text
report.md / report.html / report.pdf
-> reports/{normalized-report}.md
-> sources/reports/*.md
-> wiki/drug-design/*.md or wiki/overviews/*.md
-> wiki/proteins/{target}.md
-> index.md
```

Internal reports are not academic papers by default. They can guide strategy, assay planning, ADMET thinking, competitive landscape, and literature backlog, but scientific and clinical claims remain `hypothesis` unless supported by ingested academic PDFs.

Report input handling:

- `.md`: copy or clean directly into `reports/`.
- `.html`: remove UI-only elements and convert meaningful content into cleaned markdown under `reports/`.
- `.pdf`: read the PDF or extract text when useful, then convert the report content into cleaned markdown under `reports/`.
- Keep raw original paths in metadata when useful, such as `original_pdf` or `original_html`.
- Use the normalized markdown file as `original_report` for downstream report-derived pages.

Recommended split for drug design reports:

| Report section | Source note | Wiki page |
|---|---|---|
| Target biology | `sources/reports/{target}-{source}-target-biology.md` | `wiki/drug-design/{target}-target-biology.md` |
| Structural analysis | `sources/reports/{target}-{source}-structural-analysis.md` | `wiki/drug-design/{target}-structural-design-map.md` |
| Existing drugs | `sources/reports/{target}-{source}-existing-drugs.md` | `wiki/drug-design/{target}-existing-drug-landscape.md` |
| Small molecule strategy | `sources/reports/{target}-{source}-small-molecule-strategy.md` | `wiki/drug-design/{target}-{strategy}.md` |
| Assay cascade | `sources/reports/{target}-{source}-assay-cascade.md` | `wiki/drug-design/{target}-assay-cascade.md` |
| ADMET/safety | `sources/reports/{target}-{source}-admet-safety.md` | `wiki/drug-design/{target}-admet-safety-risk.md` |
| Research tools | `sources/reports/{target}-{source}-research-tools.md` | `wiki/drug-design/{target}-research-tool-biology.md` |
| Data gaps | `sources/reports/{target}-{source}-data-gaps.md` | `wiki/drug-design/{target}-data-gaps-and-literature-backlog.md` |
| Roadmap | `sources/reports/{target}-{source}-roadmap.md` | `wiki/overviews/{target}-drug-design-roadmap.md` |


## Categories

Classify by method or evidence type, not just topic.

| Category | Includes |
|---|---|
| `proteins` | Protein hub pages |
| `biochemistry` | Mechanisms, pathways, assays, functional biology |
| `structural-biology` | Structures, complexes, binding architecture |
| `proteomics` | Proteome signatures, biomarkers, large-scale protein profiling |
| `therapeutics` | Clinical trials, inhibitors, agonists, drug development |
| `drug-design` | Report-derived or paper-supported strategy pages, assay cascades, ADMET/safety plans, research-tool biology, and roadmaps |
| `concepts` | Reusable background explanations |
| `overviews` | Cross-paper synthesis pages |
| `other` | Genetics, epidemiology, miscellaneous evidence |

Example: a proteomics paper in diabetic kidney disease goes to `proteomics`, then links from the relevant protein hub.

## When To Scale Up

Hold off until you actually need to.

- If a category passes about 500 files, split it.
- If the total wiki passes about 500 pages, consider a stronger search layer such as QMD or another local search system.

Below that scale, `index.md`, `rg`, Obsidian search, and the agent's built-in search are enough.

## Rules In `CLAUDE.md`

`CLAUDE.md` is the real control surface for the agent. It should include:

```markdown
# All wiki content in Korean or English.
# Conversation can be in any language.
# PDFs are copied into papers/ and never symlinked.
# pdf_path always points inside papers/.
# Every source and wiki page has consistent YAML front matter.
# Answer from sources/ and wiki/ first.
# If the wiki is insufficient, re-read the PDF and update the wiki.
# If no paper exists, ask for the PDF.
# For protein research, update wiki/proteins/{target}.md whenever a paper belongs to a tracked target.
# Internal reports go through reports/ -> sources/reports/ -> wiki/drug-design/ or wiki/overviews/.
# Report-derived claims are hypotheses unless supported by ingested academic PDFs.
# Report PDFs are not treated as academic papers by default; classify by content and user intent.
```
```

In other words: the gist gives the template; this repository uses `CLAUDE.md` as the customized protein-version implementation.

## Getting Started

1. Install Claude Code or OpenAI Codex.
2. Open an empty folder.
3. Paste the git URL:

```text
https://github.com/arontier/llm-wiki
```

4. Ask the agent:

```text
Set up an LLM Wiki for protein papers, following this gist.
Use the CLAUDE.md template from the gist, customize it for protein and target research, and save it as CLAUDE.md.
Create papers/, sources/, wiki/, wiki/proteins/, and wiki/overviews/.
```

5. Drop in the first PDFs.
6. Ask:

```text
Add these papers to the wiki.
```

7. Start asking target-level questions.
8. Save good answers as overview pages.

## Recommended: Use Obsidian For Browsing

The agent handles ingestion and Q&A. Obsidian is useful for reading and navigating.

1. Install Obsidian from https://obsidian.md/
2. Open this repository folder as a vault.
3. Start from `index.md` or `wiki/proteins/`.

Obsidian gives:

- Graph view through `[[wikilinks]]`
- Clickable paper relationships
- Full-text search
- Tag search
- Outline navigation

Obsidian only reads Markdown files, so it works cleanly alongside the agent.


These are examples, not limits. Add a new protein hub whenever a new target becomes part of the wiki.

---

Built from the LLM Wiki pattern described by joonan30 and inspired by Karpathy's original LLM Wiki idea. This repository adapts the approach for protein-centered literature research.

# Protein Literature Wiki

A personal knowledge base of protein biology papers, following the LLM Wiki pattern:

```text
Original PDF -> sources/*.md (structured paper summary) -> wiki/{category}/*.md (final page)
Internal report -> sources/reports/*.md (split report notes) -> wiki/drug-design/*.md or wiki/overviews/*.md
```

Language policy: Most generated content may be written in Korean. Conversation with the user can be in any language.

## Core Rules

These rules keep every claim traceable to local evidence that exists in this repository.

1. Do not use web search to answer scientific questions or fill gaps in wiki content.
2. Answer from `sources/` and `wiki/` first.
3. If the wiki is insufficient, re-read the matching PDF in `papers/`, extract more detail, and update the relevant source and wiki pages.
4. If the wiki has no paper on the topic, say: "I don't have a paper on this; please give me the PDF." Do not improvise.
5. Literature Location Mode is the only exception to rule 1. Use it only when the user asks to find papers or provides a protein, gene, target, pathway, disease, indication, or biological concept without PDFs. Return locations and identifiers only. Do not generate wiki claims.
6. Internal reports can guide hypotheses, strategy, assays, and literature backlogs, but they are not equivalent to papers. Mark report-derived claims as hypotheses unless they are supported by ingested PDFs.

These rules apply to every response, including overview pages. Cite only papers that exist in the wiki unless operating in Literature Location Mode.

## Evidence Hierarchy

Use this priority order when answering, writing wiki pages, or resolving conflicts:

1. Local PDFs in `papers/`
2. Structured paper summaries in `sources/`
3. Wiki pages derived from local PDFs
4. Archived internal reports in `reports/`
5. Split report notes in `sources/reports/`
6. Report-derived strategy pages in `wiki/drug-design/` or `wiki/overviews/`

Internal reports are useful as planning files, but they are not primary scientific evidence. A report-derived scientific or clinical claim must be labeled as `hypothesis` unless supported by a local PDF that has already been ingested.

## Repository Structure

```text
protein-wiki/
|-- CLAUDE.md
|-- index.md
|-- papers/
|   `-- {author}-{year}-{title-5-words}.pdf
|-- reports/
|   `-- {source}-{year}-{target}-{report-topic}.md
|-- sources/
|   |-- {author}-{year}-{title-5-words}.md
|   `-- reports/
|       `-- {target}-{source}-{section-topic}.md
|-- wiki/
|   |-- proteins/
|   |-- drug-design/
|   |-- structural-biology/
|   |-- biochemistry/
|   |-- proteomics/
|   |-- computational-protein-modeling/
|   |-- protein-engineering/
|   |-- therapeutics/
|   |-- concepts/
|   |-- overviews/
|   `-- other/
|-- templates/
`-- scripts/
```

## File Naming Convention

All three tiers share the same stem:

```text
{first-author-lastname}-{year}-{first-5-title-words}.{ext}
```

- Use lowercase.
- Strip special characters.
- Replace spaces with hyphens.
- Use a 4-digit year.
- For consortium papers, use the consortium name.

Example:

```text
jumper-2021-highly-accurate-protein-structure.pdf
jumper-2021-highly-accurate-protein-structure.md
```

## Categories

Classify by primary method or evidence type, not by phenotype alone.

| Category | Includes |
|---|---|
| `structural-biology` | X-ray crystallography, cryo-EM, NMR, structures, conformations, protein complexes |
| `biochemistry` | Enzyme mechanisms, binding assays, kinetics, mutagenesis, functional validation |
| `proteomics` | Mass spectrometry, protein abundance, PTMs, interactomics, large-scale profiling |
| `computational-protein-modeling` | Protein structure prediction, docking, molecular dynamics, sequence models, protein language models |
| `protein-engineering` | Directed evolution, rational design, variant libraries, stability design, enzyme or binder engineering |
| `therapeutics` | Antibodies, biologics, degraders, inhibitors, target validation with therapeutic intent |
| `proteins` | Protein-level hub pages that group all papers, diseases, mechanisms, and drug development notes for one target |
| `drug-design` | Report-derived or paper-supported strategy pages, assay cascades, ADMET/safety plans, competitive/IP notes, and development roadmaps |
| `concepts` | Methods, assays, algorithms, and reusable background explanations |
| `overviews` | Synthesis pages spanning multiple papers |
| `other` | Cross-cutting or miscellaneous papers that do not fit cleanly elsewhere |

Split a category once it grows beyond roughly 500 files.

## Mode A: PDF Ingestion Mode

Use this mode when the user provides one or more PDF files.

Workflow:

```text
Original PDF -> papers/*.pdf -> sources/*.md -> wiki/{category}/*.md -> index.md
```

Rules:

- Copy the PDF into `papers/`.
- Never symlink PDFs.
- Extract text from the PDF.
- Create a structured source summary in `sources/`.
- Create or update the corresponding wiki page in `wiki/{category}/`.
- Create or update a protein hub page in `wiki/proteins/{gene-or-protein}.md` when a paper belongs to a protein target already tracked in the wiki.
- Update `index.md`.
- Keep `pdf_path` pointing inside `papers/`.
- Keep `pdf_filename` equal to the basename of `pdf_path`.

## Mode B: Literature Location Mode

Use this mode when the user provides a protein, gene, target, pathway, disease, indication, or biological concept without a PDF.

Workflow:

```text
User query -> literature search -> ranked paper locations only
```

Rules:

- Do not create or update wiki pages.
- Do not create `sources/*.md`.
- Do not infer unsupported biological claims.
- Return paper locations and identifiers only.

For each paper, provide:

- Title
- Authors
- Year
- Journal
- PMID, DOI, or URL
- Source location, such as PubMed, PMC, internal repository, or local path if available
- Short reason why the paper appears relevant

If no relevant paper is found, say so clearly.

## Mode C: Internal Report Ingestion Mode

Use this mode when the user provides an internal report, AI-generated target review, feasibility report, drug design report, competitive assessment, or strategy document.

Workflow:

```text
Internal report -> reports/*.md -> sources/reports/*.md -> wiki/drug-design/*.md -> wiki/proteins/{target}.md -> index.md
```
Report input formats:

- Internal reports may arrive as `.md`, `.html`, or `.pdf`.
- Regardless of input format, the working source for Mode C is always a normalized markdown file under `reports/`.
- For `.md` reports, copy or clean the report directly into `reports/`.
- For `.html` reports, remove UI-only elements such as `style`, `script`, `noscript`, toolbar, navigation, search boxes, print controls, duplicate headings, and collapsed-section markers. Preserve scientific or strategic content, heading hierarchy, tables, lists, and section order, then write a cleaned markdown copy under `reports/`.
- For `.pdf` reports, read the PDF or extract text when useful, then convert the report content into cleaned markdown under `reports/`.
- Do not treat report PDFs as academic papers by default. Classify by content and user intent. A PDF report remains an internal report unless it is clearly an academic paper.
- Use the normalized markdown file as `original_report` for downstream `sources/reports/*.md` and `wiki/*.md` pages.
- Keep the raw original file path in metadata when useful, such as `original_pdf` or `original_html`.
- Report-derived claims follow the same evidence rules regardless of input format: mark them as `hypothesis` unless supported by ingested academic PDFs.




Rules:

- Archive the original report in `reports/`.
- Do not treat reports as papers.
- Do not put report-derived claims into normal paper source pages.
- Split the report into topic-specific source notes under `sources/reports/`.
- Create separate wiki pages for target biology, structural analysis, strategy pages, assay plans, ADMET/safety, competitive landscape, and literature backlog when useful.
- Every report-derived source note and wiki page must include `source_type` and `evidence_level` metadata.
- Mark report-derived scientific or clinical claims as hypotheses unless supported by ingested PDFs.
- Update the relevant protein hub with a clearly labeled report-derived section.
- Update `index.md`.

Suggested split for drug design reports:


| Report section | Source note | Wiki page |
|---|---|---|
| Target biology | `sources/reports/{target}-{source}-target-biology.md` | `wiki/drug-design/{target}-target-biology.md` or protein hub |
| Structural analysis | `sources/reports/{target}-{source}-structural-analysis.md` | `wiki/drug-design/{target}-structural-design-map.md` |
| Existing drugs | `sources/reports/{target}-{source}-existing-drugs.md` | `wiki/drug-design/{target}-existing-drug-landscape.md` |
| Small molecule strategy | `sources/reports/{target}-{source}-small-molecule-strategy.md` | `wiki/drug-design/{target}-ser273-spparm-strategy.md` |
| Covalent strategy | `sources/reports/{target}-{source}-covalent-strategy.md` | `wiki/drug-design/{target}-cys285-covalent-strategy.md` |
| PROTAC/degrader strategy | `sources/reports/{target}-{source}-protac-strategy.md` | `wiki/drug-design/{target}-galnac-protac-strategy.md` |
| Molecular glue strategy | `sources/reports/{target}-{source}-molecular-glue-strategy.md` | `wiki/drug-design/{target}-ncor-glue-strategy.md` |
| Research-tool biology | `sources/reports/{target}-{source}-research-tools.md` | `wiki/drug-design/{target}-research-tool-biology.md` |
| Assay cascade | `sources/reports/{target}-{source}-assay-cascade.md` | `wiki/drug-design/{target}-assay-cascade.md` |
| ADMET/safety | `sources/reports/{target}-{source}-admet-safety.md` | `wiki/drug-design/{target}-admet-safety-risk.md` |
| Competitive/IP | `sources/reports/{target}-{source}-competitive-ip.md` | `wiki/drug-design/{target}-competitive-ip.md` |
| Data gaps/literature backlog | `sources/reports/{target}-{source}-data-gaps.md` | `wiki/drug-design/{target}-data-gaps-and-literature-backlog.md` |
| Program roadmap | `sources/reports/{target}-{source}-roadmap.md` | `wiki/overviews/{target}-drug-design-roadmap.md` |

When a report cites papers that are not yet in `papers/`, list them as a literature backlog. Do not present those claims as paper-grounded until the PDFs are ingested.



## Adding a New Paper

1. Copy the PDF into `papers/`.
2. Extract text with Python 3:

```bash
python3 scripts/extract_pdf_text.py papers/{stem}.pdf --max-pages 15 --max-chars 12000
```

On this Windows workspace, `python` may point to Python 2. Use the bundled Python 3 runtime when needed:

```powershell
& "$env:USERPROFILE\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe" scripts\extract_pdf_text.py papers\{stem}.pdf --max-pages 15 --max-chars 12000
```

3. Write `sources/{stem}.md` using `templates/source-template.md`.
4. Write `wiki/{category}/{stem}.md` using `templates/wiki-page-template.md`.
5. Update `index.md` under the right category.

## Metadata Requirements

Every paper source and paper-derived wiki page should include YAML front matter with:

```yaml
---
title: "Paper Title"
authors: Author List
year: YYYY
doi: DOI
category: category-name
pdf_path: /full/path/to/papers/{stem}.pdf
pdf_filename: {stem}.pdf
source_collection: external
---
```

Wiki pages also include:

```yaml
source: {stem}.md
tags: []
```

Report source notes should include:

```yaml
---
title: "Report Section Title"
target: TARGET
protein: Protein Name
report_author: Report Author or System
report_date: YYYY-MM-DD
source_type: internal-report-derived
evidence_level: hypothesis
original_report: /full/path/to/reports/{stem}.md
report_section: "Section Name"
supported_by_ingested_papers: []
not_yet_ingested_references: []
tags: []
---
```

Report-derived wiki pages should include:

```yaml
---
title: "Strategy or Analysis Page Title"
target: TARGET
protein: Protein Name
category: drug-design
source_type: internal-report-derived
evidence_level: hypothesis
source: reports/{section-source}.md
original_report: /full/path/to/reports/{stem}.md
supported_by_ingested_papers: []
not_yet_ingested_references: []
tags: []
---
```

Use `evidence_level: mixed` only when the page explicitly separates paper-supported claims from report-derived hypotheses.

## Knowledge Compounding

The most valuable pages are `wiki/overviews/` pages that synthesize across papers already present in the wiki. When a question is answered well and the user asks to save it, create or update an overview page in `wiki/overviews/`.

Overview pages must cite only papers that exist in `sources/` and `wiki/`.

Report-derived overview pages may synthesize strategy, assay plans, and literature backlogs from internal reports, but they must clearly state that they are report-derived and hypothesis-level unless supported by ingested PDFs.

## Browsing With Obsidian

The user can open this folder as an Obsidian vault. The wiki uses plain Markdown and `[[wikilinks]]`, so Obsidian graph view and full-text search work without changing the repository.

When in doubt, follow the Core Rules.

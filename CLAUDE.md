# Protein Literature Wiki

A personal knowledge base of protein biology papers, following the LLM Wiki pattern:

```text
Original PDF -> sources/*.md (structured paper summary) -> wiki/{category}/*.md (final page)
```

Language policy: all wiki content is written in English. Conversation with the user can be in any language.

## Core Rules

These rules keep every claim traceable to papers that exist in this repository.

1. Do not use web search to answer scientific questions or fill gaps in wiki content.
2. Answer from `sources/` and `wiki/` first.
3. If the wiki is insufficient, re-read the matching PDF in `papers/`, extract more detail, and update the relevant source and wiki pages.
4. If the wiki has no paper on the topic, say: "I don't have a paper on this; please give me the PDF." Do not improvise.
5. Literature Location Mode is the only exception to rule 1. Use it only when the user asks to find papers or provides a protein, gene, target, pathway, disease, indication, or biological concept without PDFs. Return locations and identifiers only. Do not generate wiki claims.

These rules apply to every response, including overview pages. Cite only papers that exist in the wiki unless operating in Literature Location Mode.

## Repository Structure

```text
protein-wiki/
|-- CLAUDE.md
|-- index.md
|-- papers/
|   `-- {author}-{year}-{title-5-words}.pdf
|-- sources/
|   `-- {author}-{year}-{title-5-words}.md
|-- wiki/
|   |-- proteins/
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

Every source and wiki page should include YAML front matter with:

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

## Knowledge Compounding

The most valuable pages are `wiki/overviews/` pages that synthesize across papers already present in the wiki. When a question is answered well and the user asks to save it, create or update an overview page in `wiki/overviews/`.

Overview pages must cite only papers that exist in `sources/` and `wiki/`.

## Browsing With Obsidian

The user can open this folder as an Obsidian vault. The wiki uses plain Markdown and `[[wikilinks]]`, so Obsidian graph view and full-text search work without changing the repository.

When in doubt, follow the Core Rules.

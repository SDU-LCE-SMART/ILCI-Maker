# AI-Assisted Literature Review Engine (Intelligent LCI Maker)

## Project Overview
This repository contains an AI-assisted research workflow for literature review in the domain of fertilizers, life cycle assessment (LCA), and sustainability. The current system automatically searches major academic sources, merges and cleans records, filters them with rule-based and semantic methods, scores relevance, and exports a structured Excel sheet that includes both static metadata and dynamic AI-generated observations. The project is intentionally practical: it helps researchers move from broad query design to an analyzable shortlist of papers with less manual effort.

This project is still in the development stage and is not yet complete. The current phase (Phase 1) focuses on making literature discovery, ranking, and extraction more consistent and reproducible. The next phase (Phase 2) will be much larger in scope: an AI-assisted Life Cycle Inventory (LCI) maker that supports experts in constructing LCI datasets based on their specific needs. In that phase, the system will review public studies and published papers, learn patterns from existing inventories, and clearly cite source publications so domain experts can build new inventories with traceable evidence.

## Development Status
- Status: Active development
- Maturity: Prototype / research tooling
- Current phase: Phase 1 (AI-assisted literature review)
- Next phase: Phase 2 (AI-assisted LCI maker)

## Two-Phase Vision

### Phase 1 (Current): AI-Assisted Literature Review Tool
Phase 1 is designed to reduce repetitive screening tasks and increase consistency in paper evaluation. The tool:
- runs multi-source paper search from configurable research queries,
- performs deduplication and year filtering,
- computes keyword and semantic relevance,
- combines relevance with citation and recency signals,
- uses a local LLM to produce dynamic, paper-grounded observations,
- exports all results to a structured Excel workbook.

### Phase 2 (Planned): AI-Assisted Life Cycle Inventory (LCI) Maker
Phase 2 will expand from paper review into inventory construction support for experts. The goal is to help experts build fit-for-purpose LCIs by:
- reviewing current public studies and papers,
- extracting useful inventory patterns and assumptions,
- generating candidate LCI structures that are transparent and editable,
- providing explicit citations for each suggested component,
- preserving expert control while accelerating inventory drafting.

In short, Phase 2 is intended to move from "finding and understanding evidence" to "building citation-backed inventory assets" for real LCA workflows.

## Why This Architecture

### Why use external APIs?
The project uses multiple literature APIs because no single source provides complete coverage, metadata consistency, and availability for all topics.

- Scopus API: strong indexing and citation ecosystem for scholarly records.
- OpenAlex API: broad open catalog with useful metadata and transparent access model.
- Semantic Scholar API: additional coverage and citation-linked metadata.

Using multiple APIs improves recall, reduces source bias, and allows cross-source enrichment after deduplication.

### Why use a local LLM?
The local LLM layer is used to transform raw metadata into review-ready observations per dynamic headers (for example: methodology, key findings, boundaries, impact categories). A local model is useful in development because it enables:
- lower recurring cost per run,
- fast iteration on prompt and extraction rules,
- more control over model behavior,
- easier privacy management for local data processing.

The extraction behavior is configured to be paper-grounded: dynamic fields should be based on the exact forwarded paper record. If a field is outside scope or unsupported for a paper, the model can return "Not applicable".

## Current Pipeline
1. Read user configuration from `config/user_config.yaml`.
2. Expand research queries for better retrieval coverage.
3. Search APIs (Scopus, OpenAlex, Semantic Scholar).
4. Normalize and merge records.
5. Remove duplicates (DOI and high-similarity title checks).
6. Apply year and keyword-based initial filtering.
7. Compute semantic similarity against research description.
8. Score each paper using weighted metrics.
9. Run local LLM extraction for dynamic analysis fields.
10. Export final ranked dataset to Excel.

## Repository Structure

```text
LR/
  config/
    user_config.yaml
  engine/
    engine.py
    search/
      search.py
  main.py
  requirements.txt
  README.md
```

## Core Components

### `main.py`
- Loads environment variables.
- Loads YAML config.
- Instantiates and runs `LiteratureEngine`.

### `engine/search/search.py`
- Handles API integration and retries.
- Manages query execution per source.
- Normalizes source-specific metadata into a common record shape.

### `engine/engine.py`
- Query expansion, filtering, deduplication.
- Semantic ranking with Sentence Transformers.
- Weighted scoring (keyword, semantic, citation, recency).
- Local LLM prompt construction and dynamic field extraction.
- Excel export formatting and post-processing.

### `config/user_config.yaml`
- Research description and query list.
- Year window and keyword constraints.
- Limits for each pipeline stage.
- Scoring weights.
- Local LLM settings.
- Static and dynamic export headers.

## Configuration Highlights

### Research & query scope
You define domain intent in:
- `research.description`
- `research.queries`
- `research.year_min` / `research.year_max`

### Filtering and limits
You control throughput using:
- `limits.max_initial_results_per_source`
- `limits.max_after_initial_filtering`
- `limits.max_after_filtering`
- `limits.max_for_local_llm`

### Scoring model
`scoring_weights` controls ranking behavior:
- `keyword_match`
- `semantic_score`
- `citation_score`
- `recency_score`

### Dynamic extraction behavior
`local_llm` controls model execution and handling of non-applicable fields:
- `enabled`
- `model`
- `url`
- `apply_to_all_export_rows`
- `not_applicable_text`

## Installation

### Prerequisites
- Python 3.9+
- Access to one or more literature APIs
- Local LLM server endpoint (for example, Ollama-compatible endpoint)

### Install dependencies
```bash
pip install -r requirements.txt
```

### Environment variables
Create a `.env` file in the project root with only the keys you plan to use:

```env
SCOPUS_API_KEY=...
SEMANTIC_SCHOLAR_API_KEY=...
OPENALEX_EMAIL=your_email@example.com
```

Do not commit real API keys to version control.

## Usage
Run from the project root:

```bash
python main.py
```

After completion, the tool writes an Excel output file (configured by `excel_export.filename`, currently `literature_results.xlsx`).

## Output
The exported workbook combines:
- static columns (title, year, journal, DOI, source, citations, final score, document type),
- dynamic columns generated from local LLM observations,
- formatting improvements such as wrapped text, header styling, and adjusted column widths.

## Current Limitations (Expected in Development)
- API metadata quality varies across providers.
- Some records contain incomplete abstracts or missing fields.
- Dynamic field quality depends on local model capability and prompt tuning.
- Domain coverage is currently tuned to fertilizer/LCA research scope.
- No GUI is included yet (CLI workflow only).

## Planned GUI Direction
A future GUI is planned to make the workflow easier and more feasible for non-technical users. The interface is expected to support:
- visual query and filter builder,
- one-click pipeline execution,
- interactive review/edit of dynamic fields,
- citation traceability views,
- export presets for literature review and future LCI workflows.

## Roadmap
1. Stabilize Phase 1 extraction quality and error handling.
2. Add richer evaluation and traceability for dynamic fields.
3. Introduce GUI for improved usability and adoption.
4. Start Phase 2 design and prototype for AI-assisted LCI maker.
5. Build citation-aware LCI drafting workflow from public studies.

## Contribution
This project is in active development and architecture may evolve. Contributions are welcome in:
- retrieval quality,
- ranking and scoring strategies,
- prompt and extraction reliability,
- export/report usability,
- GUI and Phase 2 planning.

## Disclaimer
This tool is designed to assist experts, not replace expert judgment. All extracted insights should be reviewed before use in formal analysis or publication.

# AI-Assisted Literature Review Engine (Intelligent LCI Maker)

## Project Overview
This project is a working prototype which already performs end-to-end literature review automation: it accepts configurable research queries, collects records from multiple scholarly APIs, removes duplicates, applies filters, ranks papers with combined scoring logic, runs local LLM extraction for dynamic analysis fields, and exports structured outputs to Excel. The current fertilizer-focused setup is used as a sample prototyping scenario to validate the pipeline against real-world LCA-style search and extraction complexity.

The project is still in the improvement stage and is intentionally evolving. The architecture is domain-flexible, which means the same engine can be used for any topic, title, and study field by changing configuration (queries, keywords, year range, scoring weights, and dynamic headers). Phase 1 is the active implementation today (AI-assisted literature review). Phase 2 will expand this into a larger AI-assisted LCI maker that helps experts draft inventory structures by learning from published studies and linking each suggested element to explicit source evidence.

## Development Status
- Status: Active development
- Delivery stage: Prototype implemented, currently being improved
- Current phase: Phase 1 (AI-assisted literature review engine)
- Next phase: Phase 2 (AI-assisted, citation-grounded LCI maker)

## Evidence That the Prototype Is Implemented
The repository includes executable code paths, not placeholder stubs:

- Pipeline entrypoint in `main.py` that loads environment/config and executes the engine.
- Retrieval engine in `engine/search/search.py` with real API requests, retry logic, pagination, and source-specific metadata mapping.
- Processing and ranking engine in `engine/engine.py` for query expansion, year filtering, duplicate removal, keyword filtering, semantic ranking, and weighted scoring.
- Local LLM extraction in `engine/engine.py` to fill dynamic review fields using paper-grounded prompts.
- Export pipeline in `engine/engine.py` that writes formatted Excel output using configured static and dynamic headers.
- User-driven behavior in `config/user_config.yaml` so study setup can be changed without code edits.

This means the system is already usable for iterative research work while engineering improvements continue.

## Why Fertilizer Is Mentioned Today
Fertilizer/LCA is currently a sample prototype scenario used to test difficult conditions (mixed terminology, domain-specific keywords, and structured extraction needs). It is not a hard limit.

To switch domains, update configuration only:
- `research.description`
- `research.queries`
- `keywords.must_include` / `keywords.optional` / `keywords.exclude`
- `excel_export.dynamic_headers`
- `scoring_weights`

No major refactor is required for topic changes.

## Two-Phase Vision

### Phase 1 (Current): AI-Assisted Literature Review Tool
Implemented focus:
- configurable multi-source paper retrieval,
- deduplication and quality filtering,
- semantic ranking + weighted scoring,
- dynamic field extraction with local LLM,
- analyst-friendly Excel export.

Current improvements in progress:
- stronger extraction consistency,
- better handling for sparse metadata,
- tuning prompt behavior per dynamic header,
- improving reproducibility and observability.

### Phase 2 (Planned): Intelligent LCI Maker
Planned expansion:
- assist experts in drafting fit-for-purpose LCI structures,
- derive inventory inspirations from public studies,
- keep every suggested inventory element traceable to cited evidence,
- support expert-in-the-loop validation before finalization.

## Technical Architecture

### Core Modules
- `main.py`: app bootstrap and run control.
- `engine/search/search.py`: API retrieval layer (Scopus, OpenAlex, Semantic Scholar).
- `engine/engine.py`: processing, ranking, local LLM extraction, Excel export.
- `config/user_config.yaml`: runtime behavior and study-specific setup.

### Pipeline Flow
1. Load runtime config and env variables.
2. Expand each user query to improve retrieval recall.
3. Fetch candidate records from available APIs.
4. Normalize records and merge sources.
5. Remove duplicates by DOI and title similarity.
6. Apply year + keyword filtering.
7. Compute semantic similarity using embedding model.
8. Score records with weighted criteria (keyword/semantic/citation/recency).
9. Run local LLM extraction for dynamic headers.
10. Export to Excel with configured columns and formatting.

## Why We Use APIs and Local LLM Together

### API Layer Rationale
No single source gives complete literature coverage and consistent metadata quality. Multi-source retrieval increases recall and reduces source bias.

- Scopus: strong curated indexing and citation context.
- OpenAlex: broad open access catalog with flexible metadata access.
- Semantic Scholar: complementary metadata and citation coverage.

### Local LLM Rationale
The local LLM transforms raw metadata into analyst-ready dynamic fields. It is useful for prototyping and iterative development because it provides:
- lower recurring cost during rapid testing,
- control over prompt behavior and output style,
- easier privacy control in local workflows,
- fast iteration when refining extraction headers.

Dynamic extraction is paper-grounded by design. If a field is out of scope for a paper, output can be set to `Not applicable`.

## How To Create Effective Queries
Use boolean queries that combine:
- concept block A (method or framework),
- concept block B (topic object),
- optional context block C (impact/sustainability/region/time).

### Query Design Pattern
```text
("concept A synonym 1" OR "concept A synonym 2" OR ACRONYM)
AND
("concept B synonym 1" OR "concept B synonym 2")
AND
("context term 1" OR "context term 2")
```

### Example (Current Prototype Topic)
```text
("life cycle assessment" OR "life-cycle assessment" OR "life cycle analysis" OR LCA)
AND
("controlled release fertilizer" OR "slow release fertilizer" OR CRF OR SRF)
AND
("sustainability" OR "environmental impact" OR "GHG emissions")
```

### Example (Different Future Topic)
```text
("life cycle assessment" OR LCA)
AND
("battery recycling" OR "end-of-life battery" OR "lithium-ion recycling")
AND
("resource recovery" OR "emissions" OR "circular economy")
```

## How To Define Keywords and Excel Dynamic Headers

### 1. Keyword Strategy
Use `keywords` to shape filtering behavior before semantic ranking:

- `must_include`: high-priority terms that should appear.
- `optional`: supporting terms that improve ranking but are not mandatory.
- `exclude`: terms to remove obvious out-of-scope papers.

Example:
```yaml
keywords:
  must_include: ["life cycle", "fertilizer"]
  optional: ["slow release", "controlled release", "biochar"]
  exclude: ["policy", "editorial"]
```

For another domain, replace these terms only; pipeline logic stays the same.

### 2. Dynamic Header Strategy (Excel)
Use `excel_export.dynamic_headers` to tell the LLM what analysis fields to produce per paper.

Example:
```yaml
excel_export:
  dynamic_headers:
    - key: summary
      header: Summary
    - key: methodology
      header: Methodology
    - key: key_findings
      header: Key Findings
    - key: system_boundaries
      header: System Boundaries
    - key: impact_categories
      header: Impact Categories
```

Guidance:
- keep header keys short and stable (snake_case),
- use human-friendly `header` labels for Excel output,
- avoid redundant fields that ask the same question in different words,
- keep dynamic fields aligned with your research objective,
- allow `Not applicable` for out-of-scope fields.

## Configuration Walkthrough

### Minimum Required Blocks
`config/user_config.yaml` should define:
- `research`
- `keywords`
- `limits`
- `scoring_weights`
- `local_llm`
- `excel_export`

### Local LLM Block Example
```yaml
local_llm:
  enabled: true
  model: "llama3"
  url: "http://localhost:11434/api/generate"
  apply_to_all_export_rows: true
  not_applicable_text: "Not applicable"
```

## Setup and Run

### Prerequisites
- Python 3.9+
- API keys for at least one source (Scopus optional, Semantic Scholar optional, OpenAlex works with/without email)
- Local model endpoint for dynamic extraction

### Install
```bash
pip install -r requirements.txt
```

### Environment Variables
Create `.env` in project root:

```env
SCOPUS_API_KEY=...
SEMANTIC_SCHOLAR_API_KEY=...
OPENALEX_EMAIL=your_email@example.com
```

### Execute
```bash
python main.py
```

Output file is controlled by `excel_export.filename` (default in this prototype: `literature_results.xlsx`).

## Output Model
The Excel export combines:
- static metadata columns (title, year, source, DOI, citations, score, and related fields),
- dynamic LLM-generated analysis columns,
- formatting improvements for readability (header style, wrapping, width handling).

## Improvement Backlog (Current Step)
- strengthen handling for missing abstracts and sparse records,
- improve dynamic header instruction quality by field type,
- add evaluation metrics for extraction consistency,
- improve explainability for final score composition,
- prepare GUI layer for non-technical users.

## Planned GUI
The planned GUI will make study setup and review easier:
- guided query builder,
- keyword and header designer,
- one-click run and progress indicators,
- review/edit panel for dynamic extraction,
- evidence and citation traceability views,
- export profiles for different study types.

## Future Direction: Any Topic, Any Study Title
This engine is designed for reuse across research fields. Fertilizer is a prototype case. The same workflow can support health, energy, materials, policy, circular economy, and other domains by changing configuration inputs while preserving the core pipeline.

## Contribution
This project is active and evolving. Contributions are welcome for:
- retrieval quality,
- scoring/ranking strategy,
- dynamic extraction reliability,
- observability and evaluation,
- GUI implementation,
- Phase 2 LCI design.

## Disclaimer
This tool assists researchers and experts; it does not replace expert judgment. Outputs should be reviewed before publication, compliance reporting, or decision-making.

# Automated Document Metadata Pipeline

A semi-automated pipeline that converts heterogeneous documents into normalized text, classifies them into two domains, enriches them with locally extracted and LLM-generated metadata, and stores the results in a structured SQLite database for search, retrieval, and analysis.

## Document Classes

| Class | Domain | Examples |
|-------|--------|----------|
| **A** | Business | Ideas, proposals, client briefings, articles, post drafts, statistics, reports |
| **B** | Creative Writing | Story ideas, outlines, chapters, drafts, background material, critical analyses, meta-writing |

Both classes may contain chatbot dialogue transcripts (classified by subject matter, flagged with `is_transcript`).

## Pipeline Stages

1. **Intake & Inventory** — Recursive scan, content hashing (SHA-256), deduplication, change detection
2. **Conversion** — Convert to `.md` (fallback `.txt`) via Pandoc / PyMuPDF. Originals are never modified; converted files live in a parallel tree
3. **Manual Classification** — Human sorts files into `Documents/A/` or `Documents/B/`. Unsorted files remain in `_inbox/`
4. **Local Metadata Extraction** — Filename, title, author, timestamps, file size, content hash, word count, language
5. **LLM Enrichment** — Structured JSON via API: summary, notes, classification confidence, relevance/quality ratings, topics, entities, document type, processing status
6. **Persistence** — JSON sidecars (raw truth) + SQLite database. LLM metadata stored separately to allow re-runs without losing history

## Key Design Principles

- **Idempotent & resumable:** Unchanged files are skipped; crashed runs resume cleanly
- **Non-destructive:** Originals are never modified or moved by the pipeline
- **Auditable:** Processing logs, model IDs, prompt versions, and content hashes tracked throughout
- **Error-isolated:** Per-file failures never abort the batch

## CLI

pipeline scan | convert | extract | enrich | load | report | run-all

Supports `--dry-run`, `--force` (reprocess all), and per-run budget caps.

## Open Questions

1. Privacy policy for documents sent to external LLM APIs
2. Chunking strategy for long documents (truncate / chunk-and-merge / head+tail)
3. Concrete rating rubrics for relevance and quality per class
4. Whether to add a class "C/other" or hard-reject documents fitting neither A nor B

## Status

Pre-development. See the project brief for full specifications.

## CLI


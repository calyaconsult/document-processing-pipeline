# Project Brief: Automated Document Metadata Pipeline

## 1. Overview

A semi-automated pipeline that converts heterogeneous documents into a normalized text format, classifies them into two domains (manually), enriches them with locally extracted and LLM-generated metadata, and stores the results in a structured database for search, retrieval, and analysis.

**Domains:**
- **Class A — Business:** business ideas, proposals, client briefings, downloaded articles, business post drafts, statistics, reports
- **Class B — Creative Writing:** story ideas, outlines, chapters, drafts, background material, critical analyses, meta-writing

Both classes may contain chatbot dialogue transcripts.

---

## 2. Goals & Non-Goals

**Goals**
- Every document in `Documents/` ends up with one normalized text version and one complete metadata record.
- Metadata is machine-queryable (database), reproducible (content hashes), and auditable (processing logs, model IDs).
- The pipeline is re-runnable: already-processed, unchanged files are skipped; changed files are reprocessed.

**Non-Goals (v1)**
- Automatic A/B classification (manual only; the LLM provides a *confidence check*, not the decision).
- OCR of scanned images / handwriting (flag as unprocessable instead — can be a v2 feature).
- Full-text search UI (database-level queries suffice initially).

---

## 3. Pipeline Stages

### Stage 0 — Intake & Inventory *(addition)*
- Scan `Documents/` recursively.
- Build/refresh an inventory table: path, size, mtime, content hash (e.g., SHA-256).
- **Deduplication:** identical hashes are recorded once; duplicates are logged and linked, not reprocessed.
- **Change detection:** hash comparison against the database decides skip / process / reprocess.

### Stage 1 — Conversion to Markdown/Text
- Target format: `.md`; fallback `.txt` if structure can't be preserved.
- Supported source formats (define explicitly): `.docx`, `.odt`, `.pdf` (text-based), `.rtf`, `.html`, `.epub`, `.txt`, `.md`.
- Suggested tooling: Pandoc as workhorse; `pdftotext`/PyMuPDF or MarkItDown for PDFs.
- **Originals are never modified.** Converted files live in a parallel tree (e.g., `Converted/A/...`) or alongside with a suffix — decide once, apply consistently.
- Conversion failures → status `conversion_failed`, file listed in an exceptions report for manual handling.
- *(Addition)* Record conversion tool + version in metadata for reproducibility.

### Stage 2 — Manual Classification
- Human moves files into `Documents/A/` or `Documents/B/`.
- *(Addition)* Add `Documents/_inbox/` for unclassified new arrivals and `Documents/_rejected/` for out-of-scope files. The pipeline only processes A and B; anything left in `_inbox` appears in a "pending classification" report.
- *(Addition)* Chatbot transcripts: classification follows the transcript's *subject matter*, not its format. The LLM should additionally flag `is_transcript: true` so these can be filtered later.

### Stage 3 — Local Metadata Extraction
Per file:
| Field | Source |
|---|---|
| filename | filesystem |
| relative path & class (A/B) | filesystem |
| document title | embedded metadata if available, else first heading, else filename |
| author | embedded metadata; fallback: configurable default or null |
| last modified | filesystem mtime (note: unreliable after copies — record both mtime and, if available, embedded creation date) |
| file size | filesystem |
| content hash | SHA-256 of original **and** of converted text *(addition — the converted-text hash detects semantically identical duplicates in different formats)* |
| word/character count *(addition)* | converted text |
| language *(addition)* | detected locally or by LLM |

### Stage 4 — LLM Enrichment
- Send converted text via API; receive **structured JSON** (use JSON mode / structured output if the API supports it).
- **Chunking policy** *(addition)*: define max input length; long documents are either truncated with a marker, summarized per chunk and merged, or sent as head+tail sample. Record which strategy was used.
- **Prompt versioning** *(addition)*: store `prompt_version` with every record so results are comparable after prompt changes.

**LLM output schema:**
```json
{
  "model_id": "string (from API response, not the prompt)",
  "summary": "string, 2–5 sentences",
  "notes": "string, free-form observations",
  "class_confidence": {"label": "A|B", "confidence": 0.0-1.0},
  "relevance_rating": "1-5",
  "quality_rating": "1-5",
  "topics": ["string"],
  "entities": [{"name": "string", "type": "person|org|place|product|character|work|other"}],
  "is_transcript": "boolean",
  "doc_type": "string (proposal|report|article|chapter|outline|transcript|...)",
  "language": "ISO 639-1",
  "processing_status": "ok|partial|failed"
}
```
*(Additions vs. your list: `is_transcript`, `doc_type`, `language`, typed entities.)*

- **Validation** *(addition)*: validate JSON against a schema (e.g., Pydantic/JSON Schema). Invalid responses → up to N retries with error feedback → then status `llm_failed`.
- **Rating rubrics** *(addition)*: define what relevance and quality mean per class (e.g., relevance = usefulness to your current business / current writing projects; quality = completeness, coherence, polish). Put the rubric in the prompt, otherwise ratings drift.
- **Disagreement flag** *(addition)*: if the LLM's suggested class contradicts the manual folder placement with confidence > threshold (e.g., 0.8), flag for human review — this catches misfiled documents.
- **Privacy note** *(addition)*: client briefings and personal creative work go to a third-party API. Decide: acceptable? Redaction step? Local model for sensitive subsets? Document the decision.

### Stage 5 — Persistence
- Write per-file JSON sidecars (raw truth) **and** load into a database.
- **SQLite recommended** for a single-user local pipeline (skip the CSV intermediate unless you need it for spreadsheets — if so, generate CSV *from* the DB, not as the pipeline's source of truth).

**Suggested schema (simplified):**
```sql
documents(
  id, original_path, converted_path, class TEXT CHECK(class IN ('A','B')),
  filename, title, author, mtime, created_date, file_size,
  hash_original, hash_converted, word_count, language,
  doc_type, is_transcript BOOLEAN, is_duplicate_of INTEGER NULL
);
llm_metadata(
  id, document_id, model_id, prompt_version, run_timestamp,
  summary, notes, class_confidence_label, class_confidence_value,
  relevance INTEGER, quality INTEGER, chunking_strategy,
  processing_status
);
topics(document_id, topic);         -- normalized 1:n
entities(document_id, name, type);  -- normalized 1:n
processing_log(id, document_id, stage, status, message, timestamp);
```
*(Addition: `llm_metadata` as a separate table with timestamps allows re-runs with newer models without losing history.)*

---

## 4. Operational Concerns *(all additions)*

- **Idempotency & resume:** every stage checks the DB before working; a crashed run resumes cleanly.
- **Error handling:** per-file failures never abort the batch; end-of-run exceptions report lists all failures by stage.
- **Logging:** structured log per run (files seen, skipped, processed, failed, API cost/tokens).
- **Cost control:** token counting before submission, per-run budget cap, dry-run mode.
- **Rate limiting & retries:** exponential backoff on API errors.
- **Configuration file:** paths, API keys (via env vars, not config), model choice, thresholds, prompt version — no hardcoding.
- **Backup:** DB and sidecar JSONs are backed up; originals remain untouched.

---

## 5. Deliverables

1. CLI tool with subcommands: `scan`, `convert`, `extract`, `enrich`, `load`, `report`, `run-all`
2. Config file template + prompt template(s) with rubrics
3. SQLite schema + migration script
4. Reports: pending classification, exceptions, classification disagreements, duplicates
5. README with re-run and recovery procedures

## 6. Open Questions (decide before build)

1. Converted files: parallel tree or in-place? (Recommendation: parallel tree.)
2. Privacy: which documents may leave the machine? Redaction or local model needed?
3. Long documents: truncate, chunk-and-merge, or head+tail? Max token budget?
4. Rating rubrics: concrete definitions for relevance/quality per class.
5. Is CSV export actually needed, or is SQLite + ad-hoc export enough?
6. Should re-runs with a new model *replace* or *append* LLM metadata? (Recommendation: append.)
7. What happens to files that fit neither A nor B — hard reject, or add class "C/other"?

*Prompt:* 

I'm considering setting up a (mostly) automated document processing pipeline that will create a set of metadata for each file in a folder named "Documents". The pipeline will have the following steps:
- convert documents to markdown (.md) or, if that's not possible, to plain text (.txt)
- manually classify as "A" or "B" by storing the documents in directory "Documents/A" or directory "Documents/B"
- gather file metadata locally: filename, document title, author, last modification time, file size, content hash
- send each file via API to an LLM and receive JSON metadata: model ID, summary, notes, A/B classification confidence, relevance rating, quality rating, list of topics, list of entities, processing status
- convert JSON to CSV/SQL for database insertion

*Response:*

In my specific case class "A" will be business-related documents, including but not confined to: business ideas, proposals, client briefings, downloaded articles, drafts for business-related posts, statistics, reports.
Class "B" will be creative writing-related documents: story ideas, outlines, single chapters, drafts, background material, critical analyses, meta-writing (writing about writing)
Either class may contain transcrippts of chatbot dialogues.
This is a sensible architecture. Here's feedback on each stage plus some design decisions worth settling before you build.

## Pipeline order and metadata extraction

**Extract embedded metadata *before* conversion.** Title and author live in document properties (DOCX, PDF, ODT). Once you convert to `.md`/`.txt`, that's gone. So the metadata-gathering step should run against the *original* file, even though the LLM step runs on the converted text.

**Decide what the content hash covers.** Hashing the original file is best for deduplication and idempotency ("have I processed this exact file before?"). Hashing the converted text detects content-identical documents in different formats. You may want both — original hash as the primary key for pipeline state, text hash for dedup.

## Conversion

- **Tools:** `pandoc` covers most formats; Microsoft's `markitdown` or IBM's `docling` handle PDFs/Office files well. For scanned PDFs you'll need OCR (e.g., `ocrmypdf`) — worth a check upfront (PDFs with no extractable text should be flagged, not silently converted to empty files).
- Record which converter and fallback path was used in your metadata — useful for debugging bad summaries later.
- Tables, footnotes, and images degrade in conversion; for statistics/reports (your class A) this can matter for summary quality.

## The LLM step

This is where most of the subtle design decisions live:

1. **Human label vs. LLM classification.** You classify manually via directory, then ask the LLM for "A/B classification confidence." Decide the semantics: is the LLM *told* the label and asked how confident it is, or does it classify *blind* and you compare? The blind approach is more useful — disagreements flag mislabeled files. Store both: `human_label`, `llm_label`, `llm_confidence`.

2. **Define your rating scales.** "Relevance" and "quality" are meaningless without a rubric. Relevant *to what*? Quality by what criteria (a rough story outline can be excellent *as an outline*)? Put explicit rubrics and scale anchors (e.g., 1–5 with descriptions) in the prompt, or ratings will be noisy and incomparable across runs.

3. **Use structured output.** Most APIs support JSON schema enforcement (OpenAI structured outputs, Anthropic tool use, etc.). Don't parse freeform JSON out of prose — you'll get malformed output at scale.

4. **Long documents.** Chatbot transcripts and reports can exceed context comfortably. Decide on truncation, chunk-and-merge summarization, or a long-context model. Record which strategy was applied per file.

5. **Determinism and cost.** Temperature 0 (or low), record model ID *and* prompt version — if you refine the prompt later, you'll want to know which files were processed under which regime. Consider `prompt_version` as an explicit field.

6. **Topics/entities:** freeform lists will drift ("AI", "artificial intelligence", "A.I."). Either provide a controlled vocabulary in the prompt, or plan a normalization pass afterward.

7. **Privacy:** business proposals and client briefings going to a third-party API is a real consideration. Check the provider's data retention/training policy, or run a local model (Ollama etc.) if the content is sensitive — quality of summaries will be lower but may suffice.

## Storage

Skip CSV as an intermediate if you can — go straight to **SQLite**. It's zero-setup, handles incremental updates, and avoids CSV escaping pain with multi-line summaries. Suggested shape:

```sql
CREATE TABLE documents (
    id INTEGER PRIMARY KEY,
    original_hash TEXT UNIQUE,
    text_hash TEXT,
    filename TEXT,
    filepath TEXT,
    title TEXT,
    author TEXT,
    modified_at TEXT,
    file_size INTEGER,
    human_label TEXT CHECK(human_label IN ('A','B')),
    converted_format TEXT,          -- 'md' or 'txt'
    processing_status TEXT,         -- 'pending','converted','analyzed','failed'
    error_message TEXT
);

CREATE TABLE llm_analysis (
    document_id INTEGER REFERENCES documents(id),
    model_id TEXT,
    prompt_version TEXT,
    summary TEXT,
    notes TEXT,
    llm_label TEXT,
    label_confidence REAL,
    relevance INTEGER,
    quality INTEGER,
    raw_json TEXT,                  -- keep the full response
    analyzed_at TEXT
);

CREATE TABLE topics (document_id INTEGER, topic TEXT);
CREATE TABLE entities (document_id INTEGER, entity TEXT, entity_type TEXT);
```

Keeping `raw_json` means you can re-derive tables if your schema evolves without re-paying API costs.

## Operational points

- **Idempotency:** on each run, skip files whose hash is already in the DB with status `analyzed`. This makes the pipeline safely re-runnable and handles renames/moves gracefully.
- **Failure handling:** conversion failures and API failures should set `processing_status = 'failed'` with an error message, not halt the pipeline. A retry pass then targets only failed rows.
- **Validation sample:** after the first ~50 documents, manually review summaries and ratings before running the full corpus. Prompt problems are much cheaper to fix early.

One question worth answering for yourself: what are the downstream queries you actually want to run? ("Find all high-quality unposted draft posts," "find story chapters mentioning character X"?) That answer should drive whether topics/entities need controlled vocabularies and whether you also want embeddings for semantic search — which would be a natural extension since you're already sending full text to an API.

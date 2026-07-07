# 📄 Automated Document Metadata Pipeline

A semi-automated pipeline that converts heterogeneous documents into normalized text, classifies them into two domains, enriches them with locally extracted and LLM-generated metadata, and stores the results in a structured database for search, retrieval, and analysis.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Document Classes](#document-classes)
- [Pipeline Stages](#pipeline-stages)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Database Schema](#database-schema)
- [Reports](#reports)
- [Operational Details](#operational-details)
- [Open Questions & Decisions](#open-questions--decisions)
- [License](#license)

---

## Overview

This tool processes a folder of documents (`Documents/`) through a multi-stage pipeline:

1. **Scan** the folder and build an inventory
2. **Convert** documents to Markdown or plain text
3. **Classify** manually into domain A or B
4. **Extract** local file metadata
5. **Enrich** via LLM API to generate structured metadata (summary, topics, entities, ratings, etc.)
6. **Persist** all metadata to a SQLite database (with JSON sidecars as raw truth)

The pipeline is **idempotent and resumable** — unchanged files are skipped, failed files are logged without aborting the batch, and crashed runs resume cleanly.

---

## Features

- ✅ Recursive folder scanning with change detection (SHA-256 content hashes)
- ✅ Deduplication (identical files processed once; duplicates logged and linked)
- ✅ Format conversion via Pandoc / PyMuPDF / MarkItDown
- ✅ Local metadata extraction (title, author, timestamps, word count, language)
- ✅ LLM enrichment with structured JSON output (summary, topics, entities, ratings)
- ✅ Classification confidence checking (LLM flags potential misfilings)
- ✅ Prompt versioning and model tracking for reproducibility
- ✅ Structured logging, cost control, and dry-run mode
- ✅ SQLite persistence with full run history (re-runs append, not replace)

---

## Document Classes

| Class | Label | Contents |
|-------|-------|----------|
| **A** | Business | Business ideas, proposals, client briefings, downloaded articles, business post drafts, statistics, reports |
| **B** | Creative Writing | Story ideas, outlines, chapters, drafts, background material, critical analyses, meta-writing |

> **Note:** Both classes may contain chatbot dialogue transcripts. Classification follows the transcript's *subject matter*, not its format. Transcripts are additionally flagged with `is_transcript: true`.

---

## Pipeline Stages

### Stage 0 — Intake & Inventory

- Scans `Documents/` recursively
- Builds/refreshes an inventory table: path, size, mtime, content hash (SHA-256)
- **Deduplication:** identical hashes are recorded once; duplicates are logged and linked
- **Change detection:** hash comparison against the database decides skip / process / reprocess

### Stage 1 — Conversion

- **Target format:** `.md` (fallback `.txt` if structure can't be preserved)
- **Supported source formats:** `.docx`, `.odt`, `.pdf` (text-based), `.rtf`, `.html`, `.epub`, `.txt`, `.md`
- **Tooling:** Pandoc (primary); `pdftotext` / PyMuPDF / MarkItDown for PDFs
- Originals are **never modified** — converted files live in a parallel tree (`Converted/A/`, `Converted/B/`)
- Conversion tool + version recorded in metadata for reproducibility
- Failures → status `conversion_failed`, listed in exceptions report

### Stage 2 — Manual Classification

Human places files into the appropriate directory:


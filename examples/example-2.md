*Prompt:*

Use the following text as sample of class "B" and provide a realistic set of metadata:
```
Henri hielt den Atem an, so weit es der Helm seines alten Tauchanzugs zuließ. Die Kugel vor ihm lag so reglos da, und doch schien sie lebendig. Ihre Oberfläche war glatt wie Glas, aber im Inneren bewegte sich etwas – ein Schimmern, das Formen andeutete, die sich sofort wieder auflösten, als wolle es keine klare Gestalt annehmen. Er streckte vorsichtig die behandschuhte Hand aus, doch noch bevor er sie berührte, vibrierte das Wasser um ihn herum. Die Fische zogen sich zurück, die Korallen wirkten plötzlich starr, als hielte die gesamte Unterwasserwelt den Atem an. In seinem Kopf tauchte ein Klang auf – kein Geräusch, sondern eine Resonanz, ein Gefühl, das nicht durch die Ohren, sondern direkt durch seine Gedanken ging. Eine fremde Stimme ohne Worte. Sie sprach nicht in einer Sprache, und doch verstand Henri: *„Wir beobachten. Wir warten. Du bist der Erste, der uns sieht.“* Ein Schauer lief ihm über den Rücken. Hatte er den Verstand verloren? Oder war dies die Begegnung, nach der die Menschheit seit Jahrhunderten suchte? Die Kugel pulsierte stärker, und Henri spürte, wie seine eigenen Gedanken mitgerissen wurden. Bilder blitzten in ihm auf – gewaltige Städte in den Tiefen, gebaut von Wesen, die nicht menschlich waren, und doch eine fremdartige Intelligenz ausstrahlten. Zeit schien für sie keine Rolle zu spielen. Sie waren schon hier, lange bevor Menschen das Meer betraten. Henri wusste plötzlich: Die Kugel war kein Artefakt. Sie war ein Auge. Ein Auge, das in die andere Richtung blickte – und nun hatte es ihn gesehen.
```

*Response*

Here's a realistic metadata set for your sample document, following your pipeline structure:

## Step 1–3: Local file metadata

```json
{
  "filename": "henri_kugel_begegnung.md",
  "filepath": "Documents/B/henri_kugel_begegnung.md",
  "title": "Henri und die Kugel",
  "author": "unknown",
  "last_modified": "2025-05-14T21:37:12+02:00",
  "file_size_bytes": 1994,
  "content_hash": "sha256:7f3a9c2e51b8d4f6a0e19c73d5842b6f1c0e8a94d723f5b18e6c4a029d7e3f51"
}
```

**Notes on this step:**
- The hash above is illustrative — your pipeline would compute the real SHA-256.
- `author` is a common weak point: markdown/plain text files carry no embedded author metadata (unlike .docx/.pdf). You'll likely need a fallback rule (filesystem owner, a default value, or a YAML front-matter convention in your .md files).
- Same for `title`: a sensible heuristic is "first H1 heading, else filename." This sample has no heading, so a human or the LLM must supply it — worth deciding which field is authoritative.

## Step 4: LLM-generated metadata

```json
{
  "model_id": "claude-sonnet-4-20250514",
  "summary": "Ein Taucher namens Henri entdeckt auf dem Meeresgrund eine rätselhafte Kugel, die sich als telepathisches Beobachtungsorgan einer uralten, nicht-menschlichen Unterwasser-Zivilisation erweist. Die Erzählung endet mit der Erkenntnis, dass die Kugel ein 'Auge' ist – und Henri nun gesehen wurde.",
  "notes": "Kurze in-sich geschlossene Szene, vermutlich Auftakt oder Fragment einer längeren Erzählung. Genre: Science-Fiction/Unterwasser-Mystery. Personale Erzählperspektive (3. Person, Henri). Offenes Ende mit Cliffhanger-Charakter. Kein Titel im Dokument vorhanden.",
  "language": "de",
  "classification": "B",
  "classification_confidence": 0.98,
  "relevance_rating": 4,
  "quality_rating": 4,
  "topics": [
    "science fiction",
    "first contact",
    "underwater exploration",
    "telepathy",
    "ancient civilization",
    "mystery"
  ],
  "entities": [
    {"name": "Henri", "type": "character"},
    {"name": "die Kugel", "type": "object"},
    {"name": "Unterwasser-Zivilisation", "type": "collective"}
  ],
  "processing_status": "success"
}
```

## Step 5: Flattened CSV row

```csv
filename,filepath,title,author,last_modified,file_size_bytes,content_hash,model_id,language,classification,classification_confidence,relevance_rating,quality_rating,topics,entities,processing_status,summary
henri_kugel_begegnung.md,Documents/B/henri_kugel_begegnung.md,Henri und die Kugel,unknown,2025-05-14T21:37:12+02:00,1994,sha256:7f3a9c...,claude-sonnet-4-20250514,de,B,0.98,4,4,"science fiction;first contact;underwater exploration;telepathy;ancient civilization;mystery","Henri:character;die Kugel:object;Unterwasser-Zivilisation:collective",success,"Ein Taucher namens Henri entdeckt..."
```

Or as SQL:

```sql
INSERT INTO documents (filename, filepath, title, author, last_modified,
  file_size_bytes, content_hash, model_id, language, classification,
  classification_confidence, relevance_rating, quality_rating,
  processing_status, summary, notes)
VALUES ('henri_kugel_begegnung.md', 'Documents/B/henri_kugel_begegnung.md',
  'Henri und die Kugel', 'unknown', '2025-05-14 21:37:12+02:00',
  1994, 'sha256:7f3a9c...', 'claude-sonnet-4-20250514', 'de', 'B',
  0.98, 4, 4, 'success', 'Ein Taucher namens Henri...', '...');
```

## Design observations for your pipeline

1. **Topics/entities in SQL**: Semicolon-joined strings work for CSV, but for a real database consider separate `document_topics` and `document_entities` junction tables so you can query "all documents about first contact."
2. **Classification confidence semantics**: Since classification is *manual* (folder placement), the LLM's confidence is effectively an agreement check. A low confidence on a manually-classified file is a useful flag for misfiled documents — worth surfacing in a report.
3. **Rating scales**: Define them explicitly in your LLM prompt (e.g., 1–5, with anchored descriptions), otherwise ratings drift between runs and models.
4. **Language for summaries**: Decide whether summaries should be in the document's language (as above) or normalized to one language for consistent database searching.
5. **Chatbot transcripts**: Since both classes may contain them, consider adding a boolean `is_transcript` field — it's cheap to extract and useful for filtering later.

Want me to also produce a class "A" example, or draft the LLM prompt template that would reliably produce this JSON schema?

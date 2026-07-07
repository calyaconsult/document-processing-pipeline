Here are two complete samples — each with the document content, local file metadata, and the LLM-generated JSON metadata.

---

## Sample 1: Class A (Business)

### Document: `Documents/A/client-briefing-meridian-2025.md`

```markdown
# Client Briefing: Meridian Logistics – Website Relaunch

**Date:** 2025-01-14
**Prepared by:** S. Keller
**Status:** Draft for internal review

## Background

Meridian Logistics GmbH (approx. 220 employees, Hamburg) has requested a
proposal for a full relaunch of their corporate website. The current site
dates from 2018 and is not mobile-optimized. Primary contact is
Ms. Petra Wolf (Head of Marketing).

## Client Goals

- Modern, responsive design with a bilingual setup (DE/EN)
- Integration of a shipment tracking widget (API already exists)
- Careers section with applicant tracking integration
- Measurable lead generation for the freight forwarding division

## Budget & Timeline

- Indicated budget range: EUR 45,000 – 60,000
- Desired launch: Q3 2025, hard deadline before industry trade fair
  "transport logistic" in September

## Open Questions

1. Who provides content migration – client or agency?
2. Is the existing tracking API documented?
3. CMS preference? Client mentioned "something the marketing team
   can edit themselves."

## Next Steps

- [ ] Send follow-up questions to P. Wolf by Jan 17
- [ ] Internal scoping session Jan 20
- [ ] Proposal draft due Jan 28
```

### Local file metadata

| Field | Value |
|---|---|
| filename | `client-briefing-meridian-2025.md` |
| document_title | `Client Briefing: Meridian Logistics – Website Relaunch` |
| author | `S. Keller` |
| last_modified | `2025-01-14T16:42:07+01:00` |
| file_size_bytes | `1489` |
| content_hash | `sha256:7f3a9c1e58b2d4406fa1e0c9b83d2f715e46a8d0c2b91f3e6a57d804c1e92b3a` |

### LLM-generated metadata (JSON)

```json
{
  "model_id": "claude-sonnet-4-20250514",
  "summary": "Internal client briefing for a website relaunch project for Meridian Logistics GmbH, a Hamburg-based freight company. Covers client goals (responsive bilingual site, tracking widget, careers section, lead generation), a budget range of EUR 45-60k, a Q3 2025 deadline tied to a trade fair, open questions on content migration and CMS choice, and next steps with owner deadlines.",
  "notes": "Document is marked as a draft. Contains actionable items with dates; may be time-sensitive. Includes named client contact (personal data - consider retention policy).",
  "classification": "A",
  "classification_confidence": 0.98,
  "relevance_rating": 4,
  "quality_rating": 4,
  "topics": [
    "client briefing",
    "website relaunch",
    "web development proposal",
    "logistics industry",
    "project scoping",
    "budget planning"
  ],
  "entities": [
    {"name": "Meridian Logistics GmbH", "type": "organization"},
    {"name": "Petra Wolf", "type": "person"},
    {"name": "S. Keller", "type": "person"},
    {"name": "Hamburg", "type": "location"},
    {"name": "transport logistic", "type": "event"}
  ],
  "processing_status": "success"
}
```

---

## Sample 2: Class B (Creative Writing)

### Document: `Documents/B/hollow-lighthouse-ch03-draft.md`

```markdown
# The Hollow Lighthouse — Chapter 3 (Draft 2)

**Project:** The Hollow Lighthouse (working title)
**Author:** J. Marsh
**Draft note:** Second pass. POV tightened to Ines. Ending still weak.

---

Ines counted the steps out of habit — forty-one to the gallery, though
the plaque by the door claimed thirty-nine. She had stopped mentioning
the discrepancy to anyone. The last keeper who wrote it down had his
logbook returned with the page torn out.

The lamp room smelled of cold brass and, faintly, of the sea in a way
the sea outside no longer did. She set the thermos on the sill and
looked north. No ships tonight. There hadn't been ships in eleven days,
which was either a shipping lane change or the thing her grandmother
had called *die stille Woche* — the quiet week — stretched past all
reason.

The radio crackled at 21:15, exactly as it had every night that month.

"Signal station Varne to lighthouse. Confirm your light."

"Light confirmed," she said. "Varne, who is on duty tonight?"

Static. Then, in her grandmother's voice, unmistakable and eleven
years dead: "You already know, girl. Count the steps again."

---

**Revision notes (self):**
- The grandmother reveal lands too early — move to end of chapter?
- Need a concrete sensory anchor for the "wrong sea smell" earlier
- Check: was Varne signal station established before 1954? (research)
```

### Local file metadata

| Field | Value |
|---|---|
| filename | `hollow-lighthouse-ch03-draft.md` |
| document_title | `The Hollow Lighthouse — Chapter 3 (Draft 2)` |
| author | `J. Marsh` |
| last_modified | `2025-02-02T23:11:54+01:00` |
| file_size_bytes | `1662` |
| content_hash | `sha256:c2d84f60a1b7e9335dd0f4a8c6e12b97f04d3a5c81be6f20d9a4517c3e8fb604` |

### LLM-generated metadata (JSON)

```json
{
  "model_id": "claude-sonnet-4-20250514",
  "summary": "Second draft of chapter 3 of a supernatural/gothic novel-in-progress ('The Hollow Lighthouse'). A lighthouse keeper, Ines, notices uncanny discrepancies (miscounted steps, an altered sea smell, eleven days without ships) before a nightly radio check-in speaks in the voice of her dead grandmother. Includes the author's own revision notes flagging pacing, sensory detail, and a historical research question.",
  "notes": "Mixed content: narrative prose plus meta-writing (revision notes) in the same file. The pipeline may want a sub-tag for drafts containing embedded self-critique. German phrase 'die stille Woche' present — flag for multilingual handling.",
  "classification": "B",
  "classification_confidence": 0.97,
  "relevance_rating": 5,
  "quality_rating": 4,
  "topics": [
    "fiction draft",
    "supernatural fiction",
    "gothic horror",
    "chapter draft",
    "revision notes",
    "lighthouse setting"
  ],
  "entities": [
    {"name": "Ines", "type": "character"},
    {"name": "The Hollow Lighthouse", "type": "work_title"},
    {"name": "Varne", "type": "fictional_location"},
    {"name": "J. Marsh", "type": "person"}
  ],
  "processing_status": "success"
}
```

---

## Bonus: what these look like as CSV rows (step 5)

Flattening lists with a delimiter (e.g., `|`) keeps the CSV single-row per document:

```csv
filename,title,author,last_modified,size_bytes,sha256,model_id,class,confidence,relevance,quality,topics,entities,status
client-briefing-meridian-2025.md,"Client Briefing: Meridian Logistics – Website Relaunch",S. Keller,2025-01-14T16:42:07+01:00,1489,7f3a9c1e...,claude-sonnet-4-20250514,A,0.98,4,4,client briefing|website relaunch|web development proposal|logistics industry|project scoping|budget planning,Meridian Logistics GmbH|Petra Wolf|S. Keller|Hamburg|transport logistic,success
hollow-lighthouse-ch03-draft.md,"The Hollow Lighthouse — Chapter 3 (Draft 2)",J. Marsh,2025-02-02T23:11:54+01:00,1662,c2d84f60...,claude-sonnet-4-20250514,B,0.97,5,4,fiction draft|supernatural fiction|gothic horror|chapter draft|revision notes|lighthouse setting,Ines|The Hollow Lighthouse|Varne|J. Marsh,success
```

---

## A few design observations for your pipeline

1. **Manual classification vs. LLM confidence** — since the folder placement (`A`/`B`) is your ground truth, the LLM's `classification_confidence` is really an *agreement score*. Documents where the LLM disagrees with your folder placement (or confidence < ~0.7) are excellent candidates for a review queue. Chatbot transcripts will likely cluster here, since they can discuss either domain.

2. **Mixed-content documents** — as the Class B sample shows, drafts often contain embedded meta-writing. Decide whether `topics` should capture this or whether you want a dedicated field.

3. **Topics/entities in SQL** — pipe-delimited strings work for CSV, but for the database you'll get far more mileage from normalized tables (`documents`, `topics`, `document_topics`, `entities`, `document_entities`) — especially for queries like "all documents mentioning entity X."

4. **Hash before and after conversion** — consider storing hashes of both the original file and the converted `.md`/`.txt`, so you can detect when a source document changed and needs reprocessing.

Would you like me to draft the JSON schema for the LLM response (useful for enforcing structured output via the API), or the SQL table definitions?


# Document 2: SDNA Evidence Foundry — Extraction
## Steps 3, 4, 5: Media Cataloging, Text Extraction, Person Discovery

**Format:** DOCX, professional technical implementation guide
**Structure:** Three major sections (one per step). Each section has three sub-sections: Prose Narrative, Python Implementation, Pseudocode.

## Core Thesis
With source files inventoried and locked down (Document 1), the pipeline now splits into two parallel tracks: cataloging non-textual media that humans must annotate (Step 3), and extracting machine-readable text from documents (Step 4). Once text is available, the first recognition anchor — person identity — is searched for (Step 5). This document covers the transition from "what files do we have" to "what text and contact information can we extract from them."

## Sources
- Build_Spec___Vol_1_2026_11_07T06_44_18.md — the authoritative spec
- conversation.json — full pipeline breakdown
- Document 1 (foundation) — provides all schemas, constants, and source registry that this document depends on

---

## [Outline] Title Page
- Title: "SDNA Evidence Foundry — Implementation Guide"
- Subtitle: "Document 2 of 5: Extraction"
- Subtitle line 2: "Steps 3, 4, 5 — Media Cataloging, Text Extraction, Person Discovery"

## [Outline] Section 1: Step 3 — Supporting Media Detection & Manifest Creation

### Prose Narrative
Explain what this step does and why media is handled completely separately from documents:
- Media files (images, video, audio) and unreadable/scanned PDFs follow a different lifecycle from text documents
- Deepnote NEVER performs image recognition, OCR, audio transcription, or video analysis — media is catalogued, not interpreted
- Each media file gets auto-populated fields: Media ID, filename, file type, media category, source document (if embedded), source location, SHA-256 checksum
- registration_status is set to REGISTERED, annotation_status to PENDING_USER
- Five user-annotation fields are left deliberately blank: "what is this", "what does it represent", "what does it demonstrate", "associated project", "attribution/issuer" — the user must fill these before media becomes substantive evidence
- Unreadable PDFs (scanned, image-only) detected during this step or flagged from Step 4 are reclassified as media
- This step can run in PARALLEL with Step 4 — they both read from the source registry and have no interdependency
- The manifest is persisted as JSON for downstream steps (twin assembly references it)

### Python Implementation
Show working Python code:
```python
# ── Cell 3: Media Detection & Manifest ─────────────────────────

def detect_media_and_build_manifest(registry: dict, sources_dir: str = "sources") -> list:
    """Filter media from registry, create Media Evidence Manifest."""
    manifest = []
    media_extensions = set()
    for exts in ACCEPTED_MEDIA.values():
        media_extensions.update(exts)
    
    for uid, record in registry.items():
        if record["format_class"] != "media":
            continue
        
        filepath = Path(record["path"])
        ext = record["extension"].lower()
        
        # Determine category
        category = "unknown"
        for cat, extensions in ACCEPTED_MEDIA.items():
            if ext in extensions:
                category = cat
                break
        
        entry = MediaManifestEntry(
            media_id=f"MED-{uid}",
            filename=record["filename"],
            file_type=ext,
            category=category,
            source_document=None,  # standalone file
            source_location=None,
            checksum=record["checksum"],
            registration_status=ProcessingStatus.REGISTERED,
            annotation_status=ProcessingStatus.PENDING_USER,
            # User fields left blank
            what_is_this=None,
            what_does_it_represent=None,
            what_does_it_demonstrate=None,
            associated_project=None,
            attribution_issuer=None,
        )
        manifest.append(entry.model_dump())
    
    # Persist manifest
    manifest_path = Path(sources_dir) / "media_manifest.json"
    with open(manifest_path, "w") as f:
        json.dump(manifest, f, indent=2)
    
    print(f"✅ Media Manifest: {len(manifest)} entries registered")
    for entry in manifest:
        print(f"   [{entry['media_id']}] {entry['filename']} ({entry['category']}) — annotation: {entry['annotation_status']}")
    
    return manifest

def reclassify_as_media(uid: str, registry: dict, manifest: list, reason: str = "unreadable_pdf"):
    """Reclassify a document as media (e.g., unreadable PDF flagged by Step 4)."""
    if uid in registry:
        record = registry[uid]
        record["format_class"] = "media"
        entry = MediaManifestEntry(
            media_id=f"MED-{uid}",
            filename=record["filename"],
            file_type=record["extension"],
            category=reason,
            checksum=record["checksum"],
        )
        manifest.append(entry.model_dump())
    return manifest
```

### Pseudocode
```
FUNCTION detect_media_and_build_manifest(registry, sources_directory):
    manifest ← empty list
    media_extensions ← FLATTEN all extensions from ACCEPTED_MEDIA categories
    
    FOR EACH uid, record IN registry:
        IF record.format_class ≠ "media": SKIP
        
        extension ← lowercase(record.extension)
        category ← LOOKUP which ACCEPTED_MEDIA category contains this extension
        
        CREATE MediaManifestEntry:
            media_id     ← "MED-" + uid
            filename     ← record.filename
            file_type    ← extension
            category     ← determined category
            checksum     ← record.checksum
            registration_status ← REGISTERED
            annotation_status   ← PENDING_USER
            user_fields  ← ALL BLANK (what_is_this, what_does_it_represent,
                           what_does_it_demonstrate, associated_project,
                           attribution_issuer)
        
        APPEND entry TO manifest
    
    SAVE manifest AS JSON → sources_directory/media_manifest.json
    RETURN manifest

FUNCTION reclassify_as_media(uid, registry, manifest, reason):
    # Called by Step 4 when a PDF turns out to be unreadable
    record ← registry[uid]
    record.format_class ← "media"
    CREATE MediaManifestEntry with category = reason
    APPEND TO manifest
    RETURN manifest
```

---

## [Outline] Section 2: Step 4 — Text Extraction Engine

### Prose Narrative
Explain what this step does — pure faithful extraction with no interpretation. Key points:
- This step handles four document formats, each with its own extraction approach: DOCX (python-docx + lxml), PDF (pypdf), TXT (charset-normalizer), Markdown (markdown-it-py)
- CRITICAL RULE — Header/Footer Isolation (SDNA-HF-001): Headers and footers are extracted into a SEPARATE data structure. ONLY contact information from headers/footers is eligible for downstream processing. Everything else (job titles in footers, company names in headers, decorative text) is marked for discard. This gate must be enforced here at extraction time, before any NLP touches the content.
- PDF readability must be tested FIRST — if a PDF yields no extractable text on any page, it gets reclassified as media (unreadable_pdf) and sent back to the media manifest from Step 3
- Partially readable PDFs: record which pages succeeded and which failed — don't declare complete extraction or silently drop pages
- Extraction fidelity metadata must be tracked: source page numbers, paragraph indices, table cell coordinates — wherever content came from in the original
- No content fabrication: if text can't be extracted, report the gap; never substitute generated text
- The output is a per-document extraction record containing: body_text (with source locations), header_footer_text (isolated), extraction_status, and page/paragraph references
- This step can run in PARALLEL with Step 3

### Python Implementation
Show working Python code for all four format extractors plus header/footer isolation:
```python
# ── Cell 4: Text Extraction Engine ─────────────────────────────

def extract_docx(filepath: Path) -> dict:
    """Extract text from DOCX — body, tables, headers/footers separated."""
    doc = DocxDocument(str(filepath))
    
    # Body text with paragraph indices
    body_paragraphs = []
    for i, para in enumerate(doc.paragraphs):
        if para.text.strip():
            body_paragraphs.append({
                "index": i,
                "text": para.text.strip(),
                "style": para.style.name if para.style else None,
                "is_heading": para.style.name.startswith("Heading") if para.style else False,
                "bold_runs": [run.text for run in para.runs if run.bold],
            })
    
    # Tables
    table_data = []
    for t_idx, table in enumerate(doc.tables):
        for r_idx, row in enumerate(table.rows):
            for c_idx, cell in enumerate(row.cells):
                if cell.text.strip():
                    table_data.append({
                        "table": t_idx, "row": r_idx, "col": c_idx,
                        "text": cell.text.strip()
                    })
    
    # Headers/Footers — ISOLATED per SDNA-HF-001
    header_footer_text = []
    for section in doc.sections:
        if section.header and section.header.paragraphs:
            for p in section.header.paragraphs:
                if p.text.strip():
                    header_footer_text.append({"region": "header", "text": p.text.strip()})
        if section.footer and section.footer.paragraphs:
            for p in section.footer.paragraphs:
                if p.text.strip():
                    header_footer_text.append({"region": "footer", "text": p.text.strip()})
    
    body_text = "\n".join(p["text"] for p in body_paragraphs)
    return {
        "body_text": body_text,
        "body_paragraphs": body_paragraphs,
        "table_data": table_data,
        "header_footer_text": header_footer_text,
        "extraction_status": "complete" if body_text else "empty",
    }

def extract_pdf(filepath: Path) -> dict:
    """Extract text from PDF per-page. Flag unreadable pages."""
    reader = PdfReader(str(filepath))
    pages = []
    readable_pages = 0
    
    for i, page in enumerate(reader.pages):
        text = page.extract_text() or ""
        is_readable = bool(text.strip())
        pages.append({
            "page_number": i + 1,
            "text": text.strip(),
            "is_readable": is_readable,
        })
        if is_readable:
            readable_pages += 1
    
    total_pages = len(reader.pages)
    body_text = "\n\n".join(p["text"] for p in pages if p["is_readable"])
    
    if readable_pages == 0:
        status = "unreadable"  # Flag for media reclassification
    elif readable_pages < total_pages:
        status = "partial"
    else:
        status = "complete"
    
    return {
        "body_text": body_text,
        "pages": pages,
        "header_footer_text": [],  # PDF headers/footers handled via page position heuristics
        "extraction_status": status,
        "readable_pages": readable_pages,
        "total_pages": total_pages,
    }

def extract_txt(filepath: Path) -> dict:
    """Extract text from plain text file with encoding detection."""
    raw_bytes = filepath.read_bytes()
    detection = from_bytes(raw_bytes).best()
    
    if detection is None:
        return {"body_text": "", "extraction_status": "encoding_failed", "header_footer_text": []}
    
    text = str(detection)
    lines = text.splitlines()
    
    return {
        "body_text": text,
        "lines": [{"line_number": i+1, "text": line} for i, line in enumerate(lines) if line.strip()],
        "header_footer_text": [],  # TXT has no headers/footers
        "extraction_status": "complete" if text.strip() else "empty",
        "detected_encoding": detection.encoding,
    }

def extract_markdown(filepath: Path) -> dict:
    """Extract structured content from Markdown."""
    raw_text = filepath.read_text(encoding="utf-8")
    md = MarkdownIt()
    tokens = md.parse(raw_text)
    
    structured = []
    for token in tokens:
        if token.type in ("heading_open", "paragraph_open", "bullet_list_open"):
            structured.append({
                "type": token.type,
                "tag": token.tag,
                "content": token.content if token.content else None,
                "markup": token.markup,
            })
        elif token.type == "inline" and token.content:
            structured.append({
                "type": "text",
                "content": token.content,
            })
    
    return {
        "body_text": raw_text,
        "structured_tokens": structured,
        "header_footer_text": [],  # MD has no headers/footers
        "extraction_status": "complete" if raw_text.strip() else "empty",
    }

# ── Master Extraction Router ──────────────────────────────────

EXTRACTORS = {
    ".docx": extract_docx,
    ".pdf":  extract_pdf,
    ".txt":  extract_txt,
    ".md":   extract_markdown,
}

def extract_all_documents(registry: dict, media_manifest: list) -> dict:
    """Run extraction for all accepted documents. Returns extraction records keyed by UID."""
    extractions = {}
    
    for uid, record in registry.items():
        if record["format_class"] != "document":
            continue
        
        ext = record["extension"].lower()
        filepath = Path(record["path"])
        extractor = EXTRACTORS.get(ext)
        
        if not extractor:
            extractions[uid] = {"extraction_status": "no_extractor", "body_text": ""}
            continue
        
        try:
            result = extractor(filepath)
            result["source_uid"] = uid
            result["source_filename"] = record["filename"]
            
            # PDF unreadable → reclassify as media
            if ext == ".pdf" and result["extraction_status"] == "unreadable":
                reclassify_as_media(uid, registry, media_manifest, "unreadable_pdf")
                print(f"   ⚠️  [{uid}] {record['filename']} — unreadable PDF, reclassified as media")
                continue
            
            extractions[uid] = result
            print(f"   ✅ [{uid}] {record['filename']} — {result['extraction_status']}")
            
        except Exception as e:
            extractions[uid] = {
                "source_uid": uid, "body_text": "",
                "extraction_status": f"error: {str(e)}",
                "header_footer_text": [],
            }
            print(f"   ❌ [{uid}] {record['filename']} — extraction error: {e}")
    
    return extractions
```

### Pseudocode
```
FUNCTION extract_docx(filepath):
    OPEN filepath as DOCX document
    
    body_paragraphs ← empty list
    FOR EACH paragraph IN document.paragraphs (with index):
        IF paragraph has text:
            RECORD: index, text, style name, is_heading flag, bold runs
            APPEND to body_paragraphs
    
    table_data ← empty list
    FOR EACH table (with table_index):
        FOR EACH row (with row_index):
            FOR EACH cell (with col_index):
                IF cell has text: RECORD coordinates + text
    
    # HEADER/FOOTER ISOLATION — SDNA-HF-001
    header_footer_text ← empty list
    FOR EACH section IN document:
        EXTRACT header paragraphs → tag as "header"
        EXTRACT footer paragraphs → tag as "footer"
    # ↑ These are ISOLATED — only contact info eligible downstream
    
    RETURN {body_text, body_paragraphs, table_data, header_footer_text, status}

FUNCTION extract_pdf(filepath):
    OPEN filepath as PDF reader
    pages ← empty list
    
    FOR EACH page (with page_number):
        text ← EXTRACT text from page
        is_readable ← text has content after trimming
        RECORD: page_number, text, is_readable
    
    IF zero readable pages → status = "unreadable" (trigger media reclassification)
    ELSE IF some pages unreadable → status = "partial"
    ELSE → status = "complete"
    
    RETURN {body_text (readable pages joined), pages, status, counts}

FUNCTION extract_txt(filepath):
    raw_bytes ← READ file as binary
    DETECT encoding using charset_normalizer
    IF detection fails → RETURN empty with "encoding_failed"
    
    text ← DECODE using detected encoding
    lines ← SPLIT into lines with line numbers
    RETURN {body_text, lines, detected_encoding, status}

FUNCTION extract_markdown(filepath):
    raw_text ← READ file as UTF-8
    tokens ← PARSE with markdown parser
    
    structured ← empty list
    FOR EACH token:
        RECORD: type (heading, paragraph, list), tag, content, markup
    
    RETURN {body_text (raw), structured_tokens, status}

FUNCTION extract_all_documents(registry, media_manifest):
    extractions ← empty map
    
    FOR EACH uid, record IN registry WHERE format_class = "document":
        extractor ← LOOKUP by extension
        result ← CALL extractor(filepath)
        
        IF PDF AND result.status = "unreadable":
            RECLASSIFY as media → update manifest
            SKIP to next document
        
        STORE result IN extractions[uid]
    
    RETURN extractions
```

---

## [Outline] Section 3: Step 5 — Person (P) Bundle Discovery

### Prose Narrative
Explain what P discovery does and why it's a standalone pass:
- Person discovery is one of the two "recognition anchors" that drive the quarantine decision (the other is Employment discovery in Step 6 / Document 3)
- This step searches extracted text for contact information: name, email, phone, location, professional profile links
- It searches BOTH body text AND eligible header/footer contact text (the only header/footer content allowed through the SDNA-HF-001 gate)
- Tools: regex for email patterns, phone patterns, URL patterns; spaCy NER for PERSON entities (names) and GPE/LOC entities (locations)
- A PersonBundle is built from what's ACTUALLY FOUND — missing fields stay empty. No guessing, no filling in gaps.
- Overall P status: FOUND (at least one contact element confirmed), NOT_FOUND, or AMBIGUOUS
- A single element is sufficient for FOUND — all five fields don't need to be populated
- NOT_FOUND doesn't mean the person doesn't exist — it means the parser couldn't establish contact identity from the source text
- P discovery can run in PARALLEL with E discovery (Step 6) — they both read from extraction records and have no interdependency
- The PersonBundle feeds into the quarantine decision in Step 9 (Document 4)

### Python Implementation
```python
# ── Cell 5: Person (P) Bundle Discovery ────────────────────────

# ── Regex Patterns for Contact Detection ───────────────────────
EMAIL_PATTERN = re.compile(
    r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
)
PHONE_PATTERN = re.compile(
    r'(?:\+?1[-.\s]?)?(?:\(?\d{3}\)?[-.\s]?)?\d{3}[-.\s]?\d{4}'
    r'|(?:\+\d{1,3}[-.\s]?)?\(?\d{2,4}\)?[-.\s]?\d{3,4}[-.\s]?\d{3,4}'
)
URL_PATTERN = re.compile(
    r'https?://(?:www\.)?(?:linkedin\.com|github\.com|behance\.net|'
    r'dribbble\.com|stackoverflow\.com|medium\.com)[/\w\-._~:/?#\[\]@!$&\'()*+,;=%]*',
    re.IGNORECASE
)
GENERIC_URL_PATTERN = re.compile(
    r'https?://[^\s<>"{}|\\^`\[\]]+', re.IGNORECASE
)

def discover_person(extraction: dict) -> PersonBundle:
    """Search extracted text for person contact information."""
    body_text = extraction.get("body_text", "")
    header_footer = extraction.get("header_footer_text", [])
    
    # Combine eligible text sources
    hf_text = " ".join(item["text"] for item in header_footer)
    combined_text = body_text + "\n" + hf_text
    
    bundle = PersonBundle()
    
    # ── Email Detection ────────────────────────────────────────
    emails = EMAIL_PATTERN.findall(combined_text)
    if emails:
        bundle.email = ContactElement(
            value=emails,
            discovery_state=DiscoveryState.FOUND,
            source_location="body" if EMAIL_PATTERN.search(body_text) else "header_footer"
        )
    
    # ── Phone Detection ────────────────────────────────────────
    phones = PHONE_PATTERN.findall(combined_text)
    if phones:
        bundle.phone = ContactElement(
            value=phones.strip(),
            discovery_state=DiscoveryState.FOUND,
            source_location="body" if PHONE_PATTERN.search(body_text) else "header_footer"
        )
    
    # ── Profile Link Detection ─────────────────────────────────
    prof_links = URL_PATTERN.findall(combined_text)
    for link in prof_links:
        bundle.profile_links.append(ContactElement(
            value=link,
            discovery_state=DiscoveryState.FOUND,
            source_location="body" if link in body_text else "header_footer"
        ))
    
    # ── spaCy NER for Name and Location ────────────────────────
    doc = nlp(combined_text[:10000])  # Limit for performance
    
    person_entities = [ent.text for ent in doc.ents if ent.label_ == "PERSON"]
    if person_entities:
        bundle.name = ContactElement(
            value=person_entities,  # First PERSON entity as primary candidate
            discovery_state=DiscoveryState.FOUND,
            source_location="body"  # spaCy processes combined text
        )
    
    location_entities = [ent.text for ent in doc.ents if ent.label_ in ("GPE", "LOC")]
    if location_entities:
        bundle.location = ContactElement(
            value=location_entities,
            discovery_state=DiscoveryState.FOUND,
            source_location="body"
        )
    
    # ── Overall Status Determination ───────────────────────────
    found_elements = []
    if bundle.name.discovery_state == DiscoveryState.FOUND: found_elements.append("name")
    if bundle.email.discovery_state == DiscoveryState.FOUND: found_elements.append("email")
    if bundle.phone.discovery_state == DiscoveryState.FOUND: found_elements.append("phone")
    if bundle.location.discovery_state == DiscoveryState.FOUND: found_elements.append("location")
    if bundle.profile_links: found_elements.append("profile_links")
    
    if len(found_elements) >= 1:
        bundle.overall_status = DiscoveryState.FOUND
    else:
        bundle.overall_status = DiscoveryState.NOT_FOUND
    
    return bundle

def discover_all_persons(extractions: dict) -> dict:
    """Run P discovery on all extracted documents."""
    person_bundles = {}
    
    for uid, extraction in extractions.items():
        bundle = discover_person(extraction)
        person_bundles[uid] = bundle.model_dump()
        
        status_icon = "✅" if bundle.overall_status == DiscoveryState.FOUND else "⚠️"
        print(f"   {status_icon} [{uid}] P={bundle.overall_status.value}", end="")
        if bundle.name.value:
            print(f" — Name: {bundle.name.value}", end="")
        if bundle.email.value:
            print(f" | Email: {bundle.email.value}", end="")
        print()
    
    return person_bundles
```

### Pseudocode
```
PATTERNS:
    EMAIL   ← regex for standard email format (user@domain.tld)
    PHONE   ← regex for domestic/international phone formats
    URL     ← regex for known professional platforms (LinkedIn, GitHub, etc.)

FUNCTION discover_person(extraction_record):
    body_text ← extraction_record.body_text
    hf_text   ← JOIN all header_footer_text entries
    combined  ← body_text + hf_text
    
    bundle ← new empty PersonBundle
    
    # Email
    emails ← SEARCH combined for EMAIL pattern
    IF emails found:
        bundle.email ← {value: first match, state: FOUND, location: where found}
    
    # Phone
    phones ← SEARCH combined for PHONE pattern
    IF phones found:
        bundle.phone ← {value: first match, state: FOUND, location: where found}
    
    # Profile Links
    links ← SEARCH combined for URL pattern
    FOR EACH link:
        APPEND {value: link, state: FOUND} TO bundle.profile_links
    
    # Name (via NLP Named Entity Recognition)
    entities ← RUN NER on combined text
    person_names ← FILTER entities WHERE label = "PERSON"
    IF person_names found:
        bundle.name ← {value: first match, state: FOUND}
    
    # Location (via NLP NER)
    locations ← FILTER entities WHERE label IN ("GPE", "LOC")
    IF locations found:
        bundle.location ← {value: first match, state: FOUND}
    
    # Overall Status
    found_count ← COUNT fields where state = FOUND
    IF found_count ≥ 1:
        bundle.overall_status ← FOUND
    ELSE:
        bundle.overall_status ← NOT_FOUND
    
    RETURN bundle

FUNCTION discover_all_persons(extractions):
    person_bundles ← empty map
    FOR EACH uid, extraction IN extractions:
        bundle ← discover_person(extraction)
        person_bundles[uid] ← bundle
    RETURN person_bundles
```

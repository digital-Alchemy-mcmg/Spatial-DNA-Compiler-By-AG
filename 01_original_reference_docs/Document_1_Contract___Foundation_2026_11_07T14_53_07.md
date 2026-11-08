
# Document 1: SDNA Evidence Foundry — Foundation
## Steps 0, 1, 2: Environment, Data Contracts, Source Registration

**Format:** DOCX, professional technical implementation guide
**Structure:** Three major sections (one per step). Each section has three sub-sections: Prose Narrative, Python Implementation, Pseudocode.

## Core Thesis
Before the pipeline processes a single byte of résumé content, it needs three things in exact order: every library installed and confirmed (Step 0), every data structure and constant defined as the shared vocabulary (Step 1), and every submitted file inventoried, fingerprinted, and locked down as immutable source material (Step 2). This document establishes all three.

## Sources
- Build_Spec___Vol_1_2026_11_07T06_44_18.md — the authoritative spec
- conversation.json — full pipeline breakdown with all task details

---

## [Outline] Title Page
- Title: "SDNA Evidence Foundry — Implementation Guide"
- Subtitle: "Document 1 of 5: Foundation"
- Subtitle line 2: "Steps 0, 1, 2 — Environment Setup, Data Contracts, Source Registration"
- Footer note: "Part of the Deepnote Evidence Foundry Pipeline"

## [Outline] Section 1: Step 0 — Environment Setup & Dependency Installation

### Prose Narrative
Explain what this step accomplishes and WHY all dependencies must be installed in one place at the very beginning. Key points to cover in prose:
- The pipeline uses python-docx, lxml, pypdf, charset-normalizer, markdown-it-py, spacy (with en_core_web_sm model), regex, pydantic, and openpyxl
- Standard library modules needed: pathlib, hashlib, mimetypes, zipfile, shutil, json, re, logging, uuid, string
- The spec explicitly requires runtime qualification — you must CONFIRM every import succeeds, not just assume it
- pypdf gets special attention: the spec notes that PDF processing was previously blocked by missing dependencies, so pypdf must be tested against an actual PDF (or confirmed importable) before proceeding
- spaCy requires a separate model download (en_core_web_sm) which is a network operation — belongs here, not mid-pipeline
- The rationale: if you scatter installs across later steps, a failure in Step 7 could waste all work from Steps 2–6. Fail fast, fail here.

### Python Implementation
Show the actual working Python code:
```python
# ── Cell 0: Environment Setup ──────────────────────────────────
import subprocess, sys

# All pip installs — one shot
packages = [
    "python-docx", "lxml", "pypdf", "charset-normalizer",
    "markdown-it-py", "spacy", "regex", "pydantic", "openpyxl"
]
for pkg in packages:
    subprocess.check_call([sys.executable, "-m", "pip", "install", "-q", pkg])

# spaCy model download
subprocess.check_call([sys.executable, "-m", "spacy", "download", "en_core_web_sm"])

# ── Standard Library Imports ───────────────────────────────────
from pathlib import Path
import hashlib, mimetypes, zipfile, shutil, json, re, logging, uuid, string

# ── Third-Party Imports ────────────────────────────────────────
from docx import Document as DocxDocument
import lxml.etree as etree
from pypdf import PdfReader
from charset_normalizer import from_bytes
from markdown_it import MarkdownIt
import spacy
import regex as rx
from pydantic import BaseModel, Field
from typing import Optional, List, Dict
from enum import Enum
from openpyxl import Workbook

# ── Runtime Qualification ──────────────────────────────────────
nlp = spacy.load("en_core_web_sm")
print(f"[OK] spaCy model loaded: {nlp.meta['name']} v{nlp.meta['version']}")

import pypdf
print(f"[OK] pypdf version: {pypdf.__version__}")

import docx
print(f"[OK] python-docx imported successfully")

import openpyxl
print(f"[OK] openpyxl version: {openpyxl.__version__}")

print("\n✅ All dependencies installed and verified.")
```

### Pseudocode
```
PROCEDURE setup_environment():
    required_packages ← [python-docx, lxml, pypdf, charset-normalizer,
                          markdown-it-py, spacy, regex, pydantic, openpyxl]
    
    FOR EACH package IN required_packages:
        INSTALL package via package manager
        IF installation fails:
            ABORT with error: "Missing dependency: {package}"
    
    DOWNLOAD spacy language model "en_core_web_sm"
    
    # Standard library — no install needed, just verify availability
    required_stdlib ← [pathlib, hashlib, mimetypes, zipfile, shutil,
                        json, re, logging, uuid, string]
    
    FOR EACH module IN required_stdlib:
        IMPORT module
        IF import fails:
            ABORT — Python installation is broken
    
    FOR EACH third_party_module IN required_packages:
        IMPORT module
        RECORD version number
        IF import fails:
            ABORT with error: "Package installed but import failed: {module}"
    
    # Runtime qualification — special attention items
    LOAD spacy model "en_core_web_sm" → nlp
    VERIFY nlp can process a test sentence
    
    IMPORT pypdf
    VERIFY pypdf.PdfReader is callable
    
    PRINT summary of all verified dependencies with versions
    RETURN success
```

---

## [Outline] Section 2: Step 1 — Configuration, Constants & Pydantic Data Models

### Prose Narrative
Explain what this step does and why data contracts must exist before any processing. Key points:
- This step defines the entire shared vocabulary of the pipeline — every downstream step validates its inputs and outputs against these schemas
- No processing logic here — just constants, enums, and Pydantic models
- ACCEPTED_FORMATS: .docx, .txt, .pdf (machine-readable only), .md — these are the ONLY document formats that enter the pipeline
- ACCEPTED_MEDIA: images (PNG, JPG, JPEG, TIFF, WEBP), video (MP4), audio (MP3, WAV, M4A), unreadable PDFs — catalogued but never machine-interpreted
- UNSUPPORTED_FORMATS: CSV, XLS, XLSX — these get explicit rejection receipts, not silent ignoring
- UNICODE_CITATION_POOL: ¿ ¡ ¶ § ‽ ⁂ † ‡ ⁊ ⁓ — these are assigned to employment declarations in order; when the pool is exhausted, symbols are doubled (¿¿, ¿¡, etc.)
- Explain each Pydantic model briefly: what it represents, what fields it holds, and which step produces/consumes it
- The SourceRecord tracks a file's identity from admission through packaging
- MediaManifestEntry has auto-populated fields AND blank user-annotation fields — this split is deliberate
- PersonBundle and EmploymentDeclaration are the two "recognition anchors" for the quarantine decision
- AtomicBlurb is the fundamental unit of evidence — one independently meaningful claim
- DocumentTwin is the complete evidence mirror of a source document
- CompletionFooter can ONLY be written after all verification checks pass — it starts as None
- ValidationResult records pass/fail per verification check
- QuarantineRecord explains why a document was quarantined

### Python Implementation
Show complete, working Python code for all constants, enums, and Pydantic models:
```python
# ── Cell 1: Configuration, Constants & Data Models ─────────────

# ── Accepted Document Formats ──────────────────────────────────
ACCEPTED_FORMATS = {
    ".docx": "Microsoft Word — python-docx extraction",
    ".txt":  "Plain text — charset-normalizer encoding detection",
    ".pdf":  "PDF — pypdf extraction (machine-readable only)",
    ".md":   "Markdown — markdown-it-py structural parsing",
}

# ── Accepted Media Types ───────────────────────────────────────
ACCEPTED_MEDIA = {
    "image": [".png", ".jpg", ".jpeg", ".tiff", ".webp"],
    "video": [".mp4"],
    "audio": [".mp3", ".wav", ".m4a"],
    "unreadable_pdf": [".pdf"],  # PDFs that fail text extraction
}

# ── Unsupported Formats (explicit rejection) ───────────────────
UNSUPPORTED_FORMATS = [".csv", ".xls", ".xlsx"]

# ── Unicode Citation Pool ──────────────────────────────────────
UNICODE_CITATION_POOL = ["¿", "¡", "¶", "§", "‽", "⁂", "†", "‡", "⁊", "⁓"]

def get_citation_symbol(index: int) -> str:
    """Assign citation symbol by index. After pool exhaustion, double symbols."""
    pool_size = len(UNICODE_CITATION_POOL)
    if index < pool_size:
        return UNICODE_CITATION_POOL[index]
    # Doubling: index 10 = ¿¿, index 11 = ¿¡, etc.
    overflow = index - pool_size
    first = UNICODE_CITATION_POOL[overflow // pool_size % pool_size]
    second = UNICODE_CITATION_POOL[overflow % pool_size]
    return first + second

# ── Enums ──────────────────────────────────────────────────────
class DiscoveryState(str, Enum):
    FOUND = "FOUND"
    NOT_FOUND = "NOT_FOUND"
    AMBIGUOUS = "AMBIGUOUS"

class ProcessingStatus(str, Enum):
    REGISTERED = "REGISTERED"
    PENDING_USER = "PENDING_USER"
    QUARANTINED = "QUARANTINED"
    PROCESSING = "PROCESSING"
    VALIDATED = "VALIDATED"
    COMPLETE = "COMPLETE"

# ── Pydantic Models ────────────────────────────────────────────
class SourceRecord(BaseModel):
    uid: str = Field(..., min_length=3, max_length=3, description="3-char alphanumeric UID")
    filename: str
    extension: str
    checksum: str = Field(..., description="SHA-256 hex digest")
    format_class: str = Field(..., description="'document', 'media', or 'unsupported'")
    path: str
    status: ProcessingStatus = ProcessingStatus.REGISTERED

class MediaManifestEntry(BaseModel):
    media_id: str
    filename: str
    file_type: str
    category: str  # image, video, audio, unreadable_pdf
    source_document: Optional[str] = None  # if embedded in a doc
    source_location: Optional[str] = None
    checksum: str
    registration_status: ProcessingStatus = ProcessingStatus.REGISTERED
    annotation_status: ProcessingStatus = ProcessingStatus.PENDING_USER
    # User annotation fields — blank until user fills them
    what_is_this: Optional[str] = None
    what_does_it_represent: Optional[str] = None
    what_does_it_demonstrate: Optional[str] = None
    associated_project: Optional[str] = None
    attribution_issuer: Optional[str] = None

class ContactElement(BaseModel):
    value: Optional[str] = None
    discovery_state: DiscoveryState = DiscoveryState.NOT_FOUND
    source_location: Optional[str] = None

class PersonBundle(BaseModel):
    name: ContactElement = ContactElement()
    email: ContactElement = ContactElement()
    phone: ContactElement = ContactElement()
    location: ContactElement = ContactElement()
    profile_links: List[ContactElement] = []
    overall_status: DiscoveryState = DiscoveryState.NOT_FOUND

class EmploymentDeclaration(BaseModel):
    label: str  # "Employer A", "Employer B", etc.
    unicode_citation: str  # ¿, ¡, ¶, etc.
    employer: Optional[str] = None
    position: Optional[str] = None
    dates: Optional[str] = None
    source_locations: List[str] = []
    discovery_state: DiscoveryState = DiscoveryState.FOUND

class AtomicBlurb(BaseModel):
    uid: str  # e.g., "A7B-001"
    content: str
    citation: Optional[str] = None  # Unicode symbol or None if uncited
    employment_label: Optional[str] = None  # "Employer A" etc.
    source_location: str
    is_cited: bool = False

class NonBlurbBundle(BaseModel):
    bundle_type: str  # "Skills & Keywords", "Executive Summary", etc.
    content: str
    classification: str = "SELF_REPORTED"
    source_location: str

class CompletionFooter(BaseModel):
    source_uid: str
    twin_id: str
    atom_count: int
    last_atom_uid: str
    employment_count: int
    media_count: int
    pending_annotations: int
    status: ProcessingStatus
    content_checksum: str

class ValidationResult(BaseModel):
    source_uid: str
    check_id: str  # "A" through "J"
    passed: bool
    observed: str
    expected: str
    failure_reason: Optional[str] = None

class QuarantineRecord(BaseModel):
    source_uid: str
    reason: str
    discovery_results: Dict[str, str]
    processing_status: ProcessingStatus = ProcessingStatus.QUARANTINED

class DocumentTwin(BaseModel):
    twin_id: str  # "{source_uid}-000"
    source_uid: str
    source_checksum: str
    person_bundle: Optional[PersonBundle] = None
    employment_declarations: List[EmploymentDeclaration] = []
    atomic_blurbs: List[AtomicBlurb] = []
    non_blurb_info: List[NonBlurbBundle] = []
    media_refs: List[str] = []
    discovery_results: Dict[str, str] = {}
    completion_footer: Optional[CompletionFooter] = None

print("✅ All constants, enums, and Pydantic models defined.")
```

### Pseudocode
```
CONSTANTS:
    ACCEPTED_FORMATS ← {".docx", ".txt", ".pdf", ".md"}
    ACCEPTED_MEDIA ← {image: [".png",".jpg",".jpeg",".tiff",".webp"],
                       video: [".mp4"], audio: [".mp3",".wav",".m4a"],
                       unreadable_pdf: [".pdf"]}
    UNSUPPORTED_FORMATS ← {".csv", ".xls", ".xlsx"}
    CITATION_POOL ← ["¿","¡","¶","§","‽","⁂","†","‡","⁊","⁓"]

FUNCTION get_citation(index):
    IF index < LENGTH(CITATION_POOL):
        RETURN CITATION_POOL[index]
    ELSE:
        overflow ← index - LENGTH(CITATION_POOL)
        RETURN CITATION_POOL[overflow / pool_size] + CITATION_POOL[overflow % pool_size]

ENUM DiscoveryState: FOUND, NOT_FOUND, AMBIGUOUS
ENUM ProcessingStatus: REGISTERED, PENDING_USER, QUARANTINED, PROCESSING, VALIDATED, COMPLETE

RECORD SourceRecord:
    uid: string(3)          # Unique 3-char alphanumeric
    filename: string
    extension: string
    checksum: string        # SHA-256
    format_class: string    # "document" | "media" | "unsupported"
    path: string
    status: ProcessingStatus

RECORD MediaManifestEntry:
    media_id, filename, file_type, category, source_document, source_location, checksum
    registration_status: REGISTERED
    annotation_status: PENDING_USER
    user_fields: {what_is_this, what_does_it_represent, what_does_it_demonstrate,
                  associated_project, attribution_issuer} ← ALL BLANK

RECORD PersonBundle:
    name, email, phone, location, profile_links
    # Each sub-field: {value, discovery_state, source_location}
    overall_status: DiscoveryState

RECORD EmploymentDeclaration:
    label, unicode_citation, employer, position, dates, source_locations, discovery_state

RECORD AtomicBlurb:
    uid, content, citation, employment_label, source_location, is_cited

RECORD NonBlurbBundle:
    bundle_type, content, classification="SELF_REPORTED", source_location

RECORD DocumentTwin:
    twin_id, source_uid, source_checksum, person_bundle,
    employment_declarations[], atomic_blurbs[], non_blurb_info[],
    media_refs[], discovery_results, completion_footer (initially NULL)

RECORD CompletionFooter:
    source_uid, twin_id, atom_count, last_atom_uid, employment_count,
    media_count, pending_annotations, status, content_checksum

RECORD ValidationResult:
    source_uid, check_id, passed, observed, expected, failure_reason

RECORD QuarantineRecord:
    source_uid, reason, discovery_results, processing_status=QUARANTINED
```

---

## [Outline] Section 3: Step 2 — Source File Discovery & Registration

### Prose Narrative
Explain what this step does and why it's separate from extraction. Key points:
- This is pure inventory — you need to know what files you have, what format they're in, and lock them down before touching their content
- Every accepted file gets a unique 3-character alphanumeric UID (collision-checked, using uppercase letters + digits)
- SHA-256 checksums are computed from the original bytes and stored — this is the file's identity fingerprint for the rest of the pipeline
- Original files are COPIED (not moved) into a sources/ directory — from this point forward, the originals are immutable; no step may alter them
- Files are sorted into three buckets: accepted_documents (for text extraction), accepted_media (for the media manifest), unsupported (for rejection receipts)
- Unsupported formats (CSV, XLS, XLSX) get explicit rejection receipts — not silent drops
- The source registry is the pipeline's ground truth — every downstream step references it
- Byte identity matters: if the same file is submitted twice with identical bytes, the checksum match reveals it as a duplicate (handled in Step 11)
- ZIP packages may contain multiple files — they should be inspected and their contents registered individually

### Python Implementation
```python
# ── Cell 2: Source File Discovery & Registration ───────────────

import random

def generate_uid(existing_uids: set) -> str:
    """Generate a unique 3-character alphanumeric UID."""
    chars = string.ascii_uppercase + string.digits
    while True:
        uid = ''.join(random.choices(chars, k=3))
        if uid not in existing_uids:
            existing_uids.add(uid)
            return uid

def compute_checksum(filepath: Path) -> str:
    """Compute SHA-256 checksum of file bytes."""
    sha256 = hashlib.sha256()
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            sha256.update(chunk)
    return sha256.hexdigest()

def classify_file(filepath: Path) -> str:
    """Classify file as 'document', 'media', or 'unsupported'."""
    ext = filepath.suffix.lower()
    if ext in ACCEPTED_FORMATS:
        return "document"
    for category, extensions in ACCEPTED_MEDIA.items():
        if ext in extensions:
            return "media"
    if ext in UNSUPPORTED_FORMATS:
        return "unsupported"
    return "unsupported"  # Unknown formats are also unsupported

def discover_and_register(input_dir: str, output_dir: str = "sources") -> dict:
    """Scan input directory, classify files, assign UIDs, build registry."""
    input_path = Path(input_dir)
    sources_path = Path(output_dir)
    sources_path.mkdir(parents=True, exist_ok=True)
    
    existing_uids = set()
    registry = {}
    rejections = []
    
    # Discover all files
    all_files = [f for f in input_path.rglob("*") if f.is_file()]
    
    for filepath in all_files:
        ext = filepath.suffix.lower()
        format_class = classify_file(filepath)
        
        if format_class == "unsupported":
            rejections.append({
                "filename": filepath.name,
                "extension": ext,
                "reason": f"Format '{ext}' is not supported. Accepted: {list(ACCEPTED_FORMATS.keys())}",
                "action": "File rejected — not admitted to pipeline"
            })
            continue
        
        # Generate UID and compute checksum
        uid = generate_uid(existing_uids)
        checksum = compute_checksum(filepath)
        
        # Copy to sources/ (immutable from here)
        dest = sources_path / f"{uid}_{filepath.name}"
        shutil.copy2(filepath, dest)
        
        # Build source record
        record = SourceRecord(
            uid=uid,
            filename=filepath.name,
            extension=ext,
            checksum=checksum,
            format_class=format_class,
            path=str(dest),
            status=ProcessingStatus.REGISTERED
        )
        registry[uid] = record.model_dump()
    
    # Persist registry
    with open(sources_path / "source_registry.json", "w") as f:
        json.dump(registry, f, indent=2)
    
    # Persist rejections
    if rejections:
        with open(sources_path / "rejection_receipts.json", "w") as f:
            json.dump(rejections, f, indent=2)
    
    print(f"✅ Source Registration Complete:")
    print(f"   Documents: {sum(1 for r in registry.values() if r['format_class'] == 'document')}")
    print(f"   Media:     {sum(1 for r in registry.values() if r['format_class'] == 'media')}")
    print(f"   Rejected:  {len(rejections)}")
    
    return {"registry": registry, "rejections": rejections}

# Execute
# results = discover_and_register("/path/to/submitted/files")
```

### Pseudocode
```
FUNCTION generate_uid(existing_uids):
    characters ← uppercase_letters + digits
    REPEAT:
        uid ← random_pick(characters, length=3)
        IF uid NOT IN existing_uids:
            ADD uid TO existing_uids
            RETURN uid

FUNCTION compute_checksum(filepath):
    hasher ← new SHA256
    OPEN filepath as binary stream
    WHILE bytes available:
        READ chunk (8192 bytes)
        FEED chunk TO hasher
    RETURN hasher.hex_digest()

FUNCTION classify_file(filepath):
    extension ← lowercase(filepath.extension)
    IF extension IN ACCEPTED_FORMATS: RETURN "document"
    IF extension IN any ACCEPTED_MEDIA category: RETURN "media"
    RETURN "unsupported"

FUNCTION discover_and_register(input_directory, output_directory="sources"):
    CREATE output_directory IF NOT EXISTS
    existing_uids ← empty set
    registry ← empty map
    rejections ← empty list
    
    all_files ← SCAN input_directory recursively for all files
    
    FOR EACH file IN all_files:
        format_class ← classify_file(file)
        
        IF format_class = "unsupported":
            APPEND TO rejections: {filename, extension, reason, action="rejected"}
            CONTINUE to next file
        
        uid ← generate_uid(existing_uids)
        checksum ← compute_checksum(file)
        
        COPY file TO output_directory/{uid}_{filename}
        # ↑ THIS IS THE IMMUTABILITY POINT — original never touched again
        
        CREATE SourceRecord with uid, filename, extension, checksum, format_class, path
        STORE record IN registry keyed by uid
    
    SAVE registry AS JSON → output_directory/source_registry.json
    SAVE rejections AS JSON → output_directory/rejection_receipts.json
    
    PRINT summary counts (documents, media, rejected)
    RETURN {registry, rejections}
```

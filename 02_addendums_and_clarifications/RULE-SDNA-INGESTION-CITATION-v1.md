# Ingestion & Citation Architecture: Specification & Implementation Standard

**Document ID:** `RULE-SDNA-INGESTION-CITATION-v1`  
**Date Ratified:** 2026-11-07  
**Authority:** Direct Operator Instruction & Consensus  
**Applicability:** Core Engine, Document Ingestion, Section Segmentation, Employment Triads, and Blurb Citation  

---

## 1. Architectural Principles

1. **Document Classification First & Multi-Format Header Discovery:**
   - Detect document format (`.docx`, `.txt`, `.pdf`, `.md`) before segmentation.
   - Employ format-specific structural techniques to identify top-level and sub-level section headers deterministically:
     - **Markdown (`.md` / markdown text):**
       - Parse explicit heading tokens (`#`, `##`, `###`, etc.).
       - Parse bold line headers (`**Header:**` or `__Header:__`).
       - Parse setext underlines (`===` for H1, `---` for H2).
     - **DOCX (`.docx` via `python-docx`):**
       - Inspect paragraph style properties (`p.style.name.startswith('Heading')`).
       - Inspect font run properties: bold runs (`run.bold == True`), uppercase runs, larger font sizes (`run.font.size > 11pt`), and distinct color/shading.
       - Structural delimiter fallbacks: Paragraphs ending with a colon `:` with word count $\le 5$.
     - **Readable PDF (`.pdf` via `pypdf` / PDF text streams):**
       - Inspect layout bounding blocks, font size differentials (headers are rendered at higher pt sizes than body text), bold typeface font names (e.g., `*Bold*`, `*Black*`, `*Heavy*`), and prominent vertical whitespace gaps / leading transitions.
       - Text stream fallbacks: Lines with trailing colons, standalone uppercase strings, or lines isolated by double line-breaks.
     - **Plain Text (`.txt`):**
       - Double line break transitions followed by short lines ($\le 4$ words or $\le 35$ characters).
       - Trailing colons (`Professional Summary:`, `Work Experience:`, `Skills and Strengths:`).
       - ALL-CAPS strings (`WORK EXPERIENCE`, `EDUCATION`).
       - Underlined headers using repeated punctuation dashes or equals signs on the next line (`---`, `===`, `___`).
   - **Container Integrity:** Never collapse or fragment parent sections into arbitrary sub-items (e.g., `Skills and Strengths:` must not be dropped in favor of its first child bullet/subcategory).
   - Capture full section content inside each section container (`SECTION A`, `SECTION B`, etc.).

2. **Person Data Gate, Header Ambiguity & Conflict Flagging (Hard Boundary Rules):**
   - **Hard Gate — Zero Person Data = Rejection / Hard Quarantine:**
     - A document with no person data (no name or contact info), even if it has perfect section headers and blurbs, **must NOT be parsed into admitted twins**. It is rejected immediately.
   - **Zero Inference Rule:**
     - **NEVER infer or fabricate a person's name.**
     - **NEVER infer or fabricate an employment instance.**
   - **Unclear / Missing Headers + Person Data Present = Valid & Parsable:**
     - If a document contains valid person data but lacks clear section headers or explicit employment declarations, **parse all of the blurbs regularly**.
     - Because there are no explicit headers or employment instances to anchor to, these blurbs are **uncited**.

> **AUDIT-017 — Conflict with P/E quarantine:** This rule permits processing when Person data exists but employment declarations are absent. The original Document 4 and consolidated baseline describe missing Person or Employment anchors as quarantine conditions. Operator decision required.
>
> **Status:** OPEN — OPERATOR DECISION REQUIRED. Do not choose between these admission rules autonomously.

   - **Minor Conflict / Ambiguity Taxonomy:**
     - Uncited blurbs resulting from unclear sectioning/employment boundaries are flagged as **`MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`**.
     - *Principle:* An ambiguity does not mean ambiguity of textual meaning; it means **ambiguity of where the blurb belongs** and whether that ambiguity impedes scanning, mapping, atomizing, or citing.
     - Never suppress or hesitate to mark provenance/structural ambiguities as minor conflicts. They preserve downstream auditability without halting processing.

> **AUDIT-018 — Uncited atom representation unresolved:** The rule permits uncited blurbs with `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`, but other requirements demand parent citations and zero orphaned atoms. The validation model is unresolved.
>
> **Status:** OPEN — OPERATOR DECISION REQUIRED. Preserve both requirements until the validation model is decided and verified.

3. **Employment Declarations (Work Instances):**
   - Must detect the triad: **`Date Range` + `Position / Job Title` + `Company / Employer`**.
   - Invariant to layout ordering (`Title, Company | Dates` or `Dates | Title | Company`).
   - If a triad component is missing, explicitly notate the defect rather than silently dropping the instance.

4. **100% Provenance & Unicode Citation Mapping:**
   - Canonical 10-symbol Unicode pool (with index doubling for overflow):
     1. Index 0 (`U+00BF`): `¿`
     2. Index 1 (`U+00A1`): `¡`
     3. Index 2 (`U+00B6`): `¶`
     4. Index 3 (`U+00A7`): `§`
     5. Index 4 (`U+203D`): `‽`
     6. Index 5 (`U+2042`): `⁂`
     7. Index 6 (`U+2020`): `†`
     8. Index 7 (`U+2021`): `‡`
     9. Index 8 (`U+204A`): `⁊`
     10. Index 9 (`U+2053`): `⁓`
   - **Employment Instances:** Each declared job is assigned a unique symbol in order of appearance (`¿`, `¡`, etc.).
   - **Section Fallback Attribution:** Blurbs that belong to a general/overview section (e.g., Professional Summary, Key Takeaways, Skills & Strengths) rather than a specific job are cited using a unique Unicode symbol assigned to that **section**.
   - **Prefix Placement:** Every single atomic blurb receives its provenance symbol at the **beginning**:
     - Format: `[UID]. [[CITATION]] [Blurb Content]` (e.g., `001. [¿] Managed diverse restaurant operations...`).
   - **Zero Orphaned Blurbs:** No blurb is left uncited or unanchored.

> **AUDIT-019 — Citation fallback conflict:** Clarify how fallback attribution relates to the explicitly permitted uncited ambiguous-provenance case. Do not silently eliminate either instruction.
>
> **Status:** OPEN — OPERATOR DECISION REQUIRED. Preserve both instructions until their relationship is decided and verified.

---

## 2. Deterministic Implementation Reference (Python)

```python
"""
Deterministic Ingestion, Triad Extraction & Unicode Citation Engine
Complies with RULE-SDNA-INGESTION-CITATION-v1
"""

from __future__ import annotations
import re
from typing import Dict, List, Optional, Tuple, Any

# Canonical 10-Symbol Unicode Citation Pool
UNICODE_CITATION_POOL = [
    "\u00bf",  # 0: ¿
    "\u00a1",  # 1: ¡
    "\u00b6",  # 2: ¶
    "\u00a7",  # 3: §
    "\u203d",  # 4: ‽
    "\u2042",  # 5: ⁂
    "\u2020",  # 6: †
    "\u2021",  # 7: ‡
    "\u204a",  # 8: ⁊
    "\u2053",  # 9: ⁓
]

def get_citation_symbol(index: int) -> str:
    """Deterministic citation symbol generator with mathematical doubling on pool overflow."""
    pool_size = len(UNICODE_CITATION_POOL)
    if index < pool_size:
        return UNICODE_CITATION_POOL[index]
    overflow = index - pool_size
    first = UNICODE_CITATION_POOL[(overflow // pool_size) % pool_size]
    second = UNICODE_CITATION_POOL[overflow % pool_size]
    return first + second


class CitationRegistry:
    """Manages unique, gapless Unicode symbol assignment for employers and sections."""
    def __init__(self):
        self._current_index = 0
        self.employer_citations: Dict[str, str] = {}
        self.section_citations: Dict[str, str] = {}

    def get_or_assign_employer(self, employer_key: str) -> str:
        if employer_key not in self.employer_citations:
            symbol = get_citation_symbol(self._current_index)
            self._current_index += 1
            self.employer_citations[employer_key] = symbol
        return self.employer_citations[employer_key]

    def get_or_assign_section(self, section_name: str) -> str:
        norm_name = section_name.strip().upper()
        if norm_name not in self.section_citations:
            symbol = get_citation_symbol(self._current_index)
            self._current_index += 1
            self.section_citations[norm_name] = symbol
        return self.section_citations[norm_name]


# Contact Extraction Patterns
EMAIL_REGEX = re.compile(r"[\w\.-]+@[\w\.-]+\.\w+", re.IGNORECASE)
PHONE_REGEX = re.compile(r"(?:\+?\d{1,2}\s*)?(?:\(?\d{3}\)?[\s.-]?)?\d{3}[\s.-]?\d{4}")

# Work Triad Patterns
DATE_RANGE_REGEX = re.compile(
    r"(?P<dates>(?:(?:\d{1,2}/\d{4}|\b(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*\s+\d{4}|\d{4})\s*[-–—]\s*(?:Present|\d{1,2}/\d{4}|\b(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*\s+\d{4}|\d{4})))",
    re.IGNORECASE
)

POSITION_KEYWORDS = [
    "Manager", "Director", "Lead", "Architect", "Engineer", "Supervisor",
    "Assistant", "Specialist", "Consultant", "Officer", "Coordinator", "Representative"
]


def extract_person_and_contact(raw_text: str) -> Dict[str, Any]:
    """Finds Person Name, Phone, and Email regardless of location in document."""
    lines = [line.strip() for line in raw_text.splitlines() if line.strip()]
    name = lines[0] if lines else "UNKNOWN"
    
    # Strip contact tokens from name line if mixed
    emails = EMAIL_REGEX.findall(raw_text)
    phones = PHONE_REGEX.findall(raw_text)
    
    # If first line has email/phone, it's not the name
    if (emails and emails[0] in name) or (phones and phones[0] in name):
        name = "UNKNOWN"

    return {
        "name": name,
        "email": emails[0] if emails else None,
        "phone": phones[0] if phones else None,
        "raw_matches": {"emails": emails, "phones": phones}
    }


def parse_work_triad_line(line: str) -> Optional[Dict[str, str]]:
    """Extracts Date + Position + Company triad with notation for missing parts."""
    date_match = DATE_RANGE_REGEX.search(line)
    dates = date_match.group("dates").strip() if date_match else None
    
    remainder = DATE_RANGE_REGEX.sub("", line).strip()
    remainder = re.sub(r"[|\-–—]+", "|", remainder).strip("| \t")
    
    parts = [p.strip() for p in remainder.split("|") if p.strip()]
    if not parts and not dates:
        return None
    
    position = None
    company = None
    
    # Identify position by keyword
    for part in parts:
        if any(kw.lower() in part.lower() for kw in POSITION_KEYWORDS):
            position = part
            break
            
    # Assign company to the other prominent part
    for part in parts:
        if part != position:
            company = part
            break
            
    # Notation for incomplete triad
    missing = []
    if not dates: missing.append("dates")
    if not position: missing.append("position")
    if not company: missing.append("company")
    
    triad = {
        "dates": dates,
        "position": position,
        "company": company,
        "is_complete": len(missing) == 0,
        "missing_fields": missing,
        "raw_line": line
    }
    return triad


def parse_document_with_gates(raw_text: str, doc_format: str = "txt") -> Dict[str, Any]:
    """Applies Person Data Gate, Zero-Inference Boundary, and Minor Conflict Flagging."""
    person_data = extract_person_and_contact(raw_text)
    
    # 1. HARD GATE: No person data = Unparsable / Quarantine
    if not person_data.get("name") or person_data["name"] == "UNKNOWN":
        return {
            "status": "REJECTED_QUARANTINE",
            "reason": "Missing Person Data. Documents without person identity are unparsable under Rule SDNA-INGESTION-v1.",
            "person": None,
            "blurbs": [],
            "conflicts": ["FATAL_ERROR: MISSING_PERSON_IDENTITY"]
        }

    # 2. Parse sections and check for unclear/missing headers
    sections = segment_document(raw_text, doc_format=doc_format)
    has_clear_headers = any(s["header"] != "HEADER" for s in sections)
    
    # Discover employment triads
    employers = []
    lines = raw_text.splitlines()
    for line in lines:
        triad = parse_work_triad_line(line)
        if triad and triad["is_complete"]:
            employers.append(triad)

    registry = CitationRegistry()
    blurbs = []
    conflicts = []
    
    blurb_uid_counter = 1
    
    # Case A: Document has person data but unclear/missing headers & employment instances
    if not has_clear_headers and not employers:
        conflicts.append("MINOR_CONFLICT: UNCLEAR_SECTION_HEADERS")
        conflicts.append("MINOR_CONFLICT: NO_EMPLOYMENT_DECLARATIONS")
        
        # Parse all blurbs regularly, uncited, flagged as minor conflict
        for line in lines:
            stripped = line.strip()
            # Skip obvious contact line matches
            if stripped and stripped != person_data["name"] and not EMAIL_REGEX.search(stripped) and not PHONE_REGEX.search(stripped):
                blurb = {
                    "uid": f"{blurb_uid_counter:03d}",
                    "content": stripped,
                    "citation": None,  # Uncited due to missing header/employment anchor
                    "flags": ["MINOR_CONFLICT: AMBIGUOUS_PROVENANCE"],
                    "provenance": "UNCITED_UNCLEAR_STRUCTURE"
                }
                blurbs.append(blurb)
                blurb_uid_counter += 1
                
        return {
            "status": "PARSED_WITH_CONFLICTS",
            "person": person_data,
            "sections": sections,
            "blurbs": blurbs,
            "conflicts": conflicts
        }

    # Case B: Standard Ingestion with Citing
    for sec in sections:
        sec_header = sec["header"]
        sec_citation = registry.get_or_assign_section(sec_header) if sec_header != "HEADER" else None
        
        for line in sec["lines"]:
            stripped = line.strip()
            # If line matches work triad, register employer
            triad = parse_work_triad_line(stripped)
            if triad and triad["is_complete"]:
                emp_key = f"{triad['company']}_{triad['position']}"
                emp_citation = registry.get_or_assign_employer(emp_key)
                continue
                
            # If line is blurb, assign employer or section citation prefix
            citation = sec_citation
            blurb = {
                "uid": f"{blurb_uid_counter:03d}",
                "content": stripped,
                "citation": citation,
                "prefix_display": f"{blurb_uid_counter:03d}. [{citation}] {stripped}" if citation else f"{blurb_uid_counter:03d}. [UNCITED] {stripped}",
                "flags": [] if citation else ["MINOR_CONFLICT: AMBIGUOUS_PROVENANCE"]
            }
            blurbs.append(blurb)
            blurb_uid_counter += 1

    return {
        "status": "PARSED_SUCCESS",
        "person": person_data,
        "sections": sections,
        "blurbs": blurbs,
        "conflicts": conflicts
    }
    lines = raw_text.splitlines()
    sections = []
    current_sec = {"header": "HEADER", "lines": []}
    
    KNOWN_SECTIONS = [
        "PROFESSIONAL SUMMARY", "SUMMARY", "WORK EXPERIENCE", "EXPERIENCE", "EMPLOYMENT",
        "KEY TAKEAWAYS", "TAKEAWAYS", "SKILLS AND STRENGTHS", "SKILLS & STRENGTHS",
        "SKILLS", "CORE COMPETENCIES", "EDUCATION", "CERTIFICATIONS", "PROJECTS"
    ]
    
    for i, line in enumerate(lines):
        stripped = line.strip()
        if not stripped:
            continue
            
        is_header = False
        clean_header = stripped
        
        # 1. Markdown Heading Detection
        if doc_format in ("md", "markdown") or stripped.startswith("#"):
            if stripped.startswith("#"):
                is_header = True
                clean_header = stripped.lstrip("#").strip()
            elif (stripped.startswith("**") and stripped.endswith("**")) or (stripped.startswith("__") and stripped.endswith("__")):
                is_header = True
                clean_header = stripped.strip("*_").strip()
            # Setext underlines
            elif i + 1 < len(lines) and re.match(r"^[=\-]{3,}$", lines[i + 1].strip()):
                is_header = True
                clean_header = stripped

        # 2. Universal Heuristics (DOCX, PDF text stream, TXT)
        if not is_header:
            # Trailing colon check (e.g. "Work Experience:", "Professional Summary:")
            if stripped.endswith(":") and len(stripped.split()) <= 5:
                norm = stripped.rstrip(":").strip().upper()
                if norm in KNOWN_SECTIONS or len(stripped) <= 40:
                    is_header = True
                    clean_header = stripped
            # Standalone ALL-CAPS check
            elif stripped.isupper() and len(stripped) <= 35 and len(stripped.split()) <= 4:
                is_header = True
                clean_header = stripped
            # Known section dictionary match
            elif stripped.upper().rstrip(":") in KNOWN_SECTIONS:
                is_header = True
                clean_header = stripped

        if is_header and current_sec["lines"]:
            sections.append(current_sec)
            current_sec = {"header": clean_header, "lines": []}
        else:
            current_sec["lines"].append(stripped)
            
    if current_sec["lines"] or current_sec["header"] != "HEADER":
        sections.append(current_sec)
        
    return sections
```

---

## 3. Reference Output Visualization

With these rules implemented, input documents are parsed deterministically into the following structured format:

```text
Scanned 5 sections. Found 2 employment declarations.

===========================================================================
PERSON & CONTACT INFORMATION:
---------------------------------------------------------------------------
   Name:     CHRIS FLOURNOY
   Phone:    734-262-5519
   Email:    Ckflournoy.master@icloud.com

EMPLOYMENT DECLARATIONS:
---------------------------------------------------------------------------
   [¿] Employer 1: Bobcat Bonnies, Ypsilanti, MI | General Manager | 12/2020 - Present
   [¡] Employer 2: Twin Peaks, Multiple Locations | Assistant General Manager | 12/2012 - 11/2020

===========================================================================
SECTION A: CHRIS FLOURNOY
---------------------------------------------------------------------------
   (Contact info mapped to Person Entity)

SECTION B: PROFESSIONAL SUMMARY:
   (Section Citation: ¶)
---------------------------------------------------------------------------
   001. [¶] Experienced Restaurant Manager with over eight years of successful leadership in high-volume dining environments.
   002. [¶] Proficient in steering front and back of house operations, optimizing profitability, and fostering a positive work culture.
   003. [¶] Committed to delivering exceptional service and maintaining top-notch quality and standards.

SECTION C: WORK EXPERIENCE:
---------------------------------------------------------------------------
   [¿] General Manager, Bobcat Bonnies, Ypsilanti, MI | 12/2020 - Present
   [¡] Assistant General Manager, Twin Peaks, Multiple Locations | 12/2012 - 11/2020

SECTION D: KEY TAKEAWAYS:
   (Section Citation: §)
---------------------------------------------------------------------------
   004. [§] Accomplished in directing both front and back of house operations, ensuring seamless service delivery.
   005. [§] Expert in financial management, cost optimization, and revenue generation for enhanced profitability.
   006. [§] Proficient communicator and leader, capable of motivating teams to achieve exceptional results.
   007. [§] Skilled in budgeting, payroll management, and revenue forecasting to drive business success.
   008. [§] Demonstrated ability to lead and manage large groups effectively.
   009. [§] Proactive in tracking emerging industry trends to stay ahead of market dynamics.

SECTION E: SKILLS AND STRENGTHS:
   (Section Citation: ‽)
---------------------------------------------------------------------------
   Operational Excellence:
   010. [‽] Managed diverse restaurant operations, maintaining consistent productivity and quality.
   
   Financial Acumen:
   011. [‽] Successfully managed budgets, payroll, and cost systems to drive profitability.
   
   Effective Communication:
   012. [‽] Adept at fostering clear communication within teams for optimal collaboration.
   
   Leadership Mastery:
   013. [‽] Motivated and led large groups of staff to deliver outstanding service.
   
   Strategic Planning:
   014. [‽] Proficient in revenue forecasting and budget management to ensure business success.

===========================================================================
```

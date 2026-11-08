# AGENT.md: Build & Operating Instructions for Spatial DNA (SDNA) Compiler
**Repository:** [Spatial-DNA-Compiler-By-AG](https://github.com/digital-Alchemy-mcmg/Spatial-DNA-Compiler-By-AG)  
**System Designation:** Spatial DNA Compiler / SDNA Evidence Foundry Pipeline  
**Audience:** Autonomous AI Agents, Human Developers, and Pair Programmers  
**Status:** Authoritative Architectural Guide (v2.0.0 Baseline)  
**Implementation Status Notice:** See [`07_implementation_status/IMPLEMENTATION_INVENTORY.md`](file:///C:/AI_WORKSPACE/Spatial-DNA-Compiler-By-AG/07_implementation_status/IMPLEMENTATION_INVENTORY.md) and [`06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md`](file:///C:/AI_WORKSPACE/Spatial-DNA-Compiler-By-AG/06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md). This repository currently documents the complete architectural specification, data contracts, and operating requirements. Executable implementation modules (`step1` through `step13`, orchestrator) must be built or integrated from source before execution commands described in Part II can be run.

---

# TABLE OF CONTENTS
- [System Overview & Core Invariants](#system-overview--core-invariants)
- [PART I — BUILD INSTRUCTIONS (What to Build & What to Expect)](#part-i--build-instructions-what-to-build--what-to-expect)
  - [1. Architectural System Components](#1-architectural-system-components)
  - [2. Workbook Construction & Structure (5 Sheets)](#2-workbook-construction--structure-5-sheets)
  - [3. Complete Processing Workflow (Steps 0 through 13)](#3-complete-processing-workflow-steps-0-through-13)
  - [4. Data Structures, Schemas & Artifact Construction](#4-data-structures-schemas--artifact-construction)
  - [5. Python Implementation Modules & Code Architecture](#5-python-implementation-modules--code-architecture)
  - [6. Master Pipeline Pseudocode](#6-master-pipeline-pseudocode)
  - [7. Construction Sequence & Expected Milestones](#7-construction-sequence--expected-milestones)
- [PART II — OPERATING INSTRUCTIONS (How to Operate & What to Expect)](#part-ii--operating-instructions-how-to-operate--what-to-expect)
  - [1. Implementation Prerequisites & Pre-Flight Checks](#1-implementation-prerequisites--pre-flight-checks)
  - [2. Input & Intake Procedure](#2-input--intake-procedure)
  - [3. Step-by-Step Execution Sequence](#3-step-by-step-execution-sequence)
  - [4. Artifact Movement & Handoff Specifications](#4-artifact-movement--handoff-specifications)
  - [5. Validation Gates & Decision Points (Checks A through J)](#5-validation-gates--decision-points-checks-a-through-j)
  - [6. Expected Outputs & Verification Metrics](#6-expected-outputs--verification-metrics)
  - [7. Error Handling, Quarantining & Rejections](#7-error-handling-quarantining--rejections)
  - [8. Completion Conditions & Acceptance Criteria](#8-completion-conditions--acceptance-criteria)
- [PART III — SOURCE & IMPLEMENTATION FIDELITY REFERENCE](#part-iii--source--implementation-fidelity-reference)

---

# SYSTEM OVERVIEW & CORE INVARIANTS

The **Spatial DNA (SDNA) Compiler** transforms unstructured candidate resumes, curriculum vitae, and supporting non-textual media into a deterministic, cryptographically fingerprinted, and traversable **Evidence Foundry**. 

Downstream reasoning agents do not parse messy original PDFs or guess image contents. Instead, they interact with an immutable **Document Twin** and a 5-sheet evidence workbook where every single claim is an independently identifiable **Atomic Blurb** with 100% provenance back to its exact line, section, employer, or media manifest.

### The Five Invariants
1. **Source Immutability:** Original source files are fingerprinted via binary SHA-256 and copied into a protected `sources/` directory. They are never altered.
2. **Zero Machine Hallucination on Media:** The pipeline **never** uses computer vision, OCR, or speech-to-text models on media (images, audio, video). The human user inspects the raw file and provides the factual interpretation through the Media Evidence Manifest across all 5 annotation fields. The user's exact wording from `what_does_it_demonstrate` becomes the atomic blurb.
3. **Strict No-Deduplication:** Repeated skills, degrees, or responsibilities across employers or documents are **never merged or consolidated**. Every instance remains an independent atomic record. (Subject to Operator Contradiction C regarding file-level vs. content-level duplicates).
4. **Single-Person Corpus Boundary:** An entire workbook, directory, and package belongs strictly to **one individual**. A multi-document queue exhibiting a different person's identity triggers a fatal corpus collision quarantine.
5. **Preservation of Ambiguity:** Wording is never rewritten or smoothed over. Ambiguities are preserved verbatim and flagged as minor conflicts (`MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`).

---

# PART I — BUILD INSTRUCTIONS
## What to Build and What to Expect

### 1. Architectural System Components

When building the system, construct the following distinct component layers:
1. **Foundation & Data Contracts (`step1_contracts.py`):** Pydantic v2 schemas defining records, enums, `AtomicBlurb` (supporting both text citations and media manifest provenance), `NonBlurbBundle` (macro storage), and the 10-symbol Unicode citation doubling pool.
2. **Intake & Source Registry (`step2_registration.py`):** Unpacks ZIPs, generates 3-char UIDs (`[A-Z0-9]{3}`), computes SHA-256 binary digests, and enforces format boundaries.
3. **Media Intake (`step3_media.py`):** Catalogs non-textual files into `media_manifest.json` with `PENDING_USER` status and provides annotation persistence (`save_media_annotation`) across all 5 user fields.
4. **Multi-Format Text Extraction Engine (`step4_extraction.py`):** Dedicated extractors for `.docx` (python-docx + lxml), `.pdf` (pypdf), `.txt` (charset-normalizer), and `.md` (markdown-it-py), enforcing **Rule SDNA-HF-001** (Header/Footer Isolation).
5. **Recognition Anchors:**
   - **Person Discovery (`step5_person.py`):** Regex and spaCy NER extracting Name, Email, Phone, and Location into `PersonBundle`.
   - **Employment Discovery (`step6_employment.py`):** Date + Position + Company triads with ordering invariance (The Joanne's Fabrics rule) and Unicode citation binding.
6. **Atomization Engines:**
   - **Employment Blurbs (`step7_atomization.py`):** Splits bullet points while strictly preserving action-outcome pairs, assigning sequential UIDs and employer citations.
   - **Section Blurbs & Dual Storage (`step8_nonblurb.py`):** Full-document sweep extracting remaining sections (Summary, Skills, Education), assigning continuous UIDs, parent section citations, and `SELF_REPORTED` classifications, while simultaneously preserving macro text blocks in `NonBlurbBundle`.
7. **Twin Assembly & Quarantine (`step9_assembly.py`):** Unites all elements into `{source_uid}-000.json`. Enforces anchor gates and withholds the Completion Footer.
8. **Independent Audit Engine (`step10_verification.py`):** Runs automated Checks A through J. Check J signs the `CompletionFooter` only upon 100% pass across Checks A–I.
9. **Paired Admission Gate (`step11_admission.py`):** 5-condition gatekeeper and byte-level duplicate detection. Emits `admission_registry.json`.
10. **Evidence Workbook Builder (`step12_workbook.py`):** OpenPyXL engine rendering the styled 5-sheet workbook.
11. **Packaging & Integrity Sweep (`step13_packaging.py`):** Final 8-point audit sealing `{Person}_sDNA.zip` and `package_inventory.json`.

> **AUDIT-001 — Implementation claims:** The guide describes `save_media_annotation` and Step 9 behavior as existing functionality. No corresponding implementation source exists in this repository. Distinguish specified behavior from implemented and verified behavior.
>
> **Status:** OPEN — preserve until the implementation claim is corrected and verified against the repository.

> **AUDIT-005 — Missing modules:** The listed modules and functions are not present in the repository. Identify these as required components, not existing implementation.
>
> **Status:** OPEN — preserve until the component inventory is corrected and verified against the repository.

---

### 2. Workbook Construction & Structure (5 Sheets)

Construct an OpenPyXL workbook named `evidence_workbook.xlsx` with exactly five worksheets, styled with distinct visual fills:

```
evidence_workbook.xlsx
 ├── Sheet 1: "Evidence by Employment"  (Navy header: #1F4E79, White bold text)
 ├── Sheet 2: "Evidence by Document"    (Dark slate header: #2F5597, White bold text)
 ├── Sheet 3: "Non-Blurb Bundles"       (Amber header: #C65911, Gold fill: #FFF2CC)
 ├── Sheet 4: "Media Inventory"         (Teal header: #2E75B6, Ice fill: #DDEBF7)
 └── Sheet 5: "Admission Registry"      (Dark green header: #375623, Mint fill: #E2EFDA)
```

#### Sheet Specifications

#### Sheet 1: `Evidence by Employment`
* **Purpose:** Primary auditor view grouping atomic evidence by employer tenure.
* **Columns (8):**
  1. `Employment Label` (`Employer A`, `Employer B`)
  2. `Employer Name` (`Bobcat Bonnies`)
  3. `Atom UID` (`A7B-001`)
  4. `Atomic Blurb Content` (Action-outcome claim text)
  5. `Unicode Citation` (`¿`, `¡`)
  6. `Position / Title` (`General Manager`)
  7. `Date Range` (`12/2020 - Present`)
  8. `Source Location` (`A7B – line 14`)

#### Sheet 2: `Evidence by Document`
* **Purpose:** Linear provenance view listing all atomic blurbs across the document in exact UID order.
* **Columns (8):**
  1. `Source UID` (`A7B`)
  2. `Twin ID` (`A7B-000`)
  3. `Source Filename` (`resume.docx`)
  4. `Atom UID` (`A7B-001`, `A7B-002`... continuous progression across employment, section, and media blurbs)
  5. `Unicode Citation` (`¿`, `¡`, `¶`, `§` or `None` if media)
  6. `Attribution Label` (`Employer A`, `Section A: Summary`, or `Media: MED-K9L`)
  7. `Atomic Blurb Content` (Exact text)
  8. `Source Location` (`Paragraph index / line number / media manifest`)

#### Sheet 3: `Non-Blurb Bundles`
* **Purpose:** Macro-level inspection view of unatomized self-reported section text blocks. Dual storage ensures that while individual claims are atomized in Sheet 2, the complete cohesive macro block remains inspectable here.

> **AUDIT-002 — Section dual storage:** The description of Non-Blurb Bundles as unatomized text can imply that section content is stored only as macro bundles. The architecture requires both intact macro storage and granular atomic blurbs.
>
> **Status:** OPEN — preserve until dual storage is corrected and verified across the architecture.

* **Columns (6):**
  1. `Source UID`
  2. `Twin ID`
  3. `Bundle Type` (`Executive Summary`, `Skills & Keywords`, `Education`)
  4. `Classification` (`SELF_REPORTED` strictly enforced)
  5. `Source Location` (`Heading line anchor`)
  6. `Intact Bundle Content` (Full multi-line raw text)
* *Styling:* All data rows filled with Light Amber (`#FFF2CC`) to distinguish self-reported claims from verified employment evidence.

#### Sheet 4: `Media Inventory`
* **Purpose:** Complete catalog of all non-textual media, capturing file metadata, registration status, annotation state, and all five user-authored annotation fields.
* **Columns (11):**
  1. `Media ID` (`MED-{uid}`)
  2. `Filename` (`cert.png`)
  3. `Category` (`image`, `video`, `audio`, `unreadable_pdf`)
  4. `Registration Status` (`REGISTERED`)
  5. `Annotation Status` (`PENDING_USER` or `COMPLETE`)
  6. `What Is This` (User description of artifact)
  7. `What Does It Represent` (User explanation of context/qualification)
  8. `What Does It Demonstrate` (User interpretation -> becomes blurb)
  9. `Associated Project` (User-declared employer/project)
  10. `Attribution / Issuer` (User-declared issuing body)
  11. `SHA-256 Checksum` (Binary digest)

> **AUDIT-003 — Incomplete media inventory:** The Media Inventory omits `what_does_it_represent` and `attribution_issuer`. All five user annotation fields must be represented.
>
> **Status:** OPEN — preserve until all downstream media fields are corrected and verified.

#### Sheet 5: `Admission Registry`
* **Purpose:** Formal audit trail of admitted and rejected pairs.
* **Columns (6):**
  1. `Source UID`
  2. `Twin ID`
  3. `Admission Status` (`ADMITTED`, `REJECTED_DUPLICATE`, `QUARANTINED`)
  4. `Source Checksum` (SHA-256 hex)
  5. `Twin Checksum` (SHA-256 hex)
  6. `Original Filename`

---

### 3. Complete Processing Workflow (Steps 0 through 13)

```
[Step 0: Qualify] ──> [Step 1: Contracts] ──> [Step 2: Intake & Hash]
                                                      │
                       ┌──────────────────────────────┴──────────────────────────────┐
                       ▼                                                             ▼
              [Step 3: Media Manifest]                                     [Step 4: Text Extraction]
                       │                                                             │
                       │ (User Annotates 5 fields)                            (Rule SDNA-HF-001)
                       │                                                             │
                       │                                              ┌──────────────┴──────────────┐
                       │                                              ▼                             ▼
                       │                                    [Step 5: Person Anchor]      [Step 6: Employment Anchor]
                       │                                              │                             │
                       │                                              └──────────────┬──────────────┘
                       │                                                             ▼
                       │                                                  [Step 7: Employment Blurbs]
                       │                                                             │
                       │                                                  [Step 8: Section Blurbs & Dual Storage]
                       │                                                             │
                       └──────────────────────────────┬──────────────────────────────┘
                                                      ▼
                                       [Step 9: Twin Assembly & Quarantine]
                                                      │
                                       [Step 10: Checks A-J Independent Audit]
                                                      │ (Check J certifies CompletionFooter)
                                                      ▼
                                       [Step 11: Paired Admission Gate]
                                                      │
                                       [Step 12: Evidence Workbook Builder]
                                                      │
                                       [Step 13: Final Sweep & Sealed Packaging]
```

---

### 4. Data Structures, Schemas & Artifact Construction

Construct these core Pydantic v2 schemas in `step1_contracts.py`:

```python
from pydantic import BaseModel, Field
from typing import Optional, List, Dict, Any
from enum import Enum

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
    label: str                                   # "Employer A"
    unicode_citation: str                        # "¿"
    employer: Optional[str] = None
    position: Optional[str] = None
    dates: Optional[str] = None
    source_locations: List[str] = []
    discovery_state: DiscoveryState = DiscoveryState.FOUND

class AtomicBlurb(BaseModel):
    uid: str                                     # "A7B-001"
    content: str                                 # Unaltered text or user description
    citation: Optional[str] = None               # "¿", "¶", or None for media blurbs
    employment_label: Optional[str] = None       # "Employer A"
    section_label: Optional[str] = None          # "Section A: Summary"
    source_location: str
    is_cited: bool = False
    classification: str = "EVIDENCE"             # "EVIDENCE" vs "SELF_REPORTED"
    provenance_type: str = "TEXT"                # "TEXT" vs "MEDIA_MANIFEST"
    media_id: Optional[str] = None               # "MED-K9L" if media-derived
    media_checksum: Optional[str] = None         # SHA-256 of media

# AUDIT-004 — Incomplete AtomicBlurb schema: The media provenance field
# provenance_type: "MEDIA_MANIFEST" is missing. The media atom also requires
# its media ID, checksum, manifest source location, and null citation.
# Status: OPEN — preserve until the schema is corrected and verified.

class NonBlurbBundle(BaseModel):
    bundle_type: str                             # "Skills & Keywords", "Education"
    content: str                                 # Intact macro text block
    classification: str = "SELF_REPORTED"
    source_location: str

class CompletionFooter(BaseModel):
    source_uid: str
    twin_id: str                                 # "{source_uid}-000"
    atom_count: int
    last_atom_uid: str                           # e.g., "A7B-014"
    employment_count: int
    media_count: int
    source_checksum: str
    twin_checksum: str
    verified_at: str
    status: str = "COMPLETE"

class DocumentTwin(BaseModel):
    source_uid: str
    source_checksum: str
    person_bundle: PersonBundle
    employment_declarations: List[EmploymentDeclaration]
    atomic_blurbs: List[AtomicBlurb]
    non_blurb_info: List[NonBlurbBundle]
    media_refs: List[str] = []
    completion_footer: Optional[CompletionFooter] = None
```

---

# PART II — OPERATING INSTRUCTIONS
## How to Operate and What to Expect

> [!IMPORTANT]
> **Implementation Prerequisite:** This repository currently contains the authoritative architectural documentation and data contracts. The executable modules (`master_orchestrator.py`, `step0_environment.py` through `step13_packaging.py`, `verification_lap.py`) must be built or integrated from source before the execution commands below can be invoked. Refer to [`07_implementation_status/IMPLEMENTATION_INVENTORY.md`](file:///C:/AI_WORKSPACE/Spatial-DNA-Compiler-By-AG/07_implementation_status/IMPLEMENTATION_INVENTORY.md).

### 1. Implementation Prerequisites & Pre-Flight Checks

Once implementation modules are present in the working environment:
1. Python 3.10+ installed.
2. Required packages installed: `spacy`, `python-docx`, `pypdf`, `charset-normalizer`, `markdown-it-py`, `pydantic`, `openpyxl`.
3. spaCy model downloaded: `python -m spacy download en_core_web_sm`.

#### Pre-Flight Qualification Command
Run the compliance test to guarantee the runtime is intact:
```powershell
python verification_lap.py
```

> **AUDIT-007 — Unavailable verification command:** `verification_lap.py` is absent. The operating instructions must not represent this command as executable.
>
> **Status:** OPEN — preserve until the command is backed by repository source and independently verified.

*Expected Result:*
```
[PASS] Step 0 & 1: Environment qualification and shared data contracts verified.
...
SDNA VERIFICATION LAP COMPLETE - ALL 14 STEPS 100% COMPLIANT (Exit Code: 0)
```

---

### 2. Input & Intake Procedure

#### What the Agent Supplies
Place the candidate's files into the intake directory (`./intake`):
- Accepted Document Formats: `.docx`, `.pdf` (machine-readable), `.txt`, `.md`.
- Accepted Media Formats: `.png`, `.jpg`, `.jpeg`, `.tiff`, `.webp`, `.mp4`, `.mp3`, `.wav`.
- ZIP Packages: `.zip` archives containing multiple candidate files.

*Important Constraint:* All files in a single intake run must belong to the **same candidate**. Do not mix files from different individuals.

---

### 3. Step-by-Step Execution Sequence

#### Step A: Run Master Ingestion
Execute the master pipeline orchestrator:
```powershell
python master_orchestrator.py
```

> **AUDIT-008 — Unavailable orchestrator and incomplete operation:** `master_orchestrator.py` is absent. The media annotation lifecycle also lacks a complete operational explanation, including user submission and persistence/resumption.
>
> **Status:** OPEN — preserve until repository availability and the complete lifecycle are corrected and verified.

#### What the System Does
1. Inventories all files in `intake/`.
2. Computes binary SHA-256 fingerprints.
3. Copies raw files into `sources/{uid}_{filename}`.
4. Concurrently extracts text and builds `sources/media_manifest.json`.
5. Extracts `PersonBundle` and `EmploymentDeclarations`.
6. Generates atomic blurbs (`001`, `002`...) for employment accomplishments and remaining sections.
7. Checks the anchor gates and writes `{source_uid}-000.json`.
8. Audits the twin across Checks A through J; Check J certifies the `CompletionFooter`.
9. Generates `pipeline_outputs/evidence_workbook.xlsx` and `pipeline_outputs/{Person}_sDNA.zip`.

> **AUDIT-006 — Pseudocode versus implementation:** Media annotation and media-to-atom conversion are described as workflow operations, but their actual implementation is absent. Identify the missing persistence and conversion functionality.
>
> **Status:** OPEN — preserve until both functions are implemented and verified or clearly labeled as specifications.

#### What the Agent Sees in Console
```
============================================================================
SDNA EVIDENCE FOUNDRY - MASTER PIPELINE EXECUTION
============================================================================
[STEP 2] Discovering & Registering Source Files...
  [OK] 2 source item(s) registered.
[CONCURRENT DISPATCH] Running Step 3 (Media) & Step 4 (Text Extraction)...
  [OK] Step 3: 1 media item(s) cataloged.
  [OK] Step 4: 1 document(s) extracted.
[CONCURRENT DISPATCH] Running Step 5 (Person) & Step 6 (Employment)...
  [OK] Step 5: 1 person bundle(s) processed.
  [OK] Step 6: 2 employment declaration set(s) discovered.
[STEP 7] Constructing Atomic Blurbs...
  [OK] 14 atomic blurb(s) constructed with sequential UIDs.
[STEP 8] Extracting Non-Blurb Information...
  [OK] 4 non-blurb bundle(s) preserved intact under SELF_REPORTED.
[STEP 9] Applying Quarantine Gate & Assembling Document Twins...
  [OK] 1 twin(s) active, 0 quarantined.
[STEP 10] Running Independent Verification Checks A through J...
  [OK] 1 twin(s) fully verified; CompletionFooters certified.
[STEP 11] Running Paired Admission & Duplicate Detection...
  [OK] 1 source-twin pair(s) admitted to workbook.
[STEP 12] Building Evidence Workbook (openpyxl)...
  [OK] Evidence workbook generated: pipeline_outputs\evidence_workbook.xlsx
[STEP 13] Running Final Integrity Sweep & Packaging SDNA...
  [OK] SDNA Package created: pipeline_outputs\json_sDNA.zip
============================================================================
ALL 14 STEPS EXECUTED & SEALED SUCCESSFULLY (Exit Code: 0)
============================================================================
```

> **AUDIT-009 — Misleading Step 8 example:** The example shows macro bundles but does not show section-derived atomic blurbs. It must demonstrate both sides of dual storage.
>
> **Status:** OPEN — preserve until the example demonstrates both representations and is verified.

---

### 4. Artifact Movement & Handoff Specifications

| Stage Handoff | Artifact Written | Location on Disk | Consumed By |
|---|---|---|---|
| Step 2 $\rightarrow$ Step 3/4 | Source Registry | `sources/source_registry.json` | Media builder & extraction router |
| Step 3 $\rightarrow$ User / Step 9 | Media Manifest | `sources/media_manifest.json` | Human annotator & Twin assembler |
| Step 4 $\rightarrow$ Step 5/6 | Text Extractions | Internal in-memory dict | Person NER & Employment parser |
| Step 5–8 $\rightarrow$ Step 9 | Blurbs & Bundles | In-memory Pydantic objects | Twin assembler |
| Step 9 $\rightarrow$ Step 10 | Unverified Twin | `atomic_twins/{uid}-000.json` | Independent audit engine |
| Step 10 $\rightarrow$ Step 11 | Certified Twin | `atomic_twins/{uid}-000.json` (signed) | Paired admission gate |
| Step 11 $\rightarrow$ Step 12 | Admission Registry | `pipeline_outputs/admission_registry.json` | Workbook builder |
| Step 12 $\rightarrow$ Step 13 | Evidence Workbook | `pipeline_outputs/evidence_workbook.xlsx` | Package sealer |
| Step 13 $\rightarrow$ Final | Final Archive & Inventory | `pipeline_outputs/{Person}_sDNA.zip` | Downstream reasoning agents |

---

### 5. Validation Gates & Decision Points (Checks A through J)

During Step 10, the agent expects the system to audit the twin against 10 strict criteria:

* **Check A (Source Integrity):** Re-hashes source file on disk; fails if SHA-256 does not match registry.
* **Check B (Extraction Fidelity):** Re-extracts text using native library; fails if numbers or text diverge.
* **Check C (Discovery Coverage):** Confirms P and E anchors have explicit statuses (`FOUND`, `NOT_FOUND`).
* **Check D (Region Eligibility / Rule SDNA-HF-001):** Fails if any non-contact header/footer text appears in blurbs.
* **Check E (Atomic Content Fidelity):** Confirms every metric, percentage, or currency figure in a blurb exists in the source text.
* **Check F (Citation Integrity):** Confirms that for text blurbs (`provenance_type == "TEXT"`), Unicode citations resolve to declared employers or sections in that twin. For media blurbs (`provenance_type == "MEDIA_MANIFEST"`), confirms `citation is None`.
* **Check G (Identity & Numbering):** Confirms UID format (`3-char`), twin format (`{uid}-000`), and gapless continuous blurb sequence (`001`, `002`...).
* **Check H (Attribution & Classification):**
  - *Channel 1 (Media Manifest):* Confirms `media_id` is present, `media_checksum` matches manifest, `citation is None`, and `classification == "EVIDENCE"`.
  - *Channel 2 (Text Attribution):* Confirms section blurbs are `SELF_REPORTED`, employment blurbs are NOT `SELF_REPORTED`, citation matches exact parent section/employer, and macro containers have no atomic UIDs.
* **Check I (Media Inventory):** Confirms media IDs match `media_manifest.json` and audits all 5 user fields when `COMPLETE`.
* **Check J (Gatekeeper Certification):**
  - **Decision:** If Checks A–I are ALL `True`, write `CompletionFooter` with status `COMPLETE`.
  - If ANY check failed, withhold footer; twin cannot be admitted.

> **AUDIT-010 — Verification conflicts:** Check F requires employer or section citation resolution, but media atoms have `citation: null`. Check I also needs to account for the media annotation fields and annotation state.
>
> **Status:** OPEN — preserve until the verification branches are reconciled and verified across all applicable checks.

---

### 6. Expected Outputs & Verification Metrics

At the conclusion of a successful run, verify the existence of these exact files:
1. `pipeline_outputs/evidence_workbook.xlsx` (OpenPyXL 5-sheet workbook).
2. `pipeline_outputs/{Person}_sDNA.zip` (Sealed ZIP archive).
3. `pipeline_outputs/package_inventory.json` (Inventory of all zipped files with SHA-256 and byte lengths).
4. `pipeline_outputs/admission_registry.json` (Admission status for each pair).
5. `atomic_twins/{source_uid}-000.json` (The signed Document Twin containing the certified `completion_footer`).

---

### 7. Error Handling, Quarantining & Rejections

#### Scenario 1: Unsupported File Format (e.g. `schedule.xlsx`)
- **System Action:** Does not crash; does not silently drop.
- **Result:** Emits explicit machine-readable rejection receipt in the source registry. Excluded from twin assembly.

#### Scenario 2: Missing Person Anchor (No Name/Contact Data)
- **System Action:** Hard Gate triggers. Document is halted at Step 9.
- **Result:** Moved to `quarantined/{source_uid}-000.json` with `quarantine_reason = "Missing Person Data"`. Footer withheld.

#### Scenario 3: Missing Employment Anchor (No Work Declarations)
- **System Action:** Evaluated per operator decision on Contradiction A. If structural fallback applies, section-attributed atomization proceeds under `SELF_REPORTED`. If dual gate applies, document is quarantined.

> **AUDIT-011 — Conflicting quarantine rules:** Scenario 3 permits processing when Person data exists but employment declarations are missing. Step 9 describes P/E quarantine. These instructions conflict and require an operator decision.
>
> **Status:** OPEN — OPERATOR DECISION REQUIRED. Do not resolve this conflict autonomously.

#### Scenario 4: Byte-Identical Duplicate File Submitted
- **System Action:** Evaluated per operator decision on Contradiction C. If Step 11 duplicate rejection applies, second file is flagged `REJECTED_DUPLICATE` in `admission_registry.json`.

---

### 8. Completion Conditions & Acceptance Criteria

An operating run is declared **100% COMPLETE & VALID** when:
1. The orchestrator exits with code `0`.
2. Every admitted twin in `atomic_twins/` has a non-null `completion_footer` signed by Check J.
3. `evidence_workbook.xlsx` opens cleanly and displays all 5 worksheets with populated rows and styling fills.
4. `{Person}_sDNA.zip` contains the source document, twin JSON, evidence workbook, and media manifest.
5. All verification checks exit with 0 errors.

---

# PART III — SOURCE & IMPLEMENTATION FIDELITY REFERENCE

This guide reflects the exact, ratified implementation recorded in:

> **AUDIT-012 — Unsupported implementation-complete claim:** The guide claims to reflect the ratified implementation despite the absence of implementation source. Distinguish architectural completeness from implementation readiness.
>
> **Status:** OPEN — preserve until the claim is corrected and independently verified.

- [`01_original_reference_docs/`](01_original_reference_docs/) (Documents 1–5, build specs, and initial contracts).
- [`02_addendums_and_clarifications/RULE-SDNA-INGESTION-CITATION-v1.md`](02_addendums_and_clarifications/RULE-SDNA-INGESTION-CITATION-v1.md) (Ingestion, work triads, and citation doubling pool).
- [`02_addendums_and_clarifications/RULE-SDNA-ARCHITECTURE-ADDENDUM-v1.md`](02_addendums_and_clarifications/RULE-SDNA-ARCHITECTURE-ADDENDUM-v1.md) (Full-document sweep, section atomization, continuous UIDs, cohesive education bundles, and hardened Check H).
- [`02_addendums_and_clarifications/RULE-SDNA-MEDIA-PROVENANCE-v1.md`](02_addendums_and_clarifications/RULE-SDNA-MEDIA-PROVENANCE-v1.md) (Human-only media interpretation, blurb promotion from `what_does_it_demonstrate`, and manifest provenance).
- [`04_delta_analysis/DELTA_ANALYSIS.md`](04_delta_analysis/DELTA_ANALYSIS.md) (Traceability matrix documenting where legacy requirements were superseded).
- [`05_consolidated_architecture_baseline/CONSOLIDATED_ARCHITECTURE_BASELINE.md`](05_consolidated_architecture_baseline/CONSOLIDATED_ARCHITECTURE_BASELINE.md) (The master consolidated architecture baseline v2.0.0).
- [`06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md`](06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md) (Formal documentation of Contradictions A, B, and C).
- [`07_implementation_status/IMPLEMENTATION_INVENTORY.md`](07_implementation_status/IMPLEMENTATION_INVENTORY.md) (Complete audit of pipeline components and repository implementation status).

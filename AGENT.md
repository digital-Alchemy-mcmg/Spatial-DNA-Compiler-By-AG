# AGENT.md: Build & Operating Instructions for Spatial DNA (SDNA) Compiler
**Repository:** [Spatial-DNA-Compiler-By-AG](https://github.com/digital-Alchemy-mcmg/Spatial-DNA-Compiler-By-AG)  
**System Designation:** Spatial DNA Compiler / SDNA Evidence Foundry Pipeline  
**Audience:** Autonomous AI Agents, Human Developers, and Pair Programmers  
**Status:** Authoritative Architectural Guide (v2.0.0 Baseline)  

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
  - [1. Operating Environment & Pre-Flight Checks](#1-operating-environment--pre-flight-checks)
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
2. **Zero Machine Hallucination on Media:** The pipeline **never** uses computer vision, OCR, or speech-to-text models on media (images, audio, video). The human user inspects the raw file and provides the factual interpretation through the Media Evidence Manifest. That user description becomes the atomic blurb.
3. **Strict No-Deduplication:** Repeated skills, degrees, or responsibilities across employers or documents are **never merged or consolidated**. Every instance remains an independent atomic record.
4. **Single-Person Corpus Boundary:** An entire workbook, directory, and package belongs strictly to **one individual**. A multi-document queue exhibiting a different person's identity triggers a fatal corpus collision quarantine.
5. **Preservation of Ambiguity:** Wording is never rewritten or smoothed over. Ambiguities are preserved verbatim and flagged as minor conflicts (`MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`).

---

# PART I — BUILD INSTRUCTIONS
## What to Build and What to Expect

### 1. Architectural System Components

When building the system, construct the following distinct component layers:
1. **Foundation & Data Contracts (`step1_contracts.py`):** Pydantic v2 schemas defining records, enums, and the 10-symbol Unicode citation doubling pool.
2. **Intake & Source Registry (`step2_registration.py`):** Unpacks ZIPs, generates 3-char UIDs (`[A-Z0-9]{3}`), computes SHA-256 binary digests, and enforces format boundaries.
3. **Media Intake (`step3_media.py`):** Catalogs non-textual files into `media_manifest.json` with `PENDING_USER` status and provides annotation persistence (`save_media_annotation`).
4. **Multi-Format Text Extraction Engine (`step4_extraction.py`):** Dedicated extractors for `.docx` (python-docx + lxml), `.pdf` (pypdf), `.txt` (charset-normalizer), and `.md` (markdown-it-py), enforcing **Rule SDNA-HF-001** (Header/Footer Isolation).
5. **Recognition Anchors:**
   - **Person Discovery (`step5_person.py`):** Regex and spaCy NER extracting Name, Email, Phone, and Location into `PersonBundle`.
   - **Employment Discovery (`step6_employment.py`):** Date + Position + Company triads with ordering invariance (The Joanne's Fabrics rule) and Unicode citation binding.
6. **Atomization Engines:**
   - **Employment Blurbs (`step7_atomization.py`):** Splits bullet points while strictly preserving action-outcome pairs, assigning sequential UIDs and employer citations.
   - **Section Blurbs (`step8_nonblurb.py`):** Full-document sweep extracting remaining sections (Summary, Skills, Education), assigning continuous UIDs, parent section citations, and `SELF_REPORTED` classifications.
7. **Twin Assembly & Quarantine (`step9_assembly.py`):** Unites all elements into `{source_uid}-000.json`. Enforces P/E quarantine and withholds the Completion Footer.
8. **Independent Audit Engine (`step10_verification.py`):** Runs automated Checks A through J. Check J signs the `CompletionFooter` only upon 100% pass across Checks A–I.
9. **Paired Admission Gate (`step11_admission.py`):** 5-condition gatekeeper and byte-level duplicate detection. Emits `admission_registry.json`.
10. **Evidence Workbook Builder (`step12_workbook.py`):** OpenPyXL engine rendering the styled 5-sheet workbook.
11. **Packaging & Integrity Sweep (`step13_packaging.py`):** Final 8-point audit sealing `{Person}_sDNA.zip` and `package_inventory.json`.

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
  4. `Atom UID` (`A7B-001`, `A7B-002`... continuous progression)
  5. `Unicode Citation` (`¿`, `¡`, `¶`, `§` or `None` if media)
  6. `Attribution Label` (`Employer A` or `Section A: Summary`)
  7. `Atomic Blurb Content` (Exact text)
  8. `Source Location` (`Paragraph index / line number`)

#### Sheet 3: `Non-Blurb Bundles`
* **Purpose:** Macro-level inspection view of unatomized self-reported section text blocks.
* **Columns (6):**
  1. `Source UID`
  2. `Twin ID`
  3. `Bundle Type` (`Executive Summary`, `Skills & Keywords`, `Education`)
  4. `Classification` (`SELF_REPORTED` strictly enforced)
  5. `Source Location` (`Heading line anchor`)
  6. `Intact Bundle Content` (Full multi-line raw text)
* *Styling:* All data rows filled with Light Amber (`#FFF2CC`) to distinguish self-reported claims from verified employment evidence.

#### Sheet 4: `Media Inventory`
* **Purpose:** Complete catalog of all non-textual media and user annotations.
* **Columns (10):**
  1. `Media ID` (`MED-{uid}`)
  2. `Filename` (`cert.png`)
  3. `Category` (`image`, `video`, `audio`, `unreadable_pdf`)
  4. `File Type` (`.png`)
  5. `Registration Status` (`REGISTERED`)
  6. `Annotation Status` (`PENDING_USER` or `COMPLETE`)
  7. `What Is This` (User description)
  8. `What Does It Demonstrate` (User interpretation -> becomes blurb)
  9. `Associated Project` (User-declared employer/project)
  10. `SHA-256 Checksum` (Binary digest)

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
                       │ (User Annotates)                                     (Rule SDNA-HF-001)
                       │                                                             │
                       │                                              ┌──────────────┴──────────────┐
                       │                                              ▼                             ▼
                       │                                    [Step 5: Person Anchor]      [Step 6: Employment Anchor]
                       │                                              │                             │
                       │                                              └──────────────┬──────────────┘
                       │                                                             ▼
                       │                                                  [Step 7: Employment Blurbs]
                       │                                                             │
                       │                                                  [Step 8: Section Blurbs]
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
    citation: Optional[str] = None               # "¿", "¶", or None for media
    employment_label: Optional[str] = None       # "Employer A"
    section_label: Optional[str] = None          # "Section A: Summary"
    source_location: str
    is_cited: bool = False
    classification: str = "EVIDENCE"             # "EVIDENCE" vs "SELF_REPORTED"
    media_id: Optional[str] = None               # "MED-K9L" if media-derived
    media_checksum: Optional[str] = None         # SHA-256 of media

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
    pending_annotations: int
    status: ProcessingStatus = ProcessingStatus.COMPLETE
    content_checksum: str

class DocumentTwin(BaseModel):
    twin_id: str                                 # "A7B-000"
    source_uid: str                              # "A7B"
    source_checksum: str
    person_bundle: Optional[PersonBundle] = None
    employment_declarations: List[EmploymentDeclaration] = []
    atomic_blurbs: List[AtomicBlurb] = []
    non_blurb_info: List[NonBlurbBundle] = []
    media_refs: List[str] = []
    completion_footer: Optional[CompletionFooter] = None
```

---

### 5. Python Implementation Modules & Code Architecture

Organize the build into modular, standalone Python files:
- `step1_contracts.py`: Authoritative data contracts, constants, and the `get_citation_symbol(index)` generator.
- `step2_registration.py`: Directory scanner, ZIP unpacker, SHA-256 calculator, UID generator (`[A-Z0-9]{3}`), and source copy.
- `step3_media.py`: Manifest builder (`detect_media_and_build_manifest`), PDF reclassifier, and user annotation persistence (`save_media_annotation`).
- `step4_extraction.py`: DOCX, PDF, TXT, MD extractors enforcing Rule SDNA-HF-001 header/footer isolation.
- `step5_person.py`: Dual-source scanner (body + isolated HF) executing regex and spaCy NER for `PersonBundle`.
- `step6_employment.py`: Work instance triad parser with ordering invariance, ORG candidate heuristics, and citation pool assignment.
- `step7_atomization.py`: Employment region isolator, bullet/sentence splitter, action-outcome preserver, and sequential UID generator.
- `step8_nonblurb.py`: Section segmenter, delimited skill splitter, education bundle preserver, continuous UID allocator, and dual storage.
- `step9_assembly.py`: Twin constructor, P/E quarantine gatekeeper, and footer withholding logic.
- `step10_verification.py`: Independent 10-check audit engine (Checks A through J) and `CompletionFooter` signer.
- `step11_admission.py`: 5-condition pair validator, byte-level duplicate detector, and admission registry writer.
- `step12_workbook.py`: OpenPyXL builder generating the 5 styled worksheets.
- `step13_packaging.py`: 8-point final sweep, ZIP sealer, and `package_inventory.json` writer.
- `master_orchestrator.py`: Multi-threaded runner executing the entire pipeline concurrently and logging milestones.

---

### 6. Master Pipeline Pseudocode

```python
def run_sdna_pipeline(input_directory, output_directory):
    # Step 0: Qualification
    qualify_runtime_dependencies()

    # Step 2: Intake & Registration
    registry, rejections = discover_and_register(input_directory, "sources")

    # Step 3 & 4: Concurrent Media & Document Extraction
    with ThreadPoolExecutor() as pool:
        f_media = pool.submit(detect_media_and_build_manifest, registry, "sources")
        f_text = pool.submit(extract_all_documents, registry, "sources")
        media_manifest = f_media.result()
        extractions = f_text.result()

    # User Media Annotation Hook (if annotations submitted)
    apply_user_media_annotations(media_manifest)

    # Step 5 & 6: Concurrent Anchor Discovery
    with ThreadPoolExecutor() as pool:
        f_person = pool.submit(discover_all_persons, extractions)
        f_emp = pool.submit(discover_all_employment, extractions)
        person_bundles = f_person.result()
        employment_data = f_emp.result()

    # Step 7: Employment Atomic Blurbs (UIDs 001..N)
    employment_blurbs = construct_employment_blurbs(extractions, employment_data)

    # Step 8: Section-Attributed Atomization (Continuous UIDs N+1..M)
    section_blurbs, macro_bundles = construct_section_blurbs(extractions, employment_data, employment_blurbs)

    # Convert Completed Media Annotations to Blurbs (Continuous UIDs M+1..P)
    media_blurbs = construct_media_atomic_blurbs(media_manifest, section_blurbs)

    all_atomic_blurbs = merge_blurbs(employment_blurbs, section_blurbs, media_blurbs)

    # Step 9: Assemble Twins & Apply P/E Quarantine Gate
    twins, quarantined = assemble_document_twins(
        registry, person_bundles, employment_data, all_atomic_blurbs, macro_bundles, media_manifest
    )

    # Step 10: Independent Verification (Checks A through J)
    for twin in twins:
        results = run_checks_a_through_i(twin, registry, extractions, media_manifest)
        if all(r.passed for r in results):
            twin.completion_footer = check_j_sign_completion_footer(twin, results)

    # Step 11: Paired Admission Gate & Duplicate Check
    admitted_pairs, rejected_pairs = run_paired_admission(registry, twins)

    # Step 12: Build 5-Sheet Evidence Workbook
    workbook_path = build_evidence_workbook(admitted_pairs, macro_bundles, media_manifest, output_directory)

    # Step 13: 8-Point Integrity Sweep & Final ZIP Sealing
    package_path = seal_sdna_package(admitted_pairs, workbook_path, media_manifest, output_directory)

    return {"status": "SUCCESS", "workbook": workbook_path, "package": package_path}
```

---

### 7. Construction Sequence & Expected Milestones

Build the codebase in this strict dependency order:
1. **Milestone 1 (Contracts):** `step1_contracts.py` passes all Pydantic model instantiations; Unicode citation doubling test passes up to index 100.
2. **Milestone 2 (Registration & Intake):** `step2_registration.py` correctly unpacks test ZIPs, generates 3-character UIDs without collisions, computes SHA-256 hashes, and writes rejection receipts for `.xlsx` files.
3. **Milestone 3 (Extraction & Anchors):** `step4_extraction.py` routes DOCX, PDF, TXT, and MD files; Rule SDNA-HF-001 discards decorative header/footer lines while isolating contact text. `step5_person.py` and `step6_employment.py` successfully populate `PersonBundle` and `EmploymentDeclaration`.
4. **Milestone 4 (Unified Atomization):** `step7_atomization.py` and `step8_nonblurb.py` produce continuous sequential UIDs (`001`, `002`, `003`...) across both employment and non-employment sections, splitting delimited skills and keeping education as cohesive bundles.
5. **Milestone 5 (Verification & Sealing):** `step10_verification.py` executes Checks A–J; Check J refuses to sign if any prior check fails. Step 12 outputs all 5 worksheets with correct header colors and cell fills. Step 13 seals `{Person}_sDNA.zip`.

---

# PART II — OPERATING INSTRUCTIONS
## How to Operate It and What to Expect

### 1. Operating Environment & Pre-Flight Checks

#### System Requirements
- **OS:** Windows 10/11 (or Linux/macOS with standard Python 3.12).
- **Python Runtime:** Python 3.12 (Local path: `python` or `C:\Users\...\Python312\python.exe`).
- **Network Dependency:** **ZERO.** Must operate completely offline.
- **Directories Required:**
  - `intake/`: Where the agent places raw resume and media files.
  - `sources/`: Created automatically by Step 2 (immutable copy).
  - `atomic_twins/`: Created automatically by Step 9 (active twins).
  - `quarantined/`: Created automatically by Step 9 (quarantine records).
  - `pipeline_outputs/`: Where the Excel workbook and ZIP package are delivered.

#### Pre-Flight Qualification Command
Run the compliance test to guarantee the runtime is intact:
```powershell
python verification_lap.py
```
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

#### What the System Does
1. Inventories all files in `intake/`.
2. Computes binary SHA-256 fingerprints.
3. Copies raw files into `sources/{uid}_{filename}`.
4. Concurrently extracts text and builds `sources/media_manifest.json`.
5. Extracts `PersonBundle` and `EmploymentDeclarations`.
6. Generates atomic blurbs (`001`, `002`...) for employment accomplishments and remaining sections.
7. Checks the P/E quarantine gate and writes `{source_uid}-000.json`.
8. Audits the twin across Checks A through J; Check J certifies the `CompletionFooter`.
9. Generates `pipeline_outputs/evidence_workbook.xlsx` and `pipeline_outputs/{Person}_sDNA.zip`.

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
* **Check F (Citation Integrity):** Confirms every blurb's Unicode citation resolves to that specific twin's declarations.
* **Check G (Identity & Numbering):** Confirms UID format (`3-char`), twin format (`{uid}-000`), and gapless blurb sequence (`001`, `002`...).
* **Check H (Attribution & Classification):** Confirms section blurbs are `SELF_REPORTED`, employment blurbs are NOT `SELF_REPORTED`, citation matches exact parent, and macro containers have no atomic UIDs.
* **Check I (Media Inventory):** Confirms media IDs match `media_manifest.json` and checksums are valid.
* **Check J (Gatekeeper Certification):**
  - **Decision:** If Checks A–I are ALL `True`, write `CompletionFooter` with status `COMPLETE`.
  - If ANY check failed, withhold footer; twin cannot be admitted.

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
- **System Action:** If headers and employers are missing but Person data exists, parses blurbs regularly under `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE` (uncited). If neither exists, quarantined.

#### Scenario 4: Byte-Identical Duplicate File Submitted
- **System Action:** Step 11 matches binary SHA-256 against already admitted sources.
- **Result:** Second file is flagged `REJECTED_DUPLICATE` in `admission_registry.json`. Excluded from workbook to prevent data pollution.

---

### 8. Completion Conditions & Acceptance Criteria

An operating run is declared **100% COMPLETE & VALID** when:
1. The orchestrator exits with code `0`.
2. Every admitted twin in `atomic_twins/` has a non-null `completion_footer` signed by Check J.
3. `evidence_workbook.xlsx` opens cleanly and displays all 5 worksheets with populated rows and styling fills.
4. `json_sDNA.zip` contains the source document, twin JSON, evidence workbook, and media manifest.
5. `verify_telemetry.py` (if running under ratchet) exits with `0` hard findings.

---

# PART III — SOURCE & IMPLEMENTATION FIDELITY REFERENCE

This guide reflects the exact, ratified implementation recorded in:
- [`01_original_reference_docs/`](01_original_reference_docs/) (Documents 1–5, build specs, and initial contracts).
- [`02_addendums_and_clarifications/RULE-SDNA-INGESTION-CITATION-v1.md`](02_addendums_and_clarifications/RULE-SDNA-INGESTION-CITATION-v1.md) (Ingestion, work triads, and citation doubling pool).
- [`02_addendums_and_clarifications/RULE-SDNA-ARCHITECTURE-ADDENDUM-v1.md`](02_addendums_and_clarifications/RULE-SDNA-ARCHITECTURE-ADDENDUM-v1.md) (Full-document sweep, section atomization, continuous UIDs, cohesive education bundles, and hardened Check H).
- [`02_addendums_and_clarifications/RULE-SDNA-MEDIA-PROVENANCE-v1.md`](02_addendums_and_clarifications/RULE-SDNA-MEDIA-PROVENANCE-v1.md) (Human-only media interpretation, blurb promotion from `what_does_it_demonstrate`, and manifest provenance).
- [`04_delta_analysis/DELTA_ANALYSIS.md`](04_delta_analysis/DELTA_ANALYSIS.md) (Traceability matrix documenting where legacy non-blurb blob requirements were superseded by section atomization).
- [`05_consolidated_architecture_baseline/CONSOLIDATED_ARCHITECTURE_BASELINE.md`](05_consolidated_architecture_baseline/CONSOLIDATED_ARCHITECTURE_BASELINE.md) (The master consolidated architecture baseline v2.0.0).

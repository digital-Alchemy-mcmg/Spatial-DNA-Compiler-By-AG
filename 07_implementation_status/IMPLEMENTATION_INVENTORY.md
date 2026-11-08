# Spatial DNA Compiler: Implementation & Verification Inventory

**Date of Audit:** 2026-11-08  
**Audit Purpose:** Explicitly document what is specified vs. implemented vs. tested vs. verified in this repository.

---

## 1. Concrete Inventory of Architectural Components

| Component / Step | Required Module | Architectural Responsibility | Required Inputs | Required Outputs | Dependencies | Exists in SDNA Repo? | Implementation Status in SDNA Repo |
|---|---|---|---|---|---|---|---|
| **Step 0 & 1: Contracts** | `step1_contracts.py` | Pydantic v2 schemas, constants, Unicode pool doubling generator | None | Data models, enums, `get_citation_symbol()` | `pydantic` | **NO** | *Contract/Pseudocode Defined Only* |
| **Step 2: Registration** | `step2_registration.py` | Source intake, ZIP unpack, SHA-256 binary hash, 3-char UID (`[A-Z0-9]{3}`) | Intake files | `sources/{uid}_{name}`, `source_registry.json`, rejection receipts | `hashlib`, `shutil` | **NO** | *Architecturally Specified Only* |
| **Step 3: Media Manifest** | `step3_media.py` | Catalog non-text files, build `media_manifest.json`, save user annotations | `source_registry.json`, user input | `media_manifest.json`, updated entries | `json`, `pathlib` | **NO** | *Architecturally Specified Only* |
| **Step 4: Extraction Engine** | `step4_extraction.py` | Multi-format text extractors (DOCX, PDF, TXT, MD) + Rule SDNA-HF-001 | Registered sources | Extracted body text, isolated headers/footers | `python-docx`, `pypdf`, `charset-normalizer`, `markdown-it-py` | **NO** | *Architecturally Specified Only* |
| **Step 5: Person Anchor** | `step5_person.py` | Extract Name, Phone, Email, Location; evaluate Hard Person Gate | Extracted text | `PersonBundle` JSON | `spacy`, `en_core_web_sm`, `re` | **NO** | *Architecturally Specified Only* |
| **Step 6: Employment Anchor** | `step6_employment.py` | Extract work triads with ordering invariance; assign Unicode citations | Extracted body text | `EmploymentDeclaration` list | `spacy`, `re` | **NO** | *Architecturally Specified Only* |
| **Step 7: Employment Blurbs** | `step7_atomization.py` | Atomize employment accomplishments preserving action-outcome verbs | Employment regions, declarations | `AtomicBlurb` list (UIDs 001..N) | `spacy`, `re` | **NO** | *Architecturally Specified Only* |
| **Step 8: Section Atomization** | `step8_nonblurb.py` | Full-document sweep, atomize Summary, Skills, Education; dual storage | Non-employment body text, section map | `AtomicBlurb` list (UIDs N+1..M), `NonBlurbBundle` list | `re` | **NO** | *Architecturally Specified Only* |
| **Step 9: Twin Assembly** | `step9_assembly.py` | Assemble `{source_uid}-000.json`, enforce P/E gate, withhold footer | Anchors, blurbs, bundles, media refs | `atomic_twins/{uid}-000.json` or `quarantined/` | `json` | **NO** | *Architecturally Specified Only* |
| **Step 10: Verification Engine** | `step10_verification.py` | Independent Checks A–J; Check J certifies `CompletionFooter` | Twin JSON, sources on disk, manifest | `ValidationResult` list, signed `CompletionFooter` | `hashlib` | **NO** | *Contract/Pseudocode Defined Only* |
| **Step 11: Admission Gate** | `step11_admission.py` | Verify 5 conditions, check byte-level duplicates | Registry, verified twins | `admission_registry.json` | `json` | **NO** | *Architecturally Specified Only* |
| **Step 12: Workbook Builder** | `step12_workbook.py` | Build OpenPyXL 5-sheet evidence workbook with styling | Admitted twins, manifest, registry | `evidence_workbook.xlsx` | `openpyxl` | **NO** | *Architecturally Specified Only* |
| **Step 13: Packaging** | `step13_packaging.py` | Final 8-point sweep, seal ZIP archive, build inventory | Admitted twins, workbook, sources | `{Person}_sDNA.zip`, `package_inventory.json` | `zipfile`, `hashlib` | **NO** | *Architecturally Specified Only* |
| **Master Orchestrator** | `master_orchestrator.py` | Multi-threaded coordinator running Steps 2–13 | Input directory path | Console logs, full pipeline outputs | `concurrent.futures` | **NO** | *Architecturally Specified Only* |
| **Verification Lap** | `verification_lap.py` | End-to-end audit harness checking all 14 steps | None (self-contained suite) | Test assertion reports | `unittest` | **NO** | *Architecturally Specified Only* |

---

## 2. Summary of Implementation Reality

1. **Current Code Status in SDNA Repository:**
   - **Zero executable Python implementation modules exist in this repository.**
   - All modules listed above (`step1` through `step13`, `master_orchestrator.py`, `verification_lap.py`) exist in external workspace folders (`atrium/intake/wbcompiler`, `WBcompiler_coordination/deliveries`, `LAB-T`), but **have not been placed or committed into this repository**.
2. **Current Document Claims vs. Reality:**
   - `AGENT.md` previously instructed operators to run `python master_orchestrator.py` and `python verification_lap.py`. Because those files do not exist in this repository, running those commands would immediately fail with `No such file or directory`.
   - This contradiction is resolved in `AGENT.md` by explicitly labeling the code as **to be built** in Part I, and clearly marking the operating commands in Part II as **requiring implementation before execution**.

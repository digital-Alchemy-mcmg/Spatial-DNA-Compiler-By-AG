# Consolidated Architectural Baseline: Spatial DNA Compiler

**Baseline Version:** 2.0.0 (Consolidated)  
**Date Ratified:** 2026-11-08  
**Governing Documents:** Documents 1–5, Addendum v1 (`RULE-SDNA-ARCHITECTURE-ADDENDUM-v1`), Ingestion Rule v1 (`RULE-SDNA-INGESTION-CITATION-v1`), Media Rule v1 (`RULE-SDNA-MEDIA-PROVENANCE-v1`).  

---

## 1. Executive Summary & Architecture Goals

The **Spatial DNA (SDNA) Compiler** is a deterministic, offline-capable pipeline that transforms unstructured and semi-structured candidate documents (`.docx`, `.pdf`, `.txt`, `.md`) and supporting media into a verified, cryptographically audited **Evidence Foundry**.

Downstream reasoning agents do not parse raw resumes or guess image contents. Instead, they traverse a structured **Document Twin** containing individually identified, cited atomic claims whose origin, line number, employer/section parent, and file integrity are 100% traceable.

---

## 2. Global Invariants

1. **Source Immutability:** Original files are fingerprinted via binary SHA-256 and copied to `sources/`. They are never modified.
2. **Single-Person Corpus:** An entire submission directory or workbook belongs to exactly one individual. Multi-document person mismatches trigger an immediate fatal corpus collision quarantine.
3. **No Hallucination & No AI Media Interpretation:** The pipeline never uses vision or OCR models. Raw media is interpreted solely by the human user through the Media Evidence Manifest.
4. **No Content Deduplication:** Identical skills, responsibilities, or credentials appearing across employers or documents remain distinct, separately attributed atomic records.
5. **Preservation of Ambiguity:** Wording is never synthesized or smoothed over. Ambiguities are preserved verbatim and flagged as minor conflicts for downstream evaluation.

---

## 3. The 14-Step Deterministic Pipeline

### Phase 1: Foundation
* **Step 0 (Environment Qualification):** Verifies all core libraries (`spacy` with `en_core_web_sm`, `python-docx`, `pypdf`, `charset-normalizer`, `markdown-it-py`, `pydantic`, `openpyxl`).
* **Step 1 (Data Contracts & Constants):** Establishes the Pydantic v2 data models, format mappings, and the canonical 10-symbol Unicode citation pool (`¿`, `¡`, `¶`, `§`, `‽`, `⁂`, `†`, `‡`, `⁊`, `⁓`) with index doubling for overflow.
* **Step 2 (Source Discovery & Registration):** Unpacks ZIPs, computes SHA-256, assigns collision-checked 3-character alphanumeric UIDs (`[A-Z0-9]{3}`), classifies into `document`, `media`, or `unsupported`, and emits explicit rejection receipts for unsupported formats (`.csv`, `.xls`, `.xlsx`).

### Phase 2: Ingestion & Recognition Anchors
* **Step 3 (Media Detection & Manifest):** Discovers media files, generates `MED-{uid}`, records SHA-256, and initializes `media_manifest.json` with `PENDING_USER` status and blank annotation fields.
* **Step 4 (Text Extraction Engine):** Employs format-specific engines for DOCX, PDF, TXT, and Markdown. Enforces **Rule SDNA-HF-001**: headers and footers are strictly isolated; only contact info is extracted, all other text is discarded. Reclassifies unreadable/scanned PDFs to the media manifest.
* **Step 5 (Person Bundle Discovery - Anchor 1):** Extracts Name, Phone, Email, Location, and Links via regex and spaCy NER. Enforces the **Hard Gate**: documents without person identity are unparsable and quarantined.
* **Step 6 (Employment Declaration Discovery - Anchor 2):** Discovers Date + Position + Company triads with **ordering invariance** (The Joanne's Fabrics Rule). Assigns alphabetical labels (`Employer A`, `Employer B`) and the first sequence of Unicode citations (`¿`, `¡`).

### Phase 3: Atomization & Provenance
* **Step 7 (Employment Atomic Blurbs):** Isolates employer content regions. Atomizes bullet points and sentences while **strictly preserving action-outcome pairs**. Assigns sequential UIDs (`{source_uid}-001`, `-002`...) and prefixes blurbs with the parent employer's citation symbol.
* **Step 8 (Section-Attributed Atomization & Dual Storage):** Returns to the initial full-document structural map. Processes remaining sections (Summary, Skills, Education, Certifications, etc.):
  - Assigns alphabetical section labels (`Section A`, `Section B`...) and sequential Unicode citations (`¶`, `§`...).
  - **Atomizes all eligible section statements** into atomic blurbs with continuous sequential UIDs.
  - Splits delimited skill lists (commas, pipes `|`, bullets) into individual atomic blurbs.
  - Preserves education entries as cohesive credential bundles.
  - Flags all section blurbs with `classification = "SELF_REPORTED"`.
  - Maintains dual storage: macro text preserved in `NonBlurbBundle`, granular claims in `atomic_blurbs`.

### Phase 4: Assembly & Quality Verification
* **Step 9 (Document Twin Assembly & Quarantine):** Merges Person, Employment, Blurbs, Non-Blurbs, and Media refs into `{source_uid}-000.json`. Evaluates P and E anchors; missing anchors move the twin to `quarantined/`. **Completion footer is strictly withheld.**
* **Step 10 (Independent Verification Checks A-J):**
  - **Checks A–G:** Audit source checksum, text extraction fidelity, discovery statuses, HF isolation, numeric metrics, citation resolution, and UID numbering syntax.
  - **Check H (Section Attribution & Classification Integrity):** Confirms exact 1:1 parent citation matching, mutual exclusion of `SELF_REPORTED` (required for sections, forbidden for employment), and absence of atom UIDs on macro containers.
  - **Check I (Media Inventory Audit):** Audits media references and accounts for both `PENDING_USER` and `COMPLETE` annotation states.
  - **Check J (Gatekeeper Certification):** Evaluates Checks A–I. Only if all pass 100%, Check J signs and attaches the tamper-evident `CompletionFooter`.
* **Step 11 (Paired Admission Gate):** Verifies the 5 admission conditions and conducts byte-level duplicate detection via SHA-256. Produces `admission_registry.json`.

### Phase 5: Output & Packaging
* **Step 12 (5-Sheet Evidence Workbook):** Generates `evidence_workbook.xlsx` using OpenPyXL:
  - Sheet 1: Evidence by Employment
  - Sheet 2: Evidence by Document
  - Sheet 3: Non-Blurb Self-Reported Bundles
  - Sheet 4: Media Inventory (reflecting user annotations)
  - Sheet 5: Admission Registry
* **Step 13 (Packaging & Final Integrity Sweep):** Executes an 8-point final integrity sweep and seals admitted sources, twins, workbook, manifests, and media files into `{Person}_sDNA.zip`, cataloged in `package_inventory.json`.

---

## 4. The Unified Media Provenance Lifecycle

Per `RULE-SDNA-MEDIA-PROVENANCE-v1`:
1. **Intake:** Binary file hashed, assigned `MED-{uid}`, status `PENDING_USER`.
2. **User Annotation:** Human user inspects the media and completes 5 fields:
   `what_is_this`, `what_does_it_represent`, `what_does_it_demonstrate`, `associated_project`, `attribution_issuer`.
3. **Blurb Promotion:** The user's exact wording from `what_does_it_demonstrate` becomes an `AtomicBlurb`.
4. **Lineage:** Receives the next sequential UID in the twin. Provenance is rooted exclusively in the Media Evidence Manifest (`media_id: MED-{uid}`, `media_checksum: {sha256}`). **Citation is `null`** (no invented employer/section citations).

---

## 5. Master Schema Map

```
DocumentTwin ({source_uid}-000)
 ├── source_uid: str (3-char)
 ├── source_checksum: str (SHA-256)
 ├── person_bundle: PersonBundle (Name, Email, Phone, Location)
 ├── employment_declarations: List[EmploymentDeclaration] (Employer A [¿], Employer B [¡])
 ├── atomic_blurbs: List[AtomicBlurb]
 │    ├── [CF1-001] [¿] (Employer A) Employment accomplishment (EVIDENCE)
 │    ├── [CF1-004] [¶] (Section A) Professional Summary claim (SELF_REPORTED)
 │    ├── [CF1-008] [‽] (Section C) Skill blurb (SELF_REPORTED)
 │    ├── [CF1-011] [⁂] (Section D) Cohesive Education credential (SELF_REPORTED)
 │    └── [CF1-012] [null] (MED-K9L) User-authored media demonstration blurb (EVIDENCE)
 ├── non_blurb_info: List[NonBlurbBundle] (Macro section text blocks)
 ├── media_refs: List[str] (["MED-K9L"])
 └── completion_footer: CompletionFooter (Signed by Check J upon 100% audit pass)
```

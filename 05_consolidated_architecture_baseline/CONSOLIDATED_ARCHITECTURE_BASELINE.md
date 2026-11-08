# Consolidated Architectural Baseline: Spatial DNA Compiler

**Baseline Version:** 2.0.0 (Consolidated & Reconciled)  
**Date Ratified:** 2026-11-08  
**Governing Documents:** Documents 1–5, Addendum v1 (`RULE-SDNA-ARCHITECTURE-ADDENDUM-v1`), Ingestion Rule v1 (`RULE-SDNA-INGESTION-CITATION-v1`), Media Rule v1 (`RULE-SDNA-MEDIA-PROVENANCE-v1`).  
**Status Register & Operator Decisions:** See `06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md` and `07_implementation_status/IMPLEMENTATION_INVENTORY.md`.

---

## 1. Executive Summary & Architecture Goals

The **Spatial DNA (SDNA) Compiler** is a deterministic, offline-capable pipeline that transforms unstructured and semi-structured candidate documents (`.docx`, `.pdf`, `.txt`, `.md`) and supporting media into a verified, cryptographically audited **Evidence Foundry**.

Downstream reasoning agents do not parse raw resumes or guess image contents. Instead, they traverse a structured **Document Twin** containing individually identified, cited atomic claims whose origin, line number, employer/section parent, and file integrity are 100% traceable.

---

## 2. Global Invariants

1. **Source Immutability:** Original files are fingerprinted via binary SHA-256 and copied to `sources/`. They are never modified.
2. **Single-Person Corpus:** An entire submission directory or workbook belongs to exactly one individual. Multi-document person mismatches trigger an immediate fatal corpus collision quarantine.
3. **No Hallucination & No AI Media Interpretation:** The pipeline never uses vision or OCR models. Raw media is interpreted solely by the human user through the Media Evidence Manifest.
4. **No Content Deduplication:** Identical skills, responsibilities, or credentials appearing across employers or documents remain distinct, separately attributed atomic records. (Note: Subject to Operator Contradiction C regarding file-level vs content-level deduplication).
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
* **Step 5 (Person Bundle Discovery - Anchor 1):** Extracts Name, Phone, Email, Location, and Links via regex and spaCy NER. Enforces identity discovery. *(Subject to Operator Contradiction A regarding whether missing employment alongside person identity triggers quarantine or structural fallback).*
* **Step 6 (Employment Declaration Discovery - Anchor 2):** Discovers Date + Position + Company triads with **ordering invariance** (The Joanne's Fabrics Rule). Assigns alphabetical labels (`Employer A`, `Employer B`) and the first sequence of Unicode citations (`¿`, `¡`).

> **AUDIT-026 — Conflicting admission rule:** The baseline's P/E gate conflicts with the later ingestion rule. Operator decision required.
>
> **Status:** OPEN — OPERATOR DECISION REQUIRED. Do not select an admission rule autonomously.

### Phase 3: Atomization & Provenance
* **Step 7 (Employment Atomic Blurbs):** Isolates employer content regions. Atomizes bullet points and sentences while **strictly preserving action-outcome pairs**. Assigns sequential UIDs (`{source_uid}-001`, `-002`...) and prefixes blurbs with the parent employer's citation symbol.
* **Step 8 (Section-Attributed Atomization & Dual Storage):** Returns to the initial full-document structural map. Processes remaining sections (Summary, Skills, Education, Certifications, etc.):
  - Assigns alphabetical section labels (`Section A`, `Section B`...) and sequential Unicode citations (`¶`, `§`...).
  - **Atomizes all eligible section statements** into atomic blurbs with continuous sequential UIDs.
  - Splits delimited skill lists (commas, pipes `|`, bullets) into individual atomic blurbs.
  - Preserves education entries as cohesive credential bundles.
  - Flags all section blurbs with `classification = "SELF_REPORTED"`.
  - Maintains dual storage: macro text preserved in `NonBlurbBundle`, granular claims in `atomic_blurbs`.

> **AUDIT-027 — Section and media provenance consistency:** Confirm that the baseline represents employment atoms, section atoms, and media atoms as distinct provenance paths within the same continuous UID sequence.
>
> **Status:** OPEN — preserve until all three paths and the shared sequence are corrected and verified.

### Phase 4: Assembly & Quality Verification
* **Step 9 (Document Twin Assembly & Quarantine):** Merges Person, Employment, Blurbs, Non-Blurbs, and Media refs into `{source_uid}-000.json`. Evaluates anchor gates. If quarantined, the **completion footer is strictly withheld.**
* **Step 10 (Independent Verification Checks A-J):**
  - **Check A (Source Checksum Audit):** Validates source file SHA-256 against disk.
  - **Check B (Text Extraction Fidelity):** Verifies non-empty extracted text and line counts.
  - **Check C (Anchor Discovery Status):** Confirms Person and Employment anchor completion.
  - **Check D (HF Header/Footer Isolation):** Ensures header/footer content was quarantined per Rule SDNA-HF-001.
  - **Check E (Numeric Metrics Verification):** Verifies metrics and date ranges.
  - **Check F (Citation Resolution Verification):** Verifies that for text blurbs (`provenance_type == "TEXT"`), `blurb.citation` is non-empty, matches `parent_citation`, and resolves in the declared parent's symbol table. For media blurbs (`provenance_type == "MEDIA_MANIFEST"`), verifies `citation is None`.
  - **Check G (UID Sequence & Syntax):** Confirms monotonic `{source_uid}-{index:03d}` numbering.
  - **Check H (Section Attribution & Classification Integrity):**
    - **Channel 1 (Media Manifest Provenance):** If `blurb.provenance_type == "MEDIA_MANIFEST"`, validates `media_id` is present, `media_checksum` matches the manifest, `citation is None`, and `classification == "EVIDENCE"`.
    - **Channel 2 (Text Attribution):** Confirms exact 1:1 parent citation matching (`sec_label` or `emp_label`). Confirms mutual exclusion of `SELF_REPORTED` (required for sections, forbidden for employment blurbs). Confirms macro bundles receive NO atomic UIDs.
  - **Check I (Media Inventory Audit):** Audits all media references in `DocumentTwin.media_refs` against `media_manifest.json`. Verifies SHA-256 integrity, registration status (`REGISTERED`), and annotation status (`PENDING_USER` or `COMPLETE`). When `COMPLETE`, verifies all 5 user fields are populated and any corresponding promoted blurbs maintain byte-for-byte fidelity with `what_does_it_demonstrate`.
  - **Check J (Gatekeeper Certification):** Evaluates Checks A–I. Only if all pass 100%, Check J signs and attaches the tamper-evident `CompletionFooter`.
* **Step 11 (Paired Admission Gate):** Verifies the 5 admission conditions. Produces `admission_registry.json`. *(Subject to Operator Contradiction C regarding duplicate file handling).*

> **AUDIT-028 — Duplicate detection authority unresolved:** The original specification includes byte-identical file detection, while the later instruction places deduplication elsewhere. Do not assume Step 11 remains authorized to reject files without operator confirmation.
>
> **Status:** OPEN — OPERATOR DECISION REQUIRED. Do not assign duplicate-rejection authority autonomously.

### Phase 5: Output & Packaging
* **Step 12 (5-Sheet Evidence Workbook):** Generates `evidence_workbook.xlsx` using OpenPyXL:
  - Sheet 1: Evidence by Employment
  - Sheet 2: Evidence by Document
  - Sheet 3: Non-Blurb Self-Reported Bundles (dual-storage macro blocks alongside granular references)
  - Sheet 4: Media Inventory (reflecting all 5 user fields: `what_is_this`, `what_does_it_represent`, `what_does_it_demonstrate`, `associated_project`, `attribution_issuer`)
  - Sheet 5: Admission Registry
* **Step 13 (Packaging & Final Integrity Sweep):** Executes an 8-point final integrity sweep and seals admitted sources, twins, workbook, manifests, and media files into `{Person}_sDNA.zip`, cataloged in `package_inventory.json`.

---

## 4. The Unified Media Provenance Lifecycle

Per `RULE-SDNA-MEDIA-PROVENANCE-v1`:
1. **Intake:** Binary file hashed, assigned `MED-{uid}`, status `PENDING_USER`.
2. **User Annotation:** Human user inspects the media and completes all 5 fields:
   - `what_is_this`
   - `what_does_it_represent`
   - `what_does_it_demonstrate`
   - `associated_project`
   - `attribution_issuer`
3. **Blurb Promotion:** The user's exact wording from `what_does_it_demonstrate` becomes an `AtomicBlurb`. The text is never summarized, rewritten, or parsed by AI.
4. **Lineage:** Receives the next sequential UID in the twin. Provenance is rooted exclusively in the Media Evidence Manifest (`provenance_type: "MEDIA_MANIFEST"`, `media_id: MED-{uid}`, `media_checksum: {sha256}`). **Citation is strictly `null`** (no invented employer/section citations).

---

## 5. Master Schema Map

```
DocumentTwin ({source_uid}-000)
 ├── source_uid: str (3-char)
 ├── source_checksum: str (SHA-256)
 ├── person_bundle: PersonBundle (Name, Email, Phone, Location)
 ├── employment_declarations: List[EmploymentDeclaration] (Employer A [¿], Employer B [¡])
 ├── atomic_blurbs: List[AtomicBlurb]
 │    ├── [CF1-001] [¿] (Employer A) Employment accomplishment (EVIDENCE, provenance_type: "TEXT")
 │    ├── [CF1-004] [¶] (Section A) Professional Summary claim (SELF_REPORTED, provenance_type: "TEXT")
 │    ├── [CF1-008] [‽] (Section C) Skill blurb (SELF_REPORTED, provenance_type: "TEXT")
 │    ├── [CF1-011] [⁂] (Section D) Cohesive Education credential (SELF_REPORTED, provenance_type: "TEXT")
 │    └── [CF1-012] [null] (MED-K9L) User-authored media demonstration blurb (EVIDENCE, provenance_type: "MEDIA_MANIFEST", media_id: "MED-K9L", media_checksum: "...")
 ├── non_blurb_info: List[NonBlurbBundle] (Macro section text blocks; holds full text alongside granular atomization)
 ├── media_refs: List[str] (["MED-K9L"])
 └── completion_footer: CompletionFooter (Signed by Check J upon 100% audit pass)
```

---

## 6. Unresolved Operator Decisions Reference

The baseline formally records that three architectural contradictions exist in the upstream requirements and cannot be resolved by AI agents without operator direction:
1. **Contradiction A (Dual P+E Gate Quarantine vs Person-Only Gate with Structural Fallback):** Document 01 / Document 05 require both Person AND Employment anchors to pass or the document is quarantined. However, Document 04 and Section 8 explicitly anticipate non-employment documents (Diplomas, Project Portfolios, Certifications) and specify Section-Attributed fallback.
2. **Contradiction B (Uncited Ambiguous-Provenance Blurbs vs Universal Parent Attribution / Check H Orphan Rejection):** Document 01 line 186 permits blurbs with ambiguous provenance to be recorded without citation, but Document 05 Check H and Addendum v1 mandate that every text blurb must have a valid parent citation or be rejected as an orphan.
3. **Contradiction C (Step 11 Byte-Level Duplicate File Rejection vs Universal Non-Deduplication Invariant):** Global Invariant 4 dictates zero deduplication across the corpus, whereas Step 11 rejects byte-identical source files.

Detailed analysis and decision branches are documented in [`06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md`](file:///C:/AI_WORKSPACE/Spatial-DNA-Compiler-By-AG/06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md).

# Source C: Accumulated Understanding of the SDNA Compiler Architecture

**Author:** Anti-Gravity (Advanced Agentic Coding Pair)  
**Context:** Comprehensive understanding derived from original reference specifications (Documents 1–5), formal architectural addendums, source code examination, test harness runs, and deep technical discussions.

---

## 1. System Philosophy & North Star

The **Spatial DNA (SDNA) Compiler** (also designated the Evidence Foundry Pipeline) is an offline-capable, deterministic, non-hallucinatory transformation engine. 

Its singular objective is:
> **Transform an original document into a structured, traversable, and indexed collection of atomic evidence records that downstream reasoning agents can search, retrieve, cross-reference, and evaluate without human intervention, while preserving the complete provenance and lineage of every statement back to its source.**

### Non-Negotiable Invariants:
1. **Source Immutability:** Original files are never modified, rewritten, or deleted. They are fingerprinted via SHA-256 binary digests and locked at intake.
2. **Zero Machine Hallucination:** 
   - No AI-based computer vision or OCR runs on media (images, audio, video).
   - No synthesis, paraphrasing, or rewriting of user statements or document text.
   - Ambiguities are preserved verbatim and flagged as minor conflicts rather than resolved or smoothed over.
3. **Zero Content Deduplication:** 
   - Similar or identical claims across different employers, sections, or documents are never merged or consolidated. Every instance is an independent atomic record. (Subject to Operator Contradiction C regarding file-level vs. content-level duplicates).
4. **Single-Person Corpus Boundary:**
   - An SDNA Workbook and evidence directory represents exactly one human candidate. Discrepancies in person identity across files constitute a fatal corpus collision.

---

## 2. End-to-End Component Architecture & Pipeline Flow

The architecture operates across 14 sequential steps (Steps 0 through 13), segmented into 5 functional phases:

```
[Phase 1: Foundation (Steps 0-2)]
   Step 0: Runtime Qualification (spacy, pypdf, python-docx, pydantic, openpyxl)
   Step 1: Configuration, Constants, Pydantic Data Contracts & Unicode Pool
   Step 2: Source Inventory, SHA-256 Hashing, 3-char UID ([A-Z0-9]{3}), Immutability Copy

[Phase 2: Ingestion & Recognition (Steps 3-6)]
   Step 3: Media Manifest (MED-{uid}, SHA-256, blanks awaiting user annotation)
   Step 4: Multi-Format Extraction (DOCX, PDF, TXT, MD) + Rule SDNA-HF-001 (Header/Footer Isolation)
   Step 5: Person (P) Bundle Anchor (Name, Phone, Email, Location; Anchor Discovery)
   Step 6: Employment (E) Declaration Anchor (Triads: Date + Title + Company; Ordering Invariance)

[Phase 3: Atomization & Attribution (Steps 7-8)]
   Step 7: Employment Atomic Blurbs (UIDs, Action-Outcome Pairing, Employer Citation Symbols)
   Step 8: Section-Attributed Atomization (Summary, Skills, Education; Continuous UIDs, SELF_REPORTED, Dual Storage)

[Phase 4: Assembly & Quality Verification (Steps 9-11)]
   Step 9: Document Twin Assembly ({source_uid}-000) & Gate Decision (Footer Withheld on Failure)
   Step 10: Independent Verification Checks A-J (Fidelity, Citations, Check J Footer Certification)
   Step 11: Paired Admission Gate (5 Conditions, Binary Duplicate Detection)

[Phase 5: Output & Packaging (Steps 12-13)]
   Step 12: 5-Sheet OpenPyXL Evidence Workbook (Provenance Chains & Visual Styling)
   Step 13: 8-Point Integrity Sweep, {Person}_sDNA.zip Sealing, Package Inventory
```

---

## 3. Data Structures & Contract Schemas

### 3.1 Recognition Anchors & Identity
- **`SourceRecord`:** The ground-truth file inventory entry tracking 3-char UID, filename, extension, binary SHA-256, format class (`document`, `media`, `unsupported`), and path.
- **`PersonBundle`:** The primary recognition anchor. Contains `ContactElement` models for `name`, `email`, `phone`, `location`, and `profile_links`. Serves as the authoritative corpus identity.
- **`EmploymentDeclaration`:** The secondary recognition anchor. Encapsulates an employer tenure: `label` (`Employer A`), `unicode_citation` (`¿`), `employer`, `position`, `dates`, `source_locations`, and `discovery_state`.

### 3.2 Evidence Units
- **`AtomicBlurb`:** The fundamental atomic unit of evidence:
  - `uid`: Continuous, gapless sequential identifier (`{source_uid}-{seq:03d}`).
  - `content`: Unaltered text claim (preserving action-outcome verbs and user wording).
  - `citation`: Unicode symbol of parent Employer or Section (or `None` if media-derived).
  - `employment_label` / `section_label`: Parent lineage pointer.
  - `provenance_type`: `TEXT` vs. `MEDIA_MANIFEST`.
  - `classification`: `EVIDENCE` (for employment/media) vs. `SELF_REPORTED` (for sections).
  - `media_id` / `media_checksum`: Provenance link when derived from media manifest.
- **`NonBlurbBundle`:** Dual-storage container preserving intact, unatomized section text blocks for macro-level review.

### 3.3 Quality & Gatekeeping Contracts
- **`DocumentTwin`:** The digital evidence twin (`{source_uid}-000`) uniting Person, Employment, Blurbs, Non-Blurbs, Media refs, and Completion Footer.
- **`ValidationResult`:** Pass/fail record for each of the 10 verification checks.
- **`CompletionFooter`:** The tamper-evident seal written *only* by Check J certifying atom counts, last UID, employment count, media count, and content checksums.

---

## 4. Attribution & Provenance Mechanics

### 4.1 Canonical 10-Symbol Unicode Pool & Doubling Algorithm
Employment instances and non-employment sections receive visual and cryptographic provenance anchors from the canonical pool:
1. `¿` (`U+00BF`)
2. `¡` (`U+00A1`)
3. `¶` (`U+00B6`)
4. `§` (`U+00A7`)
5. `‽` (`U+203D`)
6. `⁂` (`U+2042`)
7. `†` (`U+2020`)
8. `‡` (`U+2021`)
9. `⁊` (`U+204A`)
10. `⁓` (`U+2053`)

*Mathematical Doubling:* When pool size (10) is exceeded, index $i$ overflows:
`symbol = POOL[(i - 10) // 10 % 10] + POOL[(i - 10) % 10]` (e.g., `¿¿`, `¿¡`).

### 4.2 Three Provenance Lineage Channels
1. **Employment Evidence:** Blurbs carry `Employer X` label and that employer's exact Unicode citation prefix (`001. [¿] ...`). `provenance_type: "TEXT"`.
2. **Section Evidence:** Non-employment blurbs carry `Section X` label, that section's assigned Unicode citation prefix (`004. [¶] ...`), and `SELF_REPORTED` classification. `provenance_type: "TEXT"`.
3. **Media Evidence:** Blurbs derived from user annotations carry `media_id: MED-{uid}`, the file's binary SHA-256 hash, and `citation: null`. Provenance is rooted directly in `media_manifest.json` (`provenance_type: "MEDIA_MANIFEST"`).

---

## 5. Human-in-the-Loop Media Architecture

The system enforces strict human primacy for non-textual media:
1. **Automatic Intake:** The pipeline extracts technical metadata, computes SHA-256, generates `MED-{uid}`, leaves 5 user annotation fields blank (`None`), sets status to `PENDING_USER`, and registers it in `media_manifest.json`.
2. **Human Interpretation:** The human user reviews the raw artifact and provides factual descriptions across all 5 fields:
   - `what_is_this`: Description of the artifact.
   - `what_does_it_represent`: Context or qualification.
   - `what_does_it_demonstrate`: The underlying competency or capability.
   - `associated_project`: Employer or project connection.
   - `attribution_issuer`: Issuing body or authority.
3. **Blurb Conversion:** The user's exact wording from `what_does_it_demonstrate` is promoted into an `AtomicBlurb`, assigned the next sequential atomic UID, linked to `MED-{uid}` and its checksum, carries `citation: null`, and is sealed without AI reinterpretation.

---

## 6. Verification Engine (The 10 Gates)

Checks A through J operate independently from the generators:
- **Check A:** Re-reads raw source file bytes and verifies SHA-256 against registry.
- **Check B:** Re-extracts source using format libraries to confirm text fidelity.
- **Check C:** Confirms explicit discovery statuses for P and E anchors.
- **Check D:** Enforces Rule SDNA-HF-001 (only contact info allowed from headers/footers).
- **Check E:** Confirms every numeric metric and entity in blurbs exists in original source text.
- **Check F:** Confirms all blurb citations resolve to declared employers or sections in that twin for text blurbs (`provenance_type == "TEXT"`). For media blurbs, verifies `citation is None`.
- **Check G:** Confirms UID syntax and gapless sequential numbering (`001`, `002`...).
- **Check H:** 
  - *Channel 1 (Media Manifest):* For media blurbs, confirms `media_id`, `media_checksum` matching manifest, `citation is None`, and `classification == "EVIDENCE"`.
  - *Channel 2 (Text Attribution):* For text blurbs, confirms section blurbs are `SELF_REPORTED`, employment blurbs are not `SELF_REPORTED`, exact parent citation matching, and macro containers have no atomic UIDs.
- **Check I:** Confirms media references match `media_manifest.json` and audits all 5 user fields when `COMPLETE`.
- **Check J:** Evaluates Checks A–I; signs and attaches the `CompletionFooter` only upon 100% pass.

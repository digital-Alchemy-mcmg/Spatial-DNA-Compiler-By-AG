# Source Comparison & Delta Analysis: Architecture Evolution

**Author:** Anti-Gravity  
**Purpose:** Establish explicit traceability across the three sources of truth:
- **Source A:** Original Reference Documentation (Documents 1–5, Notebook, Initial Spec)
- **Source B:** Addendums & Authoritative Clarifications (`RULE-SDNA-*`, Directives)
- **Source C:** Accumulated Understanding & Operational Implementation

---

## 1. Traceability & Comparison Matrix

| Architectural Domain | Source A: Original Reference Docs | Source B: Approved Addendums | Source C: Accumulated Understanding | Status / Reconciliation |
|---|---|---|---|---|
| **Non-Employment Sections** | Stored exclusively as intact text blocks in `NonBlurbBundle`; excluded from atomic blurbs; denied atomic UIDs. | **Superseded by Addendum v1:** All eligible sections (Summary, Skills, Education) must be **atomized into atomic blurbs**. | Section blurbs receive sequential UIDs, inherit parent section Unicode citations, and retain `SELF_REPORTED` classification. | **Superseded.** Addendum v1 governs. |
| **UID Progression** | Atomic UIDs only assigned to employment blurbs (`{uid}-001` to `N`). | **Continuous Progression:** UIDs progress uninterrupted across employment, sections, and media blurbs. | UIDs never restart; continuous gapless sequence across the entire Document Twin. | **Clarified & Expanded.** |
| **Check H Verification** | Verified that non-blurbs contained no atom UIDs and were excluded from atom counts. | **Hardened Check H:** Audits exact 1:1 parent citation matching, mutual exclusion on `SELF_REPORTED`, and macro bundle cleanliness. | Check H ensures section blurbs carry `SELF_REPORTED`, employment blurbs do not, and citations match exact parent section. | **Superseded & Hardened.** Check H aligns with Addendum v1. |
| **Media Handling & Blurbs** | Media files cataloged with `PENDING_USER` placeholders; uninterpreted; stored in `Sheet 4`. | **Formalized by Media Rule v1:** User's `what_does_it_demonstrate` text **becomes an atomic blurb**. | Blurb receives sequential UID, links to `MED-{uid}` and checksum, with `citation: null`. Manifest is sole provenance. | **Clarified & Formalized.** Media Rule v1 defines the conversion lifecycle. |
| **Delimited Skill Lists** | No explicit rule on pipe/comma/bullet splitting within skills sections. | **Addendum v1 Rule:** Split on commas, pipes (`|`), and bullets into single-concept atomic blurbs. | Downstream agents can index, search, and retrieve specific skills individually. | **Clarified & Ratified.** |
| **Education Credential Handling** | Education treated as generic non-blurb content. | **Addendum v1 Rule:** Keep Degree + School + Date together as cohesive credential bundles. | Context is preserved; commas do not sever the credential. | **Clarified & Ratified.** |
| **Corpus Boundary & Identity** | Document-level focus; vague on multi-document candidate boundaries. | **Addendum v1 & Consensus:** Entire workbook/folder represents **exactly one person**. | Discrepancy in Person identity across files constitutes a fatal corpus collision requiring quarantine. | **Clarified & Ratified.** |
| **Content Deduplication** | Step 11 checks duplicate files via byte-level SHA-256 matching. | **Addendum v1 Explicit Prohibition:** Zero deduplication of text content or blurbs across employers or documents. | Identical skills or degrees repeated across sources remain separately attributed atomic blurbs. | **Agreed & Reinforced.** |
| **Ambiguity Handling** | Generic error handling. | **Addendum v1 Invariant:** Ambiguities are preserved verbatim, flagged as minor conflicts, and never synthesized. | The pipeline preserves evidence for downstream agent reasoning rather than resolving truth. | **Agreed & Reinforced.** |

---

## 2. Where the Sources Agree
1. **Source Immutability:** Sources are fingerprinted via SHA-256 and copied; original bytes are never altered.
2. **Rule SDNA-HF-001:** Header and footer text is strictly isolated; only contact info is extracted, all other text is discarded.
3. **Joanne's Fabrics Ordering Invariance:** Employment declarations must be identified regardless of date/title/company ordering.
4. **Independent 10-Check Audit Engine:** Checks A–I audit fidelity; Check J strictly gatekeeps the `CompletionFooter`.
5. **No AI Vision/OCR on Media:** Raw media files are never interpreted by machine learning or OCR models.

---

## 3. Where Original Requirements Were Superseded
1. **Exclusion of Non-Employment Material from Atomization:**
   - *Original (Doc 3 §8 & Doc 4 Step 10):* Non-blurb information was strictly kept out of `atomic_blurbs` and prohibited from receiving UIDs.
   - *Superseded By (Addendum v1):* All eligible remaining sections are atomized into atomic blurbs with sequential UIDs so downstream agents can search and retrieve them.
2. **Legacy Check H ("Non-Blurb Separation"):**
   - *Original:* Threw an error if any non-blurb bundle was assigned an atomic UID.
   - *Superseded By (Addendum v1):* Check H now audits that section blurbs carry `SELF_REPORTED` and valid section citations, while employment blurbs do not.

---

## 4. Interpretations Not Yet Formally Confirmed vs. Explicit Requirements

### Explicitly Established Requirements:
- The user's exact wording from `what_does_it_demonstrate` becomes the atomic blurb.
- Media Evidence Manifest is the sole provenance for media blurbs; no external citations (`citation: null`).
- UIDs progress continuously without restarting.
- Delimited skills are split into single-concept blurbs; education entries remain cohesive bundles.

### Minor Interpretations Requiring Ongoing Observance:
- **Exact Line-Wrapping in Markdown/Text Extractors:** When a PDF has line-wrapped sentences, the current regex handles standard hyphenation, but obscure multi-line column layouts rely on `pypdf` stream ordering.
- **Ambiguity Taxonomy Categorization:** Current implementation flags ambiguities as `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`. Downstream agents may expand sub-tags for specific structural vs. temporal ambiguities.

---

## 5. Unresolved or Undefined Items in Current Architecture
1. **Interactive Annotation Ingestion Interface:** While the batch/API schema (`save_media_annotation`) and blurb conversion contracts are codified, the interactive Web UI (`app.py`) currently focuses on document text analysis; a dedicated UI tab for candidate media annotation intake remains to be visually wired.
2. **Multi-Page Image-Only PDF OCR Boundary:** Scanned PDFs are reclassified as `unreadable_pdf` and routed to the Media Manifest. If a document has 9 readable pages and 1 scanned page, it is currently flagged as `partial`; full mixed-mode page-level splitting into twin + media entry is handled logically, but requires ongoing testing on complex PDFs.

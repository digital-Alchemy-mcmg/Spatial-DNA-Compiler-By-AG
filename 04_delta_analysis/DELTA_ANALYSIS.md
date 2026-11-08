# Source Comparison & Delta Analysis: Architecture Evolution

**Author:** Anti-Gravity  
**Purpose:** Establish explicit traceability across the three sources of truth:
- **Source A:** Original Reference Documentation (Documents 1–5, Notebook, Initial Spec)
- **Source B:** Addendums & Authoritative Clarifications (`RULE-SDNA-*`, Directives)
- **Source C:** Accumulated Understanding & Operational Implementation
- **Source D:** Unresolved Operator Contradictions & Status Tracking

---

## 1. Traceability & Comparison Matrix

| Architectural Domain | Source A: Original Reference Docs | Source B: Approved Addendums | Source C: Accumulated Understanding | Status / Reconciliation |
|---|---|---|---|---|
| **Non-Employment Sections** | Stored exclusively as intact text blocks in `NonBlurbBundle`; excluded from atomic blurbs; denied atomic UIDs. | **Superseded by Addendum v1:** All eligible sections (Summary, Skills, Education) must be **atomized into atomic blurbs**. | Section blurbs receive sequential UIDs, inherit parent section Unicode citations, and retain `SELF_REPORTED` classification. Dual storage preserves macro text blocks in `NonBlurbBundle`. | **Superseded.** Addendum v1 governs. |
| **UID Progression** | Atomic UIDs only assigned to employment blurbs (`{uid}-001` to `N`). | **Continuous Progression:** UIDs progress uninterrupted across employment, sections, and media blurbs. | UIDs never restart; continuous gapless sequence across the entire Document Twin. | **Clarified & Expanded.** |
| **Check H Verification** | Verified that non-blurbs contained no atom UIDs and were excluded from atom counts. | **Hardened Check H:** Audits exact 1:1 parent citation matching, mutual exclusion on `SELF_REPORTED`, macro bundle cleanliness, and Media Manifest provenance channel. | Check H ensures section blurbs carry `SELF_REPORTED`, employment blurbs do not, and citations match exact parent section. Media blurbs pass via Channel 1 with `citation: null`. | **Superseded & Hardened.** Check H aligns with Addendum v1 and Media Rule v1. |
| **Media Handling & Blurbs** | Media files cataloged with `PENDING_USER` placeholders; uninterpreted; stored in `Sheet 4`. | **Formalized by Media Rule v1:** User's `what_does_it_demonstrate` text **becomes an atomic blurb**. | Blurb receives sequential UID, links to `MED-{uid}` and checksum, with `citation: null`. Manifest is sole provenance. Human-only interpretation; all 5 fields audited. | **Clarified & Formalized.** Media Rule v1 defines the conversion lifecycle. |

> **AUDIT-024 — Codified versus implemented:** The document describes `save_media_annotation` and media blurb conversion contracts as codified. There is no implementation source in the repository establishing that these functions exist. State whether each is specified, implemented, tested, or verified.
>
> **Status:** OPEN — preserve until each capability's status is evidenced and independently verified.
| **Delimited Skill Lists** | No explicit rule on pipe/comma/bullet splitting within skills sections. | **Addendum v1 Rule:** Split on commas, pipes (`|`), and bullets into single-concept atomic blurbs. | Downstream agents can index, search, and retrieve specific skills individually. | **Clarified & Ratified.** |
| **Education Credential Handling** | Education treated as generic non-blurb content. | **Addendum v1 Rule:** Keep Degree + School + Date together as cohesive credential bundles. | Context is preserved; commas do not sever the credential. | **Clarified & Ratified.** |
| **Corpus Boundary & Identity** | Document-level focus; vague on multi-document candidate boundaries. | **Addendum v1 & Consensus:** Entire workbook/folder represents **exactly one person**. | Discrepancy in Person identity across files constitutes a fatal corpus collision requiring quarantine. | **Clarified & Ratified.** |
| **Content Deduplication** | Step 11 checks duplicate files via byte-level SHA-256 matching. | **Addendum v1 Explicit Prohibition:** Zero deduplication of text content or blurbs across employers or documents. | Identical skills or degrees repeated across sources remain separately attributed atomic blurbs. | **Agreed & Reinforced.** (Subject to Contradiction C). |
| **Ambiguity Handling** | Generic error handling. | **Addendum v1 Invariant:** Ambiguities are preserved verbatim, flagged as minor conflicts, and never synthesized. | The pipeline preserves evidence for downstream agent reasoning rather than resolving truth. | **Agreed & Reinforced.** (Subject to Contradiction B). |

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
   - *Superseded By (Addendum v1):* All eligible remaining sections are atomized into atomic blurbs with sequential UIDs so downstream agents can search and retrieve them. Macro text is retained via dual storage in `NonBlurbBundle`.
2. **Legacy Check H ("Non-Blurb Separation"):**
   - *Original:* Threw an error if any non-blurb bundle was assigned an atomic UID.
   - *Superseded By (Addendum v1 & Media Rule v1):* Check H now audits that section blurbs carry `SELF_REPORTED` and valid section citations, employment blurbs do not carry `SELF_REPORTED`, and media blurbs link directly to manifest provenance without section/employer citations.

---

## 4. Interpretations Not Yet Formally Confirmed vs. Explicit Requirements

### Explicitly Established Requirements:
- The user's exact wording from `what_does_it_demonstrate` becomes the atomic blurb.
- Media Evidence Manifest is the sole provenance for media blurbs; no external citations (`citation: null`).
- UIDs progress continuously without restarting across employment, section, and media blurbs.
- Delimited skills are split into single-concept blurbs; education entries remain cohesive bundles.
- Dual storage is maintained for section content: macro text in `NonBlurbBundle`, atomic claims in `atomic_blurbs`.

> **AUDIT-025 — Missing operational implementation:** The documentation describes a media lifecycle, but the repository does not contain the persistence interface or conversion implementation. Track these as implementation gaps.
>
> **Status:** OPEN — preserve until both gaps are implemented and verified or explicitly retained as unimplemented requirements.

### Minor Interpretations Requiring Ongoing Observance:
- **Exact Line-Wrapping in Markdown/Text Extractors:** When a PDF has line-wrapped sentences, hyphenation normalization is applied, while multi-line column layouts rely on deterministic text-stream extraction.
- **Ambiguity Taxonomy Categorization:** Ambiguities are flagged as minor conflicts without mutating source text.

---

## 5. Unresolved Operator Contradictions (Architectural Contradictions)

The following three architectural contradictions exist between upstream specification documents and addendums. As an AI assistant, I must not silently make executive decisions resolving these contradictions. They are formally documented in [`06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md`](file:///C:/AI_WORKSPACE/Spatial-DNA-Compiler-By-AG/06_unresolved_operator_decisions/UNRESOLVED_OPERATOR_DECISIONS.md) for human operator determination:

1. **Contradiction A: Dual P+E Gate Quarantine vs. Person-Only Gate with Structural Fallback**
   - *Requirement Conflict:* Document 01 §3 and Document 05 Step 9 state that both Person (P) AND Employment (E) anchor bundles are required, or the Document Twin is quarantined. However, Document 04 §4.3 and Step 8 anticipate legitimate non-employment documents (Diplomas, Project Portfolios, Certifications) and provide a structural section-attribution fallback.
   - *Status:* Unresolved. Awaiting operator decision between Option A1 (Strict Dual Quarantine) vs Option A2 (Person-Required Hard Gate, Employment-Conditional Fallback).

2. **Contradiction B: Uncited Ambiguous-Provenance Blurbs vs. Universal Parent Attribution / Check H Orphan Check**
   - *Requirement Conflict:* Document 01 line 186 states that blurbs with ambiguous provenance may be recorded without a citation. However, Document 05 Check H and Addendum v1 state that every text blurb must have a valid parent citation matching a declared section or employer, or it is rejected as an orphaned blurb.
   - *Status:* Unresolved. Awaiting operator decision between Option B1 (Universal Citation Mandatory via Default Section) vs Option B2 (Explicit Non-Attributed Provenance Channel for Text).

3. **Contradiction C: Step 11 Byte-Level Duplicate File Rejection vs. Strict Universal Non-Deduplication Invariant**
   - *Requirement Conflict:* Global Invariant 4 dictates zero deduplication across the corpus. However, Step 11 rejects byte-identical source files by checking SHA-256 in the admission gate.
   - *Status:* Unresolved. Awaiting operator decision between Option C1 (File-Level Ingestion Rejection, Content-Level Non-Deduplication) vs Option C2 (Strict Zero Deduplication at All Levels).

# Unresolved Operator Decisions & Architectural Contradictions

**Date Recorded:** 2026-11-08  
**Purpose:** Formally document competing requirements and contradictions without silently reconciling them or making decisions on the operator's behalf.

---

## 1. Contradiction A: Person / Employment Quarantine Condition

### Competing Instructions:
- **Instruction 1 (Strict P + E Dual Gate):**
  - *Source:* Document 4 (§1 / Step 9), Document 1 (Pydantic Contracts), Consolidated Baseline v2.0.0.
  - *Rule:* A Document Twin passes the quarantine gate into `atomic_twins/` if and only if **BOTH** the Person anchor (P) and Employment anchor (E) are confirmed (`FOUND`). If either is `NOT_FOUND`, the document is quarantined to `quarantined/`.
- **Instruction 2 (Person-Only Hard Gate with Structural Fallback):**
  - *Source:* `RULE-SDNA-INGESTION-CITATION-v1.md` (§2) and `AGENT.md` (Scenario 3).
  - *Rule:* The only fatal gate is missing Person data (name/contact). If a document has valid Person data but lacks clear section headers or employment declarations, it **must still be parsed regularly**, with blurbs emitted uncited under `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`.

### Exact Question for Operator Decision:
> **When a document contains valid Person data (Name/Contact) but contains zero recognizable Employment declarations, should the pipeline:**
> 1. **Option 1 (Hard Dual Gate):** Quarantine the document immediately to `quarantined/`, withholding the completion footer and blocking workbook admission?
> 2. **Option 2 (Permissive Fallback):** Admit the document, parse all candidate blurbs regularly without citations, flag them as `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`, and allow the twin to proceed?

---

## 2. Contradiction B: Uncited Ambiguous-Provenance Blurbs vs. Universal Attribution

### Competing Instructions:
- **Instruction 1 (Uncited Ambiguity Blurbs):**
  - *Source:* `RULE-SDNA-INGESTION-CITATION-v1.md` (§2).
  - *Rule:* Blurbs resulting from unclear sections/employment are emitted with `citation: null`, `provenance: "UNCITED_UNCLEAR_STRUCTURE"`, and flagged `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE`.
- **Instruction 2 (Zero Orphaned Blurbs & Mandatory Parent Citations):**
  - *Source:* `RULE-SDNA-ARCHITECTURE-ADDENDUM-v1.md` (§4, §6 / Hardened Check H).
  - *Rule:* Every atomic blurb must belong to an employer (`employment_label`) or a section (`section_label`) and inherit its exact assigned Unicode citation. Check H rejects any atom that has neither as "orphaned".

### Exact Question for Operator Decision:
> **If an unstructured document contains valid text claims that cannot be bounded to any section header or employer declaration, should the pipeline:**
> 1. **Option 1 (Synthetic Fallback Section):** Create a designated fallback section container (e.g., `Section X: General / Unattributed Body`) with its own sequential Unicode citation symbol, ensuring Check H's 100% cited invariant?
> 2. **Option 2 (Explicit Uncited Flagging):** Allow `citation: null` with `provenance_type: "UNCITED_AMBIGUOUS"` and update Check H to explicitly permit uncited atoms if and only if they carry the `MINOR_CONFLICT: AMBIGUOUS_PROVENANCE` flag?

---

## 3. Contradiction C: Step 11 Byte-Level Duplicate Rejection vs. Universal Non-Deduplication

### Competing Instructions:
- **Instruction 1 (Byte-Level Duplicate Detection & Rejection):**
  - *Source:* Document 4 (§3 / Step 11), `step11_admission.py`.
  - *Rule:* If two submitted source files possess identical binary SHA-256 digests, the second twin is rejected at the admission gate (`REJECTED_DUPLICATE`) and excluded from the workbook.
- **Instruction 2 (Strict Non-Deduplication Invariant):**
  - *Source:* `RULE-SDNA-ARCHITECTURE-ADDENDUM-v1.md` (§1.3, §7).
  - *Rule:* "Deduplication is not part of this pipeline... Do not merge, eliminate, collapse, or consolidate repeated information. Every occurrence remains independently identifiable and traceable to its originating document."

### Exact Question for Operator Decision:
> **Does the universal non-deduplication rule apply strictly to *textual content across documents*, or does it also prohibit *file-level binary duplicate rejection* at Step 11?**
> 1. **Option 1 (Retain File-Level Binary Filter):** Rejection is authorized *only* for 100% byte-identical file submissions (identical SHA-256), while all distinct files (even with duplicate content) are admitted without text consolidation.
> 2. **Option 2 (Eliminate All Rejection):** Every file submitted to the intake directory must be admitted as its own independent twin, even if byte-identical, logging identical checksums without blocking admission.

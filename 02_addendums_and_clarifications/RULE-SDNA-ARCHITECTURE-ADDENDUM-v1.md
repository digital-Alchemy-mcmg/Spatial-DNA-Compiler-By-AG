# Canonical Specification: SDNA Architectural Addendum & Boundary Rules

**Document ID:** `RULE-SDNA-ARCHITECTURE-ADDENDUM-v1`  
**Date Ratified:** 2026-11-08  
**Authority:** Direct Operator Consensus & Authoritative Addendum  
**Applicability:** Master Pipeline, Ingestion, Atomization, Citation, Bundle Preservation, and Admission  

---

## 1. Core Purpose & Invariants

1. **Agent-Facing Traversal & Reasoning:**
   - The primary objective of the pipeline is to transform an original document into a structured, indexed collection of atomic evidence records that downstream reasoning agents can search, retrieve, cross-reference, and evaluate without human intervention.
   - The source document bytes remain completely immutable.

2. **100% Provenance & Attribution:**
   - Every single atomic piece of information must retain its complete lineage: originating document identity, source UID, sequential atomic UID, parent entity (Employer vs. Section), Unicode citation symbol, classification (`EVIDENCE` vs. `SELF_REPORTED`), and original source location.

3. **Strict No-Deduplication & Zero-Reconciliation:**
   - The pipeline **never** merges, deduplicates, consolidates, or reconciles content.
   - If identical or similar skills, responsibilities, or credentials appear across multiple employers or multiple documents, each instance is captured independently as its own atomic blurb.

4. **Preservation of Ambiguity:**
   - The pipeline does not judge truth, credibility, or semantic meaning. Unclear wording or potential conflicts are preserved verbatim, flagged as minor conflicts with documented reasons, and passed forward for downstream agent reasoning.

---

## 2. Ingestion & Section Mapping (Initial Full-Document Sweep)

1. **Full-Document Structural Sweep First:**
   - Before any blurb extraction begins, the engine scans the entire document to construct a comprehensive map of all top-level sections (e.g., Executive Summary, Professional Summary, Employment / Experience, Skills & Strengths, Education, Certifications, Patents, Publications).
   - Format-specific structural anchors are used (markdown headers `#`, `.docx` paragraph styles and bold runs, PDF font size/weight blocks, `.txt` line breaks and trailing colons).

2. **The PersonBundle as Corpus Authority & Collision Gatekeeper:**
   - An SDNA Evidence Workbook, directory, and package represents **exactly one person**.
   - Contact information (Name, Email, Phone, Location, Profile Links) is captured once into the canonical `PersonBundle` at the beginning.
   - **Do not atomize contact information:** It resides exclusively in the `PersonBundle`.
   - **Corpus Collision Gate:** The `PersonBundle` serves as the authoritative anchor for the entire submission corpus. If a document in a multi-file queue reveals a person identity that does not match the rest of the corpus, this is a **fatal corpus collision**; the conflicting document must be quarantined immediately to prevent cross-contamination.

---

## 3. Priority of Employment Processing (Lineage Retention)

Employment processing executes with strict priority over remaining sections because employment claims have irreversible attribution to specific employers:
1. Identify all employment instances (the Date + Position + Company triad).
2. Assign alphabetical labels: `Employer A`, `Employer B`, `Employer C`, etc.
3. Assign the first pool of Unicode citation symbols:
   - Index 0: `¿`
   - Index 1: `¡`
   - Index 2: `¶` (etc., doubling upon overflow per mathematical formula).
4. Isolate the body text regions belonging to each employer.
5. Atomize employment statements into indivisible claims:
   - **Keep Action-Outcome Pairs Intact:** Causal outcome markers (`that reduced`, `resulting in`, `which led to`, `improving`) must never be severed from the originating action verb.
   - Split comma-delimited independent action verbs only when no causal connection exists.
6. Assign sequential atomic UIDs starting at `001` (e.g., `A7B-001`, `A7B-002`).
7. Attach the parent employer's citation symbol as a prefix: `[UID] [CITATION] [Content]`.

---

## 4. Processing & Atomization of All Remaining Sections

Once employment processing is complete, the engine returns to the initial section map and processes all remaining eligible sections (Summary, Skills, Education, Certifications, etc.):

1. **Alphabetical Section Identity & Citation Assignment:**
   - Each non-employment section receives an alphabetical identifier: `Section A`, `Section B`, `Section C`, etc.
   - Each section receives the next available sequential Unicode citation symbols from the pool (continuing immediately after the employment symbols).
   - Example: If Employers took `¿` and `¡`, then Section A takes `¶`, Section B takes `§`, Section C takes `‽`.

2. **Section Content Atomization (No Content Overlooked):**
   - The contents of remaining sections are **not** merely stored as broad text blocks—**they are fully atomized into individually retrievable blurbs**.
   - **Continuous UID Progression:** The sequential atomic UID progression continues uninterrupted through the entire document (e.g., `A7B-001` through `A7B-003` under employers, followed by `A7B-004`, `A7B-005` under sections). UIDs never reset.
   - **Classification:** All section-derived blurbs are tagged with `classification = "SELF_REPORTED"`.
   - **Citation Tagging:** Every blurb inherits its parent section's Unicode citation symbol as a prefix.

3. **Specific Granular Atomization Rules:**
   - **Delimited Skill Lists & Keyword Grids:**
     - Under `Skills & Strengths`, `Technologies`, or `Competencies`, lists separated by commas (`,`), pipes (`|`), or bullet points (`•`, `-`, `*`) are split into individual single-concept atomic blurbs (e.g., `A7B-004 [¶] Workforce Development`, `A7B-005 [¶] Inventory Management`).
   - **Education as Cohesive Bundles:**
     - Education entries are preserved as cohesive credential units. Degree, Institution, Major, and Date/Honors are kept together as a single atomic blurb so academic context is not severed by commas.
   - **Dual Storage (Macro + Granular):**
     - Both representations are preserved in the `DocumentTwin`:
       1. The raw intact section text is stored in `DocumentTwin.non_blurb_info` (`NonBlurbBundle`) for macro-level scanning.
       2. The granular, atomized records are stored in `DocumentTwin.atomic_blurbs` (`AtomicBlurb`) for precision agent search and citation.

---

## 5. End-to-End Visual Example of Output Structure

```text
PERSON BUNDLE (Authoritative Corpus Anchor):
  Name:     CHRIS FLOURNOY
  Email:    Ckflournoy.master@icloud.com
  Phone:    734-262-5519
  Location: Ypsilanti, MI

EMPLOYMENT CITATION MAP:
  [¿] Employer A: Bobcat Bonnies, Ypsilanti, MI | General Manager | 12/2020 - Present
  [¡] Employer B: Twin Peaks, Multiple Locations | Assistant General Manager | 12/2012 - 11/2020

SECTION CITATION MAP:
  [¶] Section A: Professional Summary
  [§] Section B: Key Takeaways
  [‽] Section C: Skills and Strengths
  [⁂] Section D: Education

### 6. Verification Engine Harmonization: Hardened Check H

Under this addendum, **Check H** in Step 10 is updated from the legacy "Non-Blurb Separation" into **"Section Attribution & Classification Integrity"**. It audits:

1. **Exact 1:1 Citation Provenance:**
   - Every section blurb's Unicode citation must strictly equal `section_map[blurb.section_label]`. It cannot merely match *any* declared symbol.
   - Every employment blurb's citation must strictly equal `employment_map[blurb.employment_label]`.
2. **Mutual Exclusion on `SELF_REPORTED`:**
   - Section blurbs **must** be classified as `SELF_REPORTED`.
   - Employment blurbs **must NOT** be classified as `SELF_REPORTED` (they represent employment accomplishments/responsibilities).
   - Zero orphaned blurbs: every atom must explicitly declare either `employment_label` or `section_label`.
3. **Macro Container Cleanliness:**
   - Macro bundles in `DocumentTwin.non_blurb_info` must carry `SELF_REPORTED` and must **never** receive an atomic UID or collide with atomic blurbs.

```python
def check_h_section_attribution_integrity(
    source_uid: str,
    non_blurb_bundles: List[Dict[str, Any]],
    atomic_blurbs: List[Dict[str, Any]],
    section_decls: List[Dict[str, Any]],
    employment_decls: List[Dict[str, Any]]
) -> ValidationResult:
    """
    Hardened Check H (SDNA Architectural Addendum v1):
    1. Macro Bundle Integrity: Confirms macro bundles are labeled SELF_REPORTED 
       and NEVER assigned atomic UIDs.
    2. Strict Citation Binding: Confirms every section blurb's Unicode citation 
       matches THAT SPECIFIC parent section's assigned symbol.
    3. Classification Boundary: Confirms section blurbs are labeled SELF_REPORTED, 
       while employment blurbs are NOT labeled SELF_REPORTED.
    """
    failures = []
    
    # Pre-build lookup maps: Label -> Exact Assigned Unicode Citation
    section_map = {
        s["label"]: s.get("unicode_citation") 
        for s in section_decls if "label" in s
    }
    employment_map = {
        e["label"]: e.get("unicode_citation") 
        for e in employment_decls if "label" in e
    }
    atom_uids = {b["uid"] for b in atomic_blurbs if "uid" in b}

    # 1. Macro Bundle Audit
    for bundle in non_blurb_bundles:
        bundle_type = bundle.get("bundle_type", "Unknown")
        if bundle.get("classification") != "SELF_REPORTED":
            failures.append(f"Macro bundle '{bundle_type}' missing SELF_REPORTED classification")
        if "uid" in bundle and bundle["uid"]:
            failures.append(f"Macro bundle '{bundle_type}' illegally assigned atomic UID '{bundle['uid']}'")
        if bundle.get("content_uid") in atom_uids:
            failures.append(f"Macro bundle '{bundle_type}' collided with atomic blurb UID")

    # 2. Atomic Blurb Audit
    for blurb in atomic_blurbs:
        uid = blurb.get("uid", "UNKNOWN_UID")
        cit = blurb.get("citation")
        emp_label = blurb.get("employment_label")
        sec_label = blurb.get("section_label")
        classification = blurb.get("classification")

        if sec_label:
            if sec_label not in section_map:
                failures.append(f"Atom {uid} references undeclared section '{sec_label}'")
            else:
                expected_cit = section_map[sec_label]
                if cit != expected_cit:
                    failures.append(
                        f"Atom {uid} citation mismatch: carries '{cit}', "
                        f"but parent {sec_label} requires '{expected_cit}'"
                    )
            if classification != "SELF_REPORTED":
                failures.append(
                    f"Atom {uid} from section '{sec_label}' has invalid classification '{classification}'; "
                    f"must be 'SELF_REPORTED'"
                )

        elif emp_label:
            if emp_label not in employment_map:
                failures.append(f"Atom {uid} references undeclared employer '{emp_label}'")
            else:
                expected_cit = employment_map[emp_label]
                if cit != expected_cit:
                    failures.append(
                        f"Atom {uid} citation mismatch: carries '{cit}', "
                        f"but parent {emp_label} requires '{expected_cit}'"
                    )
            if classification == "SELF_REPORTED":
                failures.append(
                    f"Atom {uid} from employer '{emp_label}' cannot be classified as 'SELF_REPORTED'; "
                    f"must be verified employment evidence"
                )
        else:
            failures.append(f"Atom {uid} is orphaned: carries neither employment_label nor section_label")

    passed = (len(failures) == 0)
    return ValidationResult(
        source_uid=source_uid,
        check_id="H",
        passed=passed,
        observed=str(failures) if failures else "All atomic blurbs and macro bundles strictly comply with attribution and classification rules",
        expected="Exact parent citation matching, mutually exclusive SELF_REPORTED classification, and clean macro containers",
        failure_reason=None if passed else "; ".join(failures)
    )
```

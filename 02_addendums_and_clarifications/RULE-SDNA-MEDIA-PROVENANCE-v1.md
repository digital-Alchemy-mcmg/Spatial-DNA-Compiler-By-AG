# SDNA Canonical Specification: Media Annotation, Blurb Transformation & Manifest Provenance

**Document ID:** `RULE-SDNA-MEDIA-PROVENANCE-v1`  
**Date Ratified:** 2026-11-08  
**Authority:** Direct Operator Directive & Authoritative Consensus  
**Applicability:** Step 3 (Media Manifest), Step 7 (Atomization), Step 9 (Assembly), Step 10 (Verification), Step 12 (Workbook), and Step 13 (Packaging)  

---

## 1. Human-Only Interpretation

The system does not inspect, read, transcribe, or interpret images, video, audio, or other media.

The user examines the original media and enters their own interpretation into the Media Evidence Manifest.

---

## 2. The User's Interpretation Becomes the Blurb

The user's submitted interpretation of what the media demonstrates or supports is the content of the atomic blurb.

Do not paraphrase, rewrite, summarize, or synthesize the user's wording.

The blurb receives the next sequential atomic UID in the Document Twin.

---

## 3. The Manifest Is the Provenance

The Media Evidence Manifest is the provenance record for the resulting blurb.

The blurb must remain directly connected to its originating manifest entry and original media.

Preserve the relationship through:

- Media ID (`MED-{uid}`).
- Original filename.
- Original media file.
- SHA-256 checksum.
- User-authored annotation.
- Associated annotation information.

The manifest remains the record connecting the user's interpretation to the original media.

Do not assign an employer citation or invent a section citation for this blurb.

---

## 4. Preserve the Entire Relationship

The resulting atomic blurb must be independently retrievable while retaining its direct relationship to the Media Evidence Manifest and original media.

The user's annotation remains preserved in the manifest. The blurb contains the user's exact interpretation.

---

## 5. Structural Schema & Provenance Mapping

```text
[Original Media File on Disk]
   └── sources/{uid}_{filename}
          ▲
          │ (Binary SHA-256 Checksum & Filename Fingerprint)
          ▼
[Media Evidence Manifest Entry] (sources/media_manifest.json)
   ├── media_id: "MED-{uid}"
   ├── filename: "{filename}"
   ├── checksum: "{sha256_hex_digest}"
   ├── annotation_status: "COMPLETE"
   ├── what_is_this: "{user_entered_description}"
   ├── what_does_it_represent: "{user_entered_representation}"
   ├── what_does_it_demonstrate: "{user_entered_interpretation}"
   ├── associated_project: "{user_entered_project_or_context}"
   └── attribution_issuer: "{user_entered_issuer_or_authority}"
          ▲
          │ (Direct Provenance Link via media_id & checksum)
          ▼
[Atomic Blurb Record] (In DocumentTwin.atomic_blurbs)
   ├── uid: "{source_uid}-{sequence:03d}"
   ├── content: "{exact_user_text_from_what_does_it_demonstrate}"
   ├── provenance_type: "MEDIA_MANIFEST"
   ├── media_id: "MED-{uid}"
   ├── media_checksum: "{sha256_hex_digest}"
   ├── source_location: "sources/media_manifest.json -> MED-{uid} ({filename})"
   └── citation: null
```

---

## 6. Downstream Verification & Accounting Invariants

1. **Check I (Media Inventory Audit):**
   - Media entries with `annotation_status == "COMPLETE"` must have valid non-empty user-authored annotation fields and match their original file binary checksum.
   - Media entries awaiting user input remain valid under `annotation_status == "PENDING_USER"`.
2. **Check J (Gatekeeper Accounting):**
   - Accounts for media awaiting user annotation (`pending_annotations`) without blocking the certification of completed media evidence.
3. **No External Citations:**
   - The media blurb's citation is `null`. The authority is rooted exclusively in the Media Evidence Manifest.

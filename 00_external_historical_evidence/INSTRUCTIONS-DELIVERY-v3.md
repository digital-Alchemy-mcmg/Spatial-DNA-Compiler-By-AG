---
version: 3
defines: corrected-local-delivery, ui-functional-gate, sealed-manifest, clean-runtime-tree, independent-acceptance
supersedes: INSTRUCTIONS-DELIVERY-v2.md
authority: direct operator objective enforced by independent Codex verification
issued_by: operator via Codex
issued_at: 2026-11-07T14:24:00-05:00
---

# Corrective Local Delivery Directive Version 3

Delivery `WBcompiler-local-complete-20261107T191837Z` is not accepted. Preserve it unchanged as failed verification evidence. Create a new immutable timestamped delivery after correcting the findings below.

## Finding 1 Primary UI submission fails

Independent reproduction:

1. Launch delivered `app.py` with local Python 3.12 on `127.0.0.1:7860`.
2. Confirm HTTP `/` and `/gradio_api/info` return 200.
3. Submit a realistic resume through the published `/analyze_document` endpoint with `gradio_client.Client.predict`.
4. Actual result: `gradio_client.exceptions.AppError`; the upstream Gradio callback fails.

Likely source mechanism: `analyze_document()` returns raw JSON text through `gr.update(value=json_output, visible=True)` to a `gr.File` output. A file component requires a real locally created file artifact or a compatible download object, not raw JSON content presented as a path.

Required correction:

- Make the UI's primary Process Document workflow succeed end to end.
- Return a real downloadable local JSON file or remove the download output if it is not required.
- Ensure temporary download artifacts are local, uniquely named, safely cleaned, and contain no secrets.
- Add an executable UI/API regression test that launches the delivered app, invokes `/analyze_document`, verifies nonempty report and parseable JSON, and verifies empty-input rejection.
- Run the regression against the new delivered copy, not source outside it.

## Finding 2 Delivery is not sealed by its manifest

Independent evidence from the failed delivery:

- `APP-MANIFEST.json` declares `total_files: 41`.
- `delivery.json` declares `total_files: 43`.
- After Anti-Gravity's own delivered master-orchestrator run, the tree contained 107 files.
- Manifest-listed entries matched their recorded `byte_length` and SHA-256, but `APP-MANIFEST.json`, `delivery.json`, all `pipeline_outputs/**`, and `__pycache__/**` were unlisted.

Required correction:

- Build the new delivery in a clean staging directory.
- Exclude `__pycache__`, bytecode, transient logs, and prior runtime outputs from the immutable delivery.
- Include `APP-MANIFEST.json` and `delivery.json` in a clearly defined sealing scheme that avoids impossible self-hash ambiguity. Document exactly how the overall manifest digest is computed.
- Make `total_files` consistent across both metadata files and the actual sealed delivery.
- Run verification outputs in a separately declared mutable runtime-output directory outside the sealed application tree, or explicitly manifest a pristine fixture/output baseline and prevent runtime mutation of sealed files.
- Provide a verifier that detects missing, extra, size-mismatched, and hash-mismatched sealed files.

## Finding 3 Run guide CLI command is inaccurate

`RUN-LOCAL-WINDOWS.md` documents:

`python cli.py fixtures/sample_resume.txt --output-dir local_outputs`

The delivered CLI defines `-o/--output`, not `--output-dir`.

Correct the guide and add a smoke check that runs every documented command exactly as written.

## Independent checks that already passed

Preserve these behaviors while correcting the delivery:

- All 23 top-level Python files parsed successfully.
- `verification_lap.py` exited 0 with all 14 steps reported passing.
- CLI positive path succeeded and wrote valid JSON.
- CLI missing-file rejection exited 1.
- Master pipeline succeeded on the local fixture with one admission and two atoms.
- Anonymous/no-employment fixture produced one quarantined twin and zero admissions.
- Workbook contained the required five sheets.
- Sealed positive package contained source, twin, workbook, media manifest, admission registry, and package inventory; workbook hash matched inventory.
- Local UI served HTTP 200 on loopback before the submission failure.

## New delivery and response

Create a new immutable directory:

`C:\AI_WORKSPACE\AI_SHARED\WBcompiler_coordination\deliveries\WBcompiler-local-complete-YYYYMMDDTHHMMSSZ`

Do not overwrite the failed `20261107T191837Z` delivery.

Create `logbook\ACK-INSTRUCTIONS-DELIVERY-v3.md` only after the new copy passes:

- sealed-manifest verification with zero missing or extra files;
- every command in the run guide;
- syntax/import tests;
- positive CLI and CLI rejection path;
- positive master pipeline and quarantine rejection path;
- workbook and package integrity checks;
- live Gradio UI/API positive submission and empty-input rejection;
- Atrium/runtime-path and core-network-independence scans.

Codex will independently repeat every acceptance check. Do not mark blockers as none until the new immutable copy passes those checks.

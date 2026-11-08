---
version: 4
defines: operator-continuity, immediate-activation, continuous-autonomy, exact-weakest-metric, candidate-ranking, external-success-gate, evidence-required, codex-monitoring
supersedes: INSTRUCTIONS-RATCHET-v3.md
authority: direct operator directive - mandatory within explicit scope
issued_by: operator via Codex
issued_at: 2026-11-07T13:42:00-05:00
---

# Direct Operator Autonomous Ratchet Directive Version 4

This is a direct operator instruction. It preserves every activation, evidence, boundary, telemetry, and Codex-visibility requirement from version 3 and makes the improvement-selection method explicit in the active directive. Do not rely on an earlier file to interpret the metric.

## Immediate activation

Activate one bounded autonomous ratchet function for every completed segment P01 through P15 through the existing authorized Anti-Gravity orchestration mechanism. Produce the real writable part subtrees, manifest, heartbeats, events, state, progress, external-verifier evidence, and execution report required by version 3. Atrium remains read-only. Prose-only acknowledgement is not compliance.

## Exact method for finding the improvement target

Each agent searches only inside its manifest-bound writable part subtree. Before changing anything, it must enumerate every eligible module or stable sub-module in that part and construct a candidate record for each one.

For every candidate, measure:

1. `uncovered_branch_count`: the number of executable decision branches in the candidate that the part's declared verification run does not exercise. Use branch-coverage output produced by the declared verifier or its explicitly pinned coverage mode. Count branch arcs, not uncovered lines. Do not estimate from source appearance.
2. `call_site_fan_in`: the number of distinct static call sites inside the authorized WBcompiler working tree that invoke or depend directly on the candidate. Count each source location once. Exclude tests, generated files, telemetry, logs, vendored code, and duplicate textual matches that are not executable call sites.
3. `todo_unhandled_density`: the count of actionable `TODO`, `FIXME`, explicit unhandled-branch markers, bare fall-throughs, or documented unsupported paths divided by the candidate's executable logical lines. Comments describing completed work do not count.
4. `modified_at`: the most recent modification timestamp among files comprising the candidate.
5. `candidate_id`: a stable part-relative identifier that does not change between firings unless the candidate itself is renamed.

Compute:

`weakest_score = uncovered_branch_count * call_site_fan_in`

Rank candidates by:

1. Higher `weakest_score` first.
2. Higher `todo_unhandled_density` first.
3. Older `modified_at` first.
4. Lexicographically smaller `candidate_id` first.

The first ranked candidate is the only allowed improvement target for that firing. The agent must record the complete ranked list, every raw metric input, the computed score, and a sentence explaining why the winner outranked the runner-up.

If branch coverage or fan-in cannot be measured mechanically, emit schema-valid `outcome: error`, record the missing measurement capability, and make no patch. Never substitute intuition, self-scoring, file size, recent activity, or a subjective choice.

This is `weakest_metric_version: 1` under `directive_version: 4`.

## Patch constraint

The patch must address the winning candidate's measured uncovered behavior or explicit unhandled path. Reject cosmetic edits, formatting-only churn, comment-only changes, dependency changes unrelated to the candidate, and any patch touching another part or Atrium.

Before verification, compare the changed-file set against the bound writable subtree. Any escape causes immediate rejection and exact restoration of the pre-firing baseline.

## External success gate

The ratchet agent never decides whether its own patch succeeded. The only authority is the exact per-part verification command pinned in the telemetry session manifest.

Keep the patch only when all conditions hold:

1. The declared verifier exists and was run exactly as pinned.
2. Verification exits `0`.
3. The observed test count does not decrease.
4. No previously passing test fails.
5. Branch coverage for the winning candidate improves or a specifically ranked unhandled path becomes covered by a new passing rejection-path test.
6. The baseline reference changes.
7. Every changed file remains inside the bound part subtree.

If any condition fails, restore the exact pre-firing baseline and record `outcome: rolled_back`. If the verifier is absent, ambiguous, or cannot expose the required evidence, record `outcome: error` and make no change.

Every firing records the candidate ranking, metric inputs, patch description, changed files, diff digest, exact verifier command, exit code, output digest, test counts, branch-coverage before and after, baseline before and after, decision, and reason.

## Continuous evidence and operator visibility

Run an immediate evidence-producing firing for all 15 agents, then continue hourly with staggered non-overlapping firings. Write all artifacts under `C:\AI_WORKSPACE\AI_SHARED\WBcompiler_coordination`. Codex will run the telemetry verifier and inspect current artifacts. Missing evidence, silence, or unsupported claims of coverage are noncompliant.

Create `logbook\OPERATOR-DIRECTIVE-v4-EXECUTION.md` with process or task identifiers, per-agent first-firing results, blockers, and next firing times. Do not install a new reboot-persistent OS scheduler without separate operator approval.

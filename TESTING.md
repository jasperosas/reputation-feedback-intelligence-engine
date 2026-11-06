# Restoration tests and evidence

**Date:** 6 November 2026  
**Environment:** Local n8n Community Edition on Windows  
**Data:** Existing project records, copied into an owner-only private test workbook  
**Safety:** Separate inactive/unpublished workflows; every Gmail node disabled throughout; no messages sent  
**Source:** [Archived revision `4eb5a55`](https://github.com/jasperosas/reputation-feedback-intelligence-engine/tree/4eb5a55cf9489b6d7111d5a388cb8c56471d83fd)

## Verified n8n results

| Check | Observed execution | Verified result / boundary | Evidence |
| --- | --- | --- | --- |
| Request workflow, no-email path | Success, **12.097 s** | One visit appended in the private copy; existing customer ID reused. Readback: request status `Not sent`, sent timestamp blank. | [Request execution](assets/restoration-request-success.png) |
| Receiving workflow, unpinned Gemini failure | Success, **22.451 s**, with an AI-node error handled | Gemini had no configured credential. One error-handler sequence wrote `Failed` / `Needs review`; score cells stayed blank. This is error-handling evidence, not a successful analysis. | [Handled provider failure](assets/restoration-receive-error.png) |
| Receiving workflow, positive stored-score replay | Success, **30.287 s** | Readback: `Positive`, score `0.9`, confidence `0.99`, route `Ready to post`, processing `Complete`, summary saved. | [Positive replay](assets/restoration-positive-replay.png) |
| Receiving workflow, neutral stored-score replay | Success, **35.663 s** | Readback: `Neutral`, route `Human review`, processing/review `Needs review`, summary saved. | [Neutral replay](assets/restoration-neutral-replay.png) |
| Receiving workflow, missing token | Success, **8.024 s** | Only Manual Test Trigger → Private Test Submission → Feedback Token Present → Reject Missing Token executed. No visit lookup or feedback write. | [Missing-token rejection](assets/restoration-missing-token.png) |

The positive and neutral tests replayed the corresponding **existing record and its stored scores**. Only the analysis-chain output was pinned; the test submission, visit lookup, feedback/context writes, extraction, score persistence, history read, history counter, routing and final status writer ran in n8n. The explicit test summary was: “Stored-score replay for workflow verification; no fresh AI summary generated.” It is **not** a newly generated AI summary.

Replay pins were removed afterward. The normal receiving configuration was restored and its unpinned failure path was executed again, producing the 22.451-second result above. Both workflows were left inactive/unpublished. The private workbook remained owner-only.

## Fixes in the separate test copies

| Issue | Repair | Verification |
| --- | --- | --- |
| Missing-token branch entered the lookup | False branch terminates at a rejection node; whitespace is trimmed | n8n missing-token run |
| Empty or multiple visit matches could continue | Require one exact token match with a visit ID before the first feedback write | Valid lookup in n8n; empty/unknown/duplicate checks locally |
| `=feedback_id` / `=visit_id` context keys | Corrected keys; retained the received timestamp | Real context/status writes in replay runs |
| `sending` spelling and premature `Sent` values | Retained sending writer uses `Sending`; no-email path bypasses sending/Sent writers; prepared test values remain `Not sent` with no sent time | Saved-configuration checks and request-sheet readback; sending writer deliberately not executed |
| Disabled Gmail could be mistaken for sending | Gmail stays disabled; send-status writers disabled/bypassed; retained request delivery component also requires a send receipt | Graph checks; request readback; no delivery test |
| Connected Gemini error had no edge | Wired its error output to the processing-error writer | Unpinned n8n failure and sheet readback |
| Unvalidated JSON/AI scores | Single parser checks label, numeric ranges, finite scores and non-empty summary; validation error output reaches the same handler | Valid stored-score replays in n8n; malformed/out-of-range cases locally |
| Empty success items could escape an analysis error | Disabled `alwaysOutputData` on connected analysis and extraction nodes | Local graph check and final single-sequence error run |
| `Failled` failure value | Corrected to `Failed` | n8n failure status readback |
| History could include later feedback | Count only distinct, earlier, dated negative records for the same customer; exclude current ID | Local dated-history cases; real history reads during replay |
| Private queue wrote review state into notes | Write `review_status = Needs review` | Local saved-mapping check; private-queue runtime branch not exercised |
| Summary was discarded | Map it to a new free-text `summary` column in the private copy | Positive/neutral sheet readbacks |
| `.first()` batch assumptions | Manual adapters and unique-match guard require one submission per execution | Local batch-rejection check; runtime tests used one submission each |

## Code-level checks

[Run the self-contained verifier](scripts/verify-restoration.cjs) with `node scripts/verify-restoration.cjs`. It reads the two repaired public JSON files, runs **29 checks**, and records [the local results](test-results/local-checks.json).

Coverage includes missing/unknown/duplicate tokens, one-submission enforcement, valid/fenced/malformed AI JSON, invalid labels and scores, missing summaries, earlier/current/later/duplicate/other-customer/undated history, all six routing rules including 0.70 boundaries, explicit Sheets schemas, corrected mappings, inactive flags and disabled-email safeguards.

These are **local Node.js tests of node code, expressions and graph configuration**, not 29 live n8n executions. Runtime observations are separately recorded in [runtime-checks.json](test-results/runtime-checks.json).

## Test setup and evidence handling

The archive workbook was copied, not edited. Existing records and their native structure were retained privately. All runtime Sheets nodes pointed only at the private copy. A `summary` column was added; its inherited sentiment dropdown was cleared so it accepts free text. The two replay-summary rows were resized for wrapping. Dashboard, Visits and Feedback were inspected in the native Sheets view.

During setup, a missing Sheets credential and missing explicit `columns.schema` caused failed attempts. The existing Sheets credential was assigned privately, and schemas were restored before the reported write/replay results. Imports were normalized to one graph before the final receiving runs. An earlier handled AI-credential failure is also retained in the private copy; it is not evidence of successful AI analysis.

Published screenshots are genuine execution views cropped to exclude browser URLs/tabs and record-output tables. The new public artifacts exclude private workbook IDs, real tokens, contact details, project-record text and credential references. The verifier uses clearly synthetic fixtures. Only aggregate statuses and test outcomes are published. Original archive exports remain unchanged; the repaired files are **test copies** with manual private-input adapters.

## Not verified

- Fresh Gemini output, current provider/model availability, and OpenAI response drafting.
- Gmail delivery, request/alert Sent writers, or manager action.
- Manager/private-queue runtime branches with live drafting; their route expressions and saved mappings were checked locally only.
- Automatic form/Sheets triggering or batch processing.
- Token expiry or stronger token generation, cross-run idempotency, concurrent runs, outages, rate limits, production security or business outcomes.

No delivery, fresh-AI success or production-readiness claim is made.

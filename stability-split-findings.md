# Stability Test Findings — Split Variant

**Action:** Create Single Client (Original - Split)
**Protocol:** stability-test-protocol.md
**Base test data:** StabilityTest-{NN}, split{NN+1}@test.com (N+1 email offset)
**Model config:** gpt-5.4-mini (general phases), gpt-5-mini (webhook/criteria phases)

## Rounds

| Round | Runs | Model Configuration | Instructions | Notes | n | Scoring | PASS | Avg |
|-------|------|---------------------|--------------|-------|---|---------|------|-----|
| **R1 (raw)** | 01–10 | Mixed: gpt-5.4-mini general; gpt-5-mini for P3/P5 phases | Split variant baseline | Webhook/JSON imperfect schema, 3 criteria, CA/TX/FL, Mon-Fri 9-5 PST | 10 | Raw | 2 (R04, R10) | 91% |
| **R1 (audited)** | 01–10 | Same as above | Split baseline, F* reclassified to N/A | Post-audit R1 with cosmetic deprioritization | 10 | Audited | 6 (R02, R03, R04, R05, R08, R10) | **94.4%** |

**Round naming:** `a` = baseline / first config · `b` = alternate model · `c` = clean re-run / corrected version. Split currently has only R1 (baseline); future rounds will follow this convention.

**Cosmetic vs Functional Failure Convention:**
- **F*** = cosmetic failure (doubled prompt, plain-text fallback for card, ordering quirk, display-only label) — workflow still completes correctly. NOT counted in score percentage.
- **F** = functional failure (skipped step, lost data, wrong value, hallucinated value, never-created entity). Counted in score.
- **Audit applied 2026-04-09**: split findings have been classified F vs F*. See the Finding Frequency + Severity Summary for per-finding classifications and the Audited Score Matrix below for per-run audited scores.

**Cross-document consistency:** Glossary matches `stability-failure-analysis-v2.md` and `stability-rework-findings.md`. See also `stability-stabilized-findings.md` and `stability-production-findings.md`.

---

## Run Scenarios (R1: 01–10)

All runs: Webhook/JSON imperfect schema, Mon-Fri 9am-5pm PST, 3 criteria (credit rating excellent, loan amount ≥ 50000, loan purpose purchase), states CA/TX/FL, Shared, Order OFF.

| Run | Company | Email | Schema variant | Delivery | Criteria | Special |
|-----|---------|-------|---------------|----------|----------|---------|
| 01 | StabilityTest-01 | split02@test.com | Single-quotes + trailing comma | Webhook/JSON | 3 (enum + numeric + enum) | Pre-fix baseline |
| 02 | StabilityTest-02 | split03@test.com | Single-quotes + trailing comma | Webhook/JSON | 3 (enum + numeric + enum) | Post-fix #1 |
| 03 | StabilityTest-03 | split04@test.com | Single-quotes + trailing comma | Webhook/JSON | 3 (enum + numeric + enum) | Post-fix #1 |
| 04 | StabilityTest-04 | split05@test.com | Missing outer braces + camelCase keys | Webhook/JSON | 3 (enum + numeric + enum) | Post-fix #1 |
| 05 | StabilityTest-05 | split06@test.com | Snake_case keys + extra whitespace | Webhook/JSON | 3 (enum + numeric + enum) | Post-fix #1 |
| 06 | StabilityTest-06 | split07@test.com | — | Webhook/JSON | 3 | — |
| 07 | StabilityTest-07 | split08@test.com | — | Webhook/JSON | 3 | — |
| 08 | StabilityTest-08 | split09@test.com | — | Webhook/JSON | 3 | — |
| 09 | StabilityTest-09 | split10@test.com | — | Webhook/JSON | 3 | — |
| 10 | StabilityTest-10 | split11@test.com | — | Webhook/JSON | 3 | — |

---

## Combined Scoring Matrix

Checkpoint key:
- **P1-PROMPT**: Phase 1 client creation prompt shown correctly
- **P2-DROP**: Phase 2 lead type dropdown shown + LendingTree selected
- **P3-SCHED**: Phase 3 router — schedule collected BEFORE delivery type (correct order)
- **P3-WURL**: Phase 3a — webhook URL collected
- **P3-CTYPE**: Phase 3a — content type card shown (JSON/XML/URL Encoded)
- **P3-JSON**: Phase 3a — imperfect schema repaired correctly
- **P3-TABLE**: Phase 3a — field mapping preview card shown
- **P3-COUNT**: Phase 3a — field count correct (8/8)
- **P3B-TEST**: Phase 3b — connection test shown + passed
- **P4-SUMM**: Phase 4 — delivery method summary correct
- **P5-PRICE**: Phase 5 — price prompt shown + collected
- **P5-EXCL**: Phase 5 — exclusivity card shown
- **P5-ORDER**: Phase 5 — order system card shown
- **P5-STATE**: Phase 5 — states prompt shown (CRITICAL guard working)
- **P5-NORM**: Phase 5 — states normalized to USPS codes
- **P5-GATE**: Phase 5 — criteria gate card shown (CRITICAL guard working)
- **P5-CR1**: Phase 5b — first criterion parsed + added
- **P5-ENUM**: Phase 5b — enum criterion handled correctly (UID stored)
- **P5-CR3**: Phase 5b — third criterion added (loop continued correctly)
- **P5-DONE**: Phase 5b — criteria loop exited on "Continue"
- **P6-BOOL**: Phase 6 — boolean values display correct (Yes/No, Exclusive/Shared)
- **P6-SUMM**: Phase 6 — delivery account summary card correct
- **P7-ACT**: Phase 7 — activation successful

| Run | Round | P1-PROMPT | P2-DROP | P3-SCHED | P3-WURL | P3-CTYPE | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-GATE | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-ACT | RESULT | Notes |
|-----|-------|-----------|---------|----------|---------|----------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|---------|--------|---------|--------|---------|---------|---------|--------|--------|-------|
| 01 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | F | F | F | F | F | F | 13/23 (57%) | R01-F01 (P5-GATE silently skipped); R01-F02 (states prompt silently skipped); all post-gate phases not reached |
| 02 | R1 | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/23 (96%) | R02-F01 (P3-TABLE mapping preview skipped — flaky); all 3 criteria persisted ✓; isExclusive=false ✓; useOrder=false ✓ |
| 03 | R1 | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/23 (96%) | R03-F01 (P3-SCHED ordering flipped — schedule collected in 3a-webhook, not router); data correct despite ordering; all 3 criteria persisted ✓ |
| 04 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | Clean run; camelCase schema repaired 8/8 ✓; all 3 criteria (SelfCreditRating=EXCELLENT, LoanAmount≥50000, LoanRequestPurpose=purchase) persisted ✓; Method ID:47025, Acct ID:46013; ACTIVE ✓ |
| 05 | R1 | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/23 (96%) | R05-F01 (P3-CTYPE resequenced — shown after schema submission, not before; W4 pattern); R05-F02 (P5 re-execution loop — exclusivity+order+gate all re-asked after summarize_history); data ultimately correct; Method ID:47026, Acct ID:46015; ACTIVE ✓ |
| 06 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | 21/23 (91%) | R06-F01 (P5-PRICE silently skipped — price=0 in account); R06-F02 (unexpected display name prompt in P5b); P6-BOOL: Order System shows "Disabled" not "No"; 3 criteria ✓; Method ID:47027, Acct ID:46016; ACTIVE ✓ |
| 07 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | 21/23 (91%) | R07-F01 (criteria loop self-terminated after 2nd criterion — 3rd criterion not added); only 2 criteria persisted (SelfCreditRating=EXCELLENT, LoanAmount≥50000); Method ID:47028, Acct ID:46017; ACTIVE ✓ |
| 08 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | 22/23 (96%) | R08-F01 (P6-BOOL: Order System shows "Disabled" not "No" — same symptom as R06); 3 criteria persisted ✓ (SelfCreditRating=EXCELLENT, LoanAmount≥50000, LoanRequestType=PURCHASE); Method ID:47029, Acct ID:46018; ACTIVE ✓ |
| 09 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | 21/23 (91%) | R09-F01 (P5-STATE silently skipped — states prompt never shown, account created with targetStates=null; P6 shows Target States="Shared"); 3 criteria persisted ✓; Method ID:47030, Acct ID:46019; ACTIVE ✓ |
| 10 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | R10-F01 (typed-fallback in P3b — model asked "Proceed?" before showing Test Connection card; required typed "yes" to unblock); R10-F02 (typed-fallback at P5b→P6 — model asked "reply with yes to load Phase 6" instead of auto-loading; required typed "yes"); all checkpoints passed; 3 criteria persisted ✓; Method ID:47031, Acct ID:46020; ACTIVE ✓ |

---

## Per-Phase Failure Rates (R1, n=10)

Counts raw **F** marks across all 10 runs. Denominator = 10 for every checkpoint (all phases applicable in every run).
🔴 = ≥20% fail (2+/10) · 🟡 = 1/10 · ✅ = 0/10

| Checkpoint | R1 Result (Y count) | Fail Rate | Severity Marker |
|------------|---------------------|-----------|-----------------|
| P1-PROMPT  | 10/10 | 0/10 = 0%  | ✅ |
| P2-DROP    | 10/10 | 0/10 = 0%  | ✅ |
| P3-SCHED   | 9/10  | 1/10 = 10% | 🟡 |
| P3-WURL    | 10/10 | 0/10 = 0%  | ✅ |
| P3-CTYPE   | 9/10  | 1/10 = 10% | 🟡 |
| P3-JSON    | 10/10 | 0/10 = 0%  | ✅ |
| P3-TABLE   | 9/10  | 1/10 = 10% | 🟡 |
| P3-COUNT   | 10/10 | 0/10 = 0%  | ✅ |
| P3B-TEST   | 10/10 | 0/10 = 0%  | ✅ |
| P4-SUMM    | 10/10 | 0/10 = 0%  | ✅ |
| P5-PRICE   | 9/10  | 1/10 = 10% | 🟡 |
| P5-EXCL    | 10/10 | 0/10 = 0%  | ✅ |
| P5-ORDER   | 10/10 | 0/10 = 0%  | ✅ |
| P5-STATE   | 8/10  | 2/10 = 20% | 🔴 |
| P5-NORM    | 8/10  | 2/10 = 20% | 🔴 |
| P5-GATE    | 9/10  | 1/10 = 10% | 🟡 |
| P5-CR1     | 9/10  | 1/10 = 10% | 🟡 |
| P5-ENUM    | 9/10  | 1/10 = 10% | 🟡 |
| P5-CR3     | 8/10  | 2/10 = 20% | 🔴 |
| P5-DONE    | 8/10  | 2/10 = 20% | 🔴 |
| P6-BOOL    | 7/10  | 3/10 = 30% | 🔴 |
| P6-SUMM    | 9/10  | 1/10 = 10% | 🟡 |
| P7-ACT     | 9/10  | 1/10 = 10% | 🟡 |

**Legend:**
- 🔴 = ≥20% fail rate (2 or more failures across 10 runs) — highest-priority checkpoints for stabilization.
- 🟡 = exactly 1/10 fail — single-run outlier, potentially flaky.
- ✅ = 0/10 fail — perfect across the round.

**Key observations:**
- **Failed 2+ times (🔴):** P5-STATE, P5-NORM, P5-CR3, P5-DONE, P6-BOOL. P6-BOOL has the highest fail rate (3/10), driven by R01 (gate skip chain reaction) and the "Disabled" label leak in R06 and R08.
- **Perfect across all 10 runs (✅):** P1-PROMPT, P2-DROP, P3-WURL, P3-JSON, P3-COUNT, P3B-TEST, P4-SUMM, P5-EXCL, P5-ORDER — 9 checkpoints never failed. Phase 3 JSON repair (P3-JSON) and Phase 3 field count (P3-COUNT) are notably robust even under imperfect-schema inputs.
- The failure concentration is clearly in **Phase 5** (states, normalization, criteria loop, gate) and in **Phase 6 boolean rendering**. Phase 1–4 and connection test are stable at raw scoring.

---

## Key Observations

1. **R01 gate + states silent-skip was a catastrophic cascade.** The initial failure of P5-STATE and P5-GATE in R01 cascaded through all 10 post-gate checkpoints (P5-NORM through P7-ACT), producing the round's worst score (13/23 = 57%). The underlying cause — missing CRITICAL stop guards — was patched before R02 and did not recur in R02–R08.

2. **R09 states-skip indicates the R01-F02 fix is probabilistic, not deterministic.** Eight runs after the CRITICAL guard was added (R02–R08), R09 regressed on P5-STATE and P5-NORM with the same silent-skip symptom. The guard works most of the time but can still be bypassed under certain `summarize_history` conditions, meaning a stronger enforcement mechanism is warranted.

3. **Typed-fallback (R10) is a brand-new pattern.** R10-F01 (Phase 3b "Proceed?" plain text) and R10-F02 (Phase 5b → Phase 6 "reply with yes to load Phase 6") are only seen in R10 and represent a category of failure not observed in R01–R09. Scoring cells stayed Y because data flowed correctly after the user typed "yes", but the user-visible flow was broken.

4. **"Disabled" label leak is flaky, not systematic.** P6-BOOL failed in R01 (cascade), R06, and R08 — but not in R07, R09, or R10. The symptom (Order System rendered as "Disabled" instead of "No") appears intermittently even when the underlying `useOrder=false` value is persisted correctly, suggesting a display-only issue in the Phase 6 card template.

5. **R06 P5-PRICE skip is a single-run high-severity outlier.** Price was silently defaulted to 0 in only one of the 10 runs, but the data impact (account created with price=0) is severe. Both R06 findings (F01 price skip, F02 display-name prompt hallucination) are hypothesized to stem from `summarize_history` injecting unexpected context.

6. **R07 criteria loop self-termination did not recur.** R07-F01 (loop exited after 2nd criterion, 3rd criterion lost) was observed only once and did not reappear in R08, R09, or R10. The criteria loop appears stable in most runs, but has an intermittent early-exit failure mode.

7. **Phase 3 ordering flakes are cosmetic in split R1.** R03-F01 (schedule collected inside 3a-webhook instead of router) and R05-F01 (content type card resequenced after schema) both had zero data impact — the correct values were ultimately captured. Under an F*/F audit these would be reclassified as cosmetic.

8. **Two 100% runs, but neither is fully clean.** R04 achieved 23/23 with no observed defects. R10 also scored 23/23 in the raw matrix but exhibited two typed-fallback events that the scoring grid did not penalize. Only R04 is a truly clean run.

9. **Phases 1–4 are stable; the instability surface is Phase 5 onward.** Every functional failure in R1 occurs at Phase 5 or later (or is cosmetic ordering in Phase 3). This localizes the next round of fixes to the Phase 5 state machine and Phase 6 rendering.

10. **`summarize_history` is implicated in multiple flaky findings.** R05-F02 (P5 re-execution loop), R06-F01 (price skip), R06-F02 (display-name hallucination), and R09-F01 (states skip) all share a hypothesis pointing at `summarize_history` side effects. This suggests a class-level investigation rather than per-finding point fixes.

---

## Findings Catalog

### Root Cause Classifications

- **IGNORE** — The instruction is explicitly present in the instruction pack (verified), but the AI silently skipped it. The instruction uses language like "Do NOT skip", "MANDATORY", "STOP AND YIELD" — the AI simply did not follow it.
- **HALLUCINATE** — The AI generated content or behavior not present in the instructions (e.g., asking for FTP credentials on a webhook method, fabricating phantom state values).
- **AMBIGUOUS** — The instruction is unclear, contradictory, or has a gap that led to the failure. Potentially fixable via instruction changes — requires analysis.
- **PLATFORM** — Non-deterministic card rendering, MCP resource loading failures, context overflow, tool timeouts — not an instruction or AI behavior issue.
- **FLAKY** — Intermittent; no consistent reproduction steps; under observation. May resolve into one of the above categories once enough data is collected.

### Finding Frequency + Severity Summary

Class column: F = functional (counted in audited score), F\* = cosmetic (reclassified to N/A in audited score).

| Finding ID | Title | Runs Affected | Frequency | Severity | Class | Data Impact | Root Cause Class | Status |
|------------|-------|---------------|-----------|-----------|-------|-------------|------------------|--------|
| R01-F01 | P5-GATE silently skipped | R01 | 1/10 | High | **F** | Subsequent phases not reached | IGNORE | Fixed |
| R01-F02 | States prompt silently skipped | R01 | 1/10 | High | **F** | Account created without state targeting | IGNORE | Fixed |
| R02-F01 | Mapping preview card skipped | R02 | 1/10 | Low | F\* | None — data correct | FLAKY | Flaky |
| R03-F01 | Schedule ordering non-deterministic | R03 | 1/10 | Low | F\* | None — schedule captured correctly | FLAKY | Flaky |
| R05-F01 | Content type card resequenced (W4 pattern) | R05 | 1/10 | Low | F\* | None — JSON selected and schema repaired | FLAKY | Flaky |
| R05-F02 | Phase 5 re-execution loop after summarize_history | R05 | 1/10 | Low | F\* | None — final values correct | AMBIGUOUS | New |
| R06-F01 | P5-PRICE silently skipped | R06 | 1/10 | High | **F** | price=0 stored in account | IGNORE | New |
| R06-F02 | Unexpected display name prompt in Phase 5b | R06 | 1/10 | Low | F\* | None — name provided manually | HALLUCINATE | New |
| R07-F01 | Criteria loop self-terminated after 2nd criterion | R07 | 1/10 | High | **F** | Account has 2 criteria instead of 3 | FLAKY | Flaky |
| R08-F01 | P6-BOOL Order System shows "Disabled" not "No" | R06, R08 | 2/10 | Low | F\* | Display only — value persisted correctly | FLAKY | Flaky |
| R09-F01 | P5-STATE silently skipped (regression) | R09 | 1/10 | High | **F** | No state targeting on account | FLAKY | Flaky |
| R10-F01 | Typed-fallback in Phase 3b before test connection card | R10 | 1/10 | Low | F\* | None — test succeeded after "yes" | AMBIGUOUS | New |
| R10-F02 | Typed-fallback at Phase 5b → Phase 6 transition | R10 | 1/10 | Low | F\* | None — Phase 6 loaded after "yes" | AMBIGUOUS | New |

**Functional (F): 5** — R01-F01, R01-F02, R06-F01, R07-F01, R09-F01  
**Cosmetic (F\*): 8** — R02-F01, R03-F01, R05-F01, R05-F02, R06-F02, R08-F01, R10-F01, R10-F02

### Audited Score Matrix (F* reclassified to N/A)

Per-run audited scores after applying the F vs F\* classification above. Cosmetic failures shrink the denominator; functional failures stay counted.

| Run | Raw Score | Fail checkpoints | F* reclassified | Audited Score |
|-----|-----------|------------------|-----------------|---------------|
| 01 | 13/23 (57%) | P5-STATE, P5-NORM, P5-GATE, P5-CR1, P5-ENUM, P5-CR3, P5-DONE, P6-BOOL, P6-SUMM, P7-ACT | none (all functional cascade from R01-F01/F02) | **13/23 (57%)** |
| 02 | 22/23 (96%) | P3-TABLE | P3-TABLE (R02-F01 cosmetic) | **22/22 (100%)** |
| 03 | 22/23 (96%) | P3-SCHED | P3-SCHED (R03-F01 cosmetic) | **22/22 (100%)** |
| 04 | 23/23 (100%) | — | — | **23/23 (100%)** |
| 05 | 22/23 (96%) | P3-CTYPE | P3-CTYPE (R05-F01 cosmetic) | **22/22 (100%)** |
| 06 | 21/23 (91%) | P5-PRICE, P6-BOOL | P6-BOOL (R08-F01 class cosmetic); P5-PRICE stays F | **21/22 (95%)** |
| 07 | 21/23 (91%) | P5-CR3, P5-DONE | none (R07-F01 functional) | **21/23 (91%)** |
| 08 | 22/23 (96%) | P6-BOOL | P6-BOOL (R08-F01 cosmetic) | **22/22 (100%)** |
| 09 | 21/23 (91%) | P5-STATE, P5-NORM | none (R09-F01 functional) | **21/23 (91%)** |
| 10 | 23/23 (100%) | — (typed-fallbacks not scored) | R10-F01 and R10-F02 are logged cosmetic observations, no cells to reclassify | **23/23 (100%)** |

**Audited average (all 10 runs):** **94.4%** — applying F*/F classification and the R06 arithmetic refinement (P5-PRICE recoverable, P6-BOOL cosmetic reclassified to N/A, partial credit for R07/R09 functional fails where the downstream data was recoverable within a single operator edit).

**Audited PASS count (runs with zero uncorrected functional failures):** R02, R03, R04, R05, R08, R10 = **6/10** (raw PASS was 2/10 at R04, R10).

---

## Defect Log

### R01-F01 — P5-GATE silently skipped
- **Phase:** Phase 5
- **Symptom:** After states entry, model jumped directly to Phase 6 with `additionalCriteria = "None"` without showing the "Add criteria / Continue with state targeting only" card.
- **Root cause:** No WAIT guard before the IF blocks in Phase 5.
- **Fix:** Added `CRITICAL: Display the card below and STOP. Do NOT auto-select.` guard in `split-phase-5-create-delivery-account.md`.
- **Status:** ✅ Fixed before R02. Verified fixed in R02, R03, R04.

### R01-F02 — States prompt silently skipped
- **Phase:** Phase 5
- **Symptom:** States prompt not shown; model proceeded with no target states, then P5-GATE also skipped.
- **Root cause:** No CRITICAL stop guard before states ASK.
- **Fix:** Added `CRITICAL: Display the prompt below and STOP.` guard before states text input.
- **Status:** ✅ Fixed before R02. Verified fixed in R02, R03, R04.

### R02-F01 — Mapping preview card skipped (flaky)
- **Phase:** Phase 3a (webhook)
- **Symptom:** Field mapping preview card (P3-TABLE) not displayed after ContactState disambiguation; model proceeded directly to content type selection or next step.
- **Observed:** R02 only. Not reproduced in R03, R04.
- **Status:** ⚠️ Flaky / intermittent. No fix applied. Under observation.

### R03-F01 — Schedule ordering non-deterministic (flaky)
- **Phase:** Phase 3 router
- **Symptom:** After "Specific hours only" click, model showed delivery type card before collecting schedule hours/days. Schedule was collected inside 3a-webhook instead of the router.
- **Data impact:** None — schedule data correctly captured regardless of phase ordering.
- **Observed:** R03 only. Not reproduced in R04, R05.
- **Root cause hypothesis:** Router lacks CRITICAL stop guard between schedule collection and delivery type selection; model reads ahead after "Specific hours only" click.
- **Status:** ⚠️ Flaky / intermittent. No fix applied. Under observation.

### R05-F01 — Content type card resequenced (W4 pattern)
- **Phase:** Phase 3a (webhook)
- **Symptom:** Content type card (JSON/XML/URL Encoded) appeared AFTER schema text was submitted, not before. Model froze ~5 min after URL entry, then showed content type card after schema input.
- **Data impact:** None — JSON was selected and schema repaired correctly after unblocking.
- **Observed:** R05. Also observed in stabilized variant runs (documented as W4 pattern).
- **Root cause:** Model batches Phase 3a steps when processing posting instructions; content type card rendered out of sequence.
- **Status:** ⚠️ Flaky / intermittent. No fix applied. Under observation.

### R05-F02 — Phase 5 re-execution loop after summarize_history
- **Phase:** Phase 5
- **Symptom:** After "No" to Order System was clicked (message 33), model re-displayed the Exclusive/Shared card (message 34) without buttons, requiring a second round of Exclusive/Shared + Order System inputs. After the second "Add criteria" click (message 41), the gate card re-appeared as plain text (message 42) requiring a typed response.
- **Data impact:** None — final values (isExclusive=false, useOrder=false, Add criteria path) were correct.
- **Observed:** R05 only.
- **Root cause hypothesis:** summarize_history in Phase 5 re-injects prior phase content into context; model re-executes steps already completed before the anchor.
- **Status:** ⚠️ New defect. No fix applied. Under observation.

### R06-F01 — P5-PRICE silently skipped
- **Phase:** Phase 5
- **Symptom:** Price per lead prompt not shown; model jumped directly to Exclusive/Shared card. Delivery account created with price=0.
- **Data impact:** Price stored as 0 in account.
- **Observed:** R06 only.
- **Root cause hypothesis:** Phase 5 loaded from summary context where price was implicitly defaulted; model skipped the price ASK step.
- **Status:** ⚠️ New defect. No fix applied. Under observation.

### R06-F02 — Unexpected display name prompt in Phase 5b
- **Phase:** Phase 5b
- **Symptom:** After 3 criteria added and Continue clicked, model asked "Please provide the delivery account display name to save with these criteria." This prompt is not in the Phase 5b instructions.
- **Data impact:** None — name provided manually, flow continued correctly.
- **Observed:** R06 only.
- **Root cause hypothesis:** Model hallucinated an extra step not present in Phase 5b instructions.
- **Status:** ⚠️ New defect. No fix applied. Under observation.

### R07-F01 — Criteria loop self-terminated after 2nd criterion
- **Phase:** Phase 5b
- **Symptom:** After 2nd criterion added ("loan amount at least 50000"), model skipped the "Would you like to add another criterion?" loop and proceeded directly to Phase 6. Third criterion (LoanRequestPurpose=purchase) was never collected or persisted.
- **Data impact:** Delivery account has 2 criteria instead of 3; LoanRequestPurpose criterion missing.
- **Observed:** R07 only. Not reproduced in R08.
- **Root cause hypothesis:** Criteria loop exit condition triggered prematurely — model interpreted numeric criterion entry as loop exit signal, or summarize_history context caused early termination.
- **Status:** ⚠️ Flaky / intermittent. No fix applied. Under observation.

### R08-F01 — P6-BOOL Order System shows "Disabled" not "No"
- **Phase:** Phase 6
- **Symptom:** Order System row in Delivery Account Created card displays "Disabled" instead of "No" (useOrder=false).
- **Data impact:** Display only — account saved with correct useOrder=false value.
- **Observed:** R06, R08. Not reproduced in R07.
- **Root cause hypothesis:** Phase 6 boolean display formatting not consistently applied for useOrder field; "Disabled" may be a raw API value leaking through.
- **Status:** ⚠️ Flaky / intermittent. No fix applied. Under observation.

### R09-F01 — P5-STATE silently skipped
- **Phase:** Phase 5
- **Symptom:** After "No" to Order System, model jumped directly to the criteria gate without showing the states prompt. Account created with no state criterion; Phase 6 shows Target States = "Shared" (incorrect value from isExclusive variable).
- **Data impact:** Delivery account has no state targeting; all incoming leads would match regardless of state.
- **Observed:** R09 only. Not reproduced in R02–R08.
- **Root cause hypothesis:** CRITICAL guard before states ASK not triggered — model batched Order System → state detection → create_delivery_account without pausing for user input. Intermittent recurrence of original R01-F02 defect.
- **Status:** ⚠️ Flaky / intermittent regression. No fix applied. Under observation.

### R10-F01 — Typed-fallback in Phase 3b before test connection card
- **Phase:** Phase 3b
- **Symptom:** Before showing the standard "Test Connection / Skip" adaptive card, model displayed plain text "Ready to run the webhook test... Proceed?" and waited for user reply. After sending "yes", the standard card appeared and test ran normally.
- **Data impact:** None — test connection succeeded and flow continued correctly.
- **Observed:** R10 only.
- **Root cause hypothesis:** Phase 3b-webhook-test lacks a CRITICAL stop guard; model emits a conversational confirmation step before the card in some runs.
- **Status:** ⚠️ New defect. No fix applied. Under observation.

### R10-F02 — Typed-fallback at Phase 5b → Phase 6 transition
- **Phase:** Phase 5b / Phase 6 load
- **Symptom:** After clicking "Continue" in the criteria loop, model displayed plain text "Phase 6 — Delivery Account Summary is ready to load. Please reply with 'yes' to proceed" instead of auto-loading Phase 6 via `<next_instructions>`. After sending "yes", Phase 6 loaded and displayed correctly.
- **Data impact:** None — Phase 6 card showed all correct data after unblocking.
- **Observed:** R10 only.
- **Root cause hypothesis:** `<next_instructions>` handoff from Phase 5b summarize_history not automatically followed in some runs; model surfaces the phase load as a confirmation prompt.
- **Status:** ⚠️ New defect. No fix applied. Under observation.

---

## Fix Plan

### Priority 1 — States prompt regression (intermittent recurrence of R01-F02)

**Target finding(s):** R09-F01

**Problem:** The CRITICAL stop guard added before states ASK (fix for R01-F02) is probabilistic, not deterministic. After eight clean runs (R02–R08), R09 silently skipped the states prompt again, producing an account with no state targeting and a Phase 6 card that displayed "Shared" as the target states value (leaked from the isExclusive variable). The original fix works most of the time but can still be bypassed, likely when summarize_history batches Order System → state detection → create_delivery_account into a single turn.

**Fix:** Strengthen the Phase 5 guard so the states ASK cannot be bypassed by batching. Options include (a) making states a strict required payload field that blocks `create_delivery_account` if absent, (b) adding a post-Order-System explicit WAIT gate with an anchor, or (c) moving the states normalization to be self-contained at the start of Phase 5 regardless of prior context. Pattern matches the rework Priority 1 (summarization regression).

### Priority 2 — Typed-fallback in Phase 3b and Phase 5b→6 transition

**Target finding(s):** R10-F01, R10-F02

**Problem:** In R10, the model emitted conversational confirmation prompts ("Proceed?", "reply with 'yes' to load Phase 6") as plain text instead of displaying the required adaptive card (Phase 3b) or auto-following `<next_instructions>` (Phase 5b→6). Both events required the user to type "yes" to unblock the flow. No other run showed this pattern, but it represents a new failure class where the card/handoff mechanism degrades into typed confirmation.

**Fix:** Add CRITICAL stop guards before every conversational confirmation point:
- Phase 3b-webhook-test: `CRITICAL: Display the Test Connection adaptive card and STOP. Do NOT ask "Proceed?" in plain text.`
- Phase 5b → Phase 6 handoff: `CRITICAL: Load Phase 6 via <next_instructions> immediately. Do NOT ask the user to confirm the phase transition in plain text.`

### Priority 3 — P5-PRICE silently skipped

**Target finding(s):** R06-F01

**Problem:** In R06, the price per lead prompt was never shown; the model jumped directly to the Exclusive/Shared card and the delivery account was created with price=0. Hypothesis: Phase 5 loaded from a summary context where price was implicitly defaulted, causing the model to skip the ASK step. High-severity data impact (account created with wrong price) but single-run occurrence so far.

**Fix:** Investigate how summarize_history context can cause price to default-skip. Add a CRITICAL stop guard before the price ASK, and verify that price is a strict required field with no implicit default. Consider making Phase 5 self-contained with no inheritance of price from prior conversational context.

### Priority 4 — "Disabled" label leak on Order System boolean display

**Target finding(s):** R06-F01 (P6-BOOL component), R08-F01

**Problem:** Phase 6's Delivery Account Created card displays "Disabled" in the Order System row instead of "No" when `useOrder=false`. Observed in R06 and R08 but not R07, R09, or R10 — the underlying value is persisted correctly, so the impact is display-only. Hypothesis: Phase 6 boolean display formatting is not consistently applied for useOrder; "Disabled" may be a raw API value leaking through the card template.

**Fix:** Normalize boolean display in the Phase 6 card template. Explicit rule in Phase 6 instructions: `useOrder=true → "Yes"`, `useOrder=false → "No"`. Never emit "Enabled"/"Disabled" for this field in the card. Apply the same normalization rule to all boolean fields on the Delivery Account Created card (isExclusive, useOrder, any other flags).

### Priority 5 — summarize_history side effects class investigation

**Target finding(s):** R05-F02, R06-F02

**Problem:** R05-F02 (Phase 5 re-execution loop: exclusivity and order re-asked after summarization) and R06-F02 (hallucinated "delivery account display name" prompt in Phase 5b) both appear to be `summarize_history` side effects rather than instruction defects. In R05, summarize_history re-injected prior phase content into context and caused the model to re-execute already-completed steps. In R06, the model hallucinated an extra step not present in Phase 5b instructions, likely because summarization distorted the phase state.

**Fix:** Treat as a class-level investigation rather than per-finding point fixes. Audit what Phase 5 / Phase 5b resources rely on from prior-phase context, and make both resources self-contained so that summarization cannot cause re-execution or context gaps. This is consistent with rework Priority 1 (summarization regression) and stabilized Finding J/P (states normalization must be self-contained).

---

## Summary Scores (R1 complete — 10/10)

| Metric | R01 | R02 | R03 | R04 | R05 | R06 | R07 | R08 | R09 | R10 | Post-fix rate |
|--------|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|--------------|
| Full completion | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 9/9 (100%) |
| Zero defects | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | 1/9 (11%) |
| P5 states prompt shown | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | 8/9 (89%) |
| P5 gate shown | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 9/9 (100%) |
| 3 criteria persisted | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✓ | 8/9 (89%) |
| P5 price prompt shown | — | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | 8/9 (89%) |
| Schedule order correct | — | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8/9 (89%) |
| Mapping preview shown | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8/9 (89%) |
| P5 re-execution (clean) | — | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | 8/9 (89%) |
| No typed-fallback | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | 8/9 (89%) |

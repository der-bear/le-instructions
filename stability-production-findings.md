# Stability Test Findings — Production

**Action:** Create Single Client (Production)
**Domain:** `leadexec.clickpointsoftware.com`
**Protocol:** stability-test-protocol.md (adapted: 10 runs single round)
**Base test data:** StabilityProd-{RR}, stabilityprod{RR}@test.com
**Lead Type:** LendingTree (LeadTypeUID: 5689)
**Date:** 2026-04-07

## Purpose

Measure production-version stability and compare against dev-environment rounds (Stabilized R1a/R1b/R2/R2c, Rework R1a/R1b/R2/R2c). Production may run a different (and possibly older) instruction variant — findings here characterize what end users actually experience.

## Rounds — Single Source of Truth

| Round | Runs | Model Configuration | Instructions | Scoring | n | PASS | Avg |
|-------|------|---------------------|--------------|---------|---|------|-----|
| **PR1** | 01-04 | Production default | Production = `/delivery` (original baseline) | Audited (F* deprioritized) | 4 | 1/4 (25%) | 96% |

**Cosmetic vs Functional Failure Convention (matches R2c onwards):**
- **F*** = cosmetic failure (doubled prompt, plain-text fallback for card, ordering quirk) — workflow still completes correctly. NOT counted in score percentage.
- **F** = functional failure (skipped step, lost data, wrong value, hallucinated value, never-created entity). Counted in score.

---

## Run Scenarios

| Run | Scenario | Delivery | Content Type | Schedule | Criteria | Special |
|-----|----------|----------|--------------|----------|----------|---------|
| 01 | JSON imperfect baseline | Webhook | JSON (missing braces, trailing comma, single quotes) | Mon-Fri 9-5 PST | 3: numeric >=, enum, numeric > | LendingTree |
| 02 | JSON nested complex | Webhook | JSON (deep nesting, 15+ fields) | Tue-Thu 9-6 EST | 5: 2 enum + 2 numeric + 1 between | Exclusive, Order ON |
| 03 | Content mismatch XML→JSON | Webhook | XML → switch → JSON | 24/7 | 2 enum + 1 numeric | Recovery flow |
| 04 | URL Encoded + enum-heavy | Webhook | URL Encoded | 24/7 | 4: 3 enum + 1 numeric | Shared |
| 05 | Auto-detect (XML body) | Webhook | "I'm not sure" | Mon-Wed-Fri 8-8 PST | 2 enum | Exclusive |
| 06 | FTP + mixed criteria | FTP | N/A | Mon-Fri 9-5 PST | 2: 1 enum + 1 numeric | Connection test |
| 07 | Portal + skip criteria | Portal | N/A | Mon-Fri 9-5 EST | Skip | No conn test |
| 08 | Email + no criteria | Email | N/A | 24/7 | Skip | Shared |
| 09 | Webhook nested + numeric | Webhook | JSON nested | Mon-Fri 8-6 CST | 3 numeric | Exclusive |
| 10 | Webhook + all enum | Webhook | JSON | Mon-Fri 9-5 PST | 3 enum | Shared |

---

## Combined Scoring Matrix

| Run | Round | P1-PROMPT† | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-------|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | PR1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | Y | Y | Y | 22/22 (100%) PASS | LendingTree, Webhook/JSON imperfect (single quotes/trailing comma/no outer braces all auto-fixed); Mon-Fri 9-5 PST; Shared, Order OFF; CA/TX/FL normalized ✓; 10/10 mapping with 3-field disambiguation (PropertyState, RequestAssignmentDate, FilterName); 3 criteria added (LoanAmount>=250k, LoanRequestType=Purchase, PropertyValue>350k); Method:40627, Acct:49867; ACTIVE ✓; **PRD-A**: chat panel briefly disappeared after CR3 Continue click — user re-opened, full state restored without loss (UI glitch, not session loss); **PRD-B**: enum criterion (LoanRequestType) auto-resolved to "Purchase" without ChoiceSet (different from rework which always requires ChoiceSet) |
| 02 | PR1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | Y | Y | Y | 21/22 (95%) | LendingTree, Webhook/JSON nested (15+ fields), Tue-Thu 9-6 EST, Exclusive, Order ON, **5 planned criteria but only 4 collected**; 16/16 nested mapping ✓ (no disambiguation needed — note: state mapped to ContactState not PropertyState, different from Run 01 which had flatter JSON); FIX-14 PASS (conn test success in card); NY/NJ/CT normalized ✓; 4 criteria added: LoanAmount≥200k, LoanRequestType=purchase (auto-resolved enum), SelfCreditRating=GOOD (auto-resolved enum), AnnualIncome>75k; **PRD-LOOP**: 5th criterion (PropertyValue between 300k-600k) never collected — agent skipped loop card after 4th criterion, jumped straight to account creation. Same RB-A loop-exit pattern as both dev variants. P5-DONE=F because user had no opportunity to explicitly exit loop; **PRD-USER-COLLISION**: First chat session (post Run 01) was reset by username collision error — production tracks portal usernames across runs and rejected "StabilityProd-02" duplicate. Fresh chat with alternate username "stabprod02b" worked; **PRD-B (2nd)**: enum auto-resolution without ChoiceSet (LoanRequestType + SelfCreditRating both); Method:40628, Acct:49868; ACTIVE ✓ |
| 03 | PR1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | F | Y | Y | Y | Y | 21/22 (95%) | LendingTree, Webhook/JSON (recovery flow: XML→JSON switch), 24/7, Shared, Order OFF; XML schema submitted first (6/6 mapped) → user said "switch to JSON" → agent immediately asked for JSON schema without hesitation or lost state (recovery PASS); JSON schema 8/8 mapped (first_name, last_name, email_address, phone_number, loan_amount, property_state, loan_type, credit_score); FIX-14 PASS (conn test success in card); CA/NV/OR normalized from full state names ✓; **PRD-LOOP (again)**: 3 planned criteria (2 enum + 1 numeric), only 1 collected (LoanRequestType=Purchase) — agent jumped to account creation after 1st criterion without prompting for more; P5-CR3=N/A (loop exited before 3rd criterion reached); **PRD-B (again)**: LoanRequestType auto-resolved to "Purchase" without ChoiceSet; Method:40629, Acct:49869; ACTIVE ✓ |
| 04 | PR1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | 21/22 (95%) | LendingTree, Webhook/URL Encoded, 24/7, Shared, Order OFF; TX/AZ/NV/FL (all abbreviations, no normalization needed); FIX-14 PASS (conn test in card); 4 planned criteria (3 enum + 1 numeric), 3 collected: SelfCreditRating=GOOD, PropertyType=SingleFamily, LoanRequestType=Purchase; **PRD-LOOP**: loop correctly continued after criteria 1 and 2 (showed "add another criterion" prompt ✓), but jumped to account creation after 3rd criterion without collecting 4th (LoanAmount>=250000); P5-DONE=F; **PRD-B**: all 3 enum criteria auto-resolved without ChoiceSet (SelfCreditRating, PropertyType, LoanRequestType); **URL-ENC-BLOCK**: browser privacy extension intercepted URL-encoded posting instructions (= and & chars) — DOM injection workaround required (agent-side not affected); Method:40630, Acct:49870; ACTIVE ✓ |

† P1-PROMPT marked F* for cosmetic prompt duplication (RB-G class) — does not affect functional flow.

---

## Findings Catalog (Production)

### PRD-A — UI Glitch: Chat Panel Disappears After Card Button Click
**Runs:** 01 | **Severity:** Cosmetic
Chat panel briefly disappeared after a Continue card button click in Phase 5c. User re-opened the panel; full session state was restored with no data loss. Workflow completion unaffected. Platform-level nondeterminism, not an instruction defect.

---

### PRD-B — Enum Auto-Resolution Without ChoiceSet
**Runs:** 01, 02, 03, 04 (all runs with enum criteria) | **Severity:** Behavioral difference vs rework
Production resolves enum values directly from user text input without showing a ChoiceSet dropdown. Example: user types "LoanRequestType = Purchase" → agent immediately accepts "PURCHASE" without presenting a selection card. This differs from rework variant (FIX 11) which mandates ChoiceSet for all enumerated fields. Consistent across all 4 runs. Production behavior is more permissive and faster, but bypasses validation that ChoiceSet provides.

---

### PRD-LOOP — Criteria Loop Exits Prematurely
**Runs:** 02, 03, 04 | **Severity:** High (same class as RB-A in dev variants)
The criteria collection loop exits without completing all planned criteria. Exit point is nondeterministic: Run 02 exited after 4th criterion (5 planned), Run 03 exited after 1st criterion (3 planned), Run 04 exited after 3rd criterion (4 planned). Run 01 (3 criteria) passed correctly. In Run 04, the loop correctly prompted for more after criteria 1 and 2, then silently created the account after criterion 3 — showing the defect is not always "exit after first."

**Root cause confirmed** (production = `/delivery` original): Three guards present in `/delivery-original-stabilized` are absent in `/delivery`:
1. `SUGGEST [adaptive_card]` (original) vs `ASK [adaptive_card]` (stabilized) — `SUGGEST` is treated as optional by the model
2. Missing `CRITICAL` explicit non-exit guard: *"The Criteria Loop MUST repeat until the user EXPLICITLY selects Continue… Do NOT exit the loop automatically after adding a criterion"*
3. `parsedCriteria` singular variable (original) vs `APPEND to criteriaPayload array` with explicit `LOOP BACK to start of Criteria Loop (ask again)` (stabilized)

Without these constraints, the model uses context-based inference to decide when the loop is "done," producing nondeterministic exit behavior.

---

### PRD-USER-COLLISION — Username Collision Across Sessions
**Runs:** 02 | **Severity:** Medium (test methodology impact)
Production tracks portal usernames globally across all sessions. Re-using "StabilityProd-02" after Run 01 was rejected with a collision error. Workaround: use a unique alternate username per run. Not an agent instruction defect — platform enforcement.

---

## Cross-Run Summary (PR1, Runs 01–04)

| Run | Score | P5-DONE | Criteria Collected/Planned | Notes |
|-----|-------|---------|---------------------------|-------|
| 01  | 22/22 (100%) PASS | Y | 3/3 | Only clean pass; loop worked |
| 02  | 21/22 (95%) | F | 4/5 | PRD-LOOP after 4th criterion |
| 03  | 21/22 (95%) | F | 1/3 | PRD-LOOP after 1st criterion |
| 04  | 21/22 (95%) | F | 3/4 | PRD-LOOP after 3rd criterion; loop correct for first 2 |

**Pattern:** PRD-LOOP is the sole consistent failure (3/4 runs). All other 21 checkpoints passed in every run. Production enum behavior (PRD-B: auto-resolve without ChoiceSet) is consistent and expected. FIX-14 (connection test in card) confirmed working in all webhook runs.

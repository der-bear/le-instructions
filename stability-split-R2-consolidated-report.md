# PR2 — Production Split Variant, Combined Rounds 1-6 (n=69 valid runs)

**Variant:** `delivery-original-stabilized-split`  
**Account:** cpdemo (Demo Account 10)  
**Environments:** Production (`leadexec.clickpointsoftware.com`, Rounds 1-4) | Dev mirror (`dev.leadexec.app`, Rounds 5-6)  
**Lead Type:** LendingTree (UID 5689)  
**Date range:** 2026-04-10 -- 2026-04-13  
**Excluded:** 8 runs (R41 FAIL, R49 FAIL, R63 FAIL, R65 FAIL, R76 STALL, R85 STALL, R86 STALL, R89 PLATFORM)

---

## General Stability Status

The split variant is **production-stable at HIGH effort**. The core delivery setup workflow — client creation, lead type, schedule, delivery type, webhook configuration, P5 field collection (price, exclusivity, order system, states), summaries, and activation — completes without failure when running GPT-5.4-mini at HIGH effort. Round 6 produced 8 consecutive absolute clean passes (R77-R84), the longest unbroken streak in the entire PR2 dataset, and achieved an 83% clean pass rate overall.

The remaining instability is isolated to the **criteria builder phase** (GPT-5-mini). The criteria loop exits prematurely in 60% of criteria-attempted runs, and P6 context loss occurs in ~16% of runs after the criteria builder hands back to the base model. These are the only two failure modes that persist at HIGH effort — all other issues (states skip, P5 regression, dead buttons) have been eliminated.

The recommended production configuration is **GPT-5.4-mini HIGH effort base + GPT-5-mini medium effort criteria/webhook**. The criteria loop premature exit is the dominant remaining instability — a `draft_delivery_account_criteria` MCP helper would be the primary stabilization mechanism (see `mcp-optimization-options.md`).

---

## Summary

**Confirmed working (all 69 valid runs):**
- Client creation (P1 prompt + P2 lead type drop): 69/69 (100%)
- Schedule collection (P3): 68/69 (98.6%) -- R75 reversed schedule/type order
- Delivery type label "Webhook"/"Portal"/"Email"/"FTP" (P3/P4): 69/69 (100%)
- Field mapping pipeline (webhook runs with mapping): 12/12 runs mapped successfully, 8-11 fields each
- Price collection (P5): 68/69 (98.6%) -- R75 defaulted price
- Exclusivity + Order System (P5): 68/69 (98.6%) -- R69 skipped both
- States collection (P5): 60/69 (87.0%) -- 9 runs skipped states prompt
- Phase 6 Yes/No boolean labels: correct in all runs where P6 rendered
- Phase 8 activation: 69/69 (100%)

**Key statistics:**
- Average score: **93.1%**
- Achievable clean pass rate (100%): **31/69 (44.9%)**
- Criteria full collection rate (3/3): **14/35 criteria-attempted (40.0%)**
- States collection rate: **60/69 (87.0%)**
- Round-over-round pass rate: R1-4 36.4% --> R5 37.5% --> R6 **83.3%**

**Key pattern:** HIGH effort base model (Round 6) eliminates states skip (0/12) and P5 regression (0/12). Remaining failures are criteria loop premature exit and P6 context loss, both in the GPT-5-mini criteria builder phase.

---

## Model Configuration

| Config | Rounds 1-4 | Round 5 | Round 6 |
|--------|-----------|---------|---------|
| **Base model** | GPT-5.4-mini medium effort | GPT-5.4-mini medium effort | GPT-5.4-mini **HIGH** effort |
| **Criteria/webhook model** | GPT-5.4-mini high effort | GPT-5-mini medium effort | GPT-5-mini medium effort |
| **Environment** | Production | Dev mirror | Dev mirror |

---

## Combined Scoring Matrix

| Rnd | Run | Client | Mapping | Mapped | respSearch | Price | Excl | Order | States | Gate | Crit out | P6 out | Score % | Outcome | Class |
|-----|-----|--------|---------|--------|------------|-------|------|-------|--------|------|----------|--------|---------|---------|-------|
| 1-4 | 01 | Y | Y | 8/8 | - | Y | Y | Y | Y | Y | 2/3 | Y | 86 | 86% | IGNORE |
| 1-4 | 02 | Y | Y | 10/10 | - | Y | Y | Y | Y | Y | 1/3 | Y | 90 | 90% | IGNORE, PLATFORM |
| 1-4 | 03 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 04 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 07 | Y | Y | 8/8 | - | Y | Y | Y | **F** | - | skip | Y | 87 | 87% | IGNORE, HALLUCINATE |
| 1-4 | 08 | Y | - | - | - | Y | Y | Y | Y | Y | 1/3 | Y | 88 | 88% | IGNORE, PLATFORM |
| 1-4 | 09 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 10 | Y | Y | 9/9 | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 11 | Y | - | - | - | Y | Y | Y | Y | Y | 1/3 | Y | 88 | 88% | IGNORE, PLATFORM |
| 1-4 | 12 | Y | Y | 8/8 | - | Y | Y | Y | Y | **F** | skip | Y | 87 | 87% | IGNORE |
| 1-4 | 13 | Y | - | - | - | Y | Y | Y | Y | **F** | skip | Y | 93 | 93% | IGNORE |
| 1-4 | 15 | Y | - | - | - | Y | Y | Y | Y | Y | 1/3 | Y | 88 | 88% | IGNORE |
| 1-4 | 17 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 19 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 20 | Y | Y | 9/9 | - | Y | Y | Y | Y | Y | 1/3 | Y | 86 | 86% | IGNORE |
| 1-4 | 21 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 22 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED |
| 1-4 | 23 | Y | - | - | - | Y | Y | Y | Y | Y | 1/3 | Y | 88 | 88% | IGNORE, PLATFORM |
| 1-4 | 25 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 26 | Y | Y | 9/9 | - | Y | Y | Y | Y | Y | 2/3 | Y | 91 | 91% | IGNORE |
| 1-4 | 27 | Y | Y | 9/9 | - | Y | Y | Y | Y | Y | 1/3 | Y | 86 | 86% | IGNORE, PLATFORM |
| 1-4 | 28 | Y | Y | 11/11 | - | Y | Y | Y | Y | Y | 1/3 | Y | 86 | 86% | IGNORE |
| 1-4 | 29 | Y | Y | 9/9 | - | Y | Y | Y | Y | Y | 2/3 | Y | 91 | 91% | IGNORE |
| 1-4 | 30 | Y | - | - | - | Y | Y | Y | Y | Y | 2/3 | Y | 91 | 91% | IGNORE, PLATFORM |
| 1-4 | 34 | Y | Y | 9/9 | - | Y | Y | Y | Y | Y | 2/3 | Y | 91 | 91% | IGNORE |
| 1-4 | 35 | Y | Y | 10/10 | - | Y | Y | Y | Y | Y | 1/3 | Y | 86 | 86% | IGNORE |
| 1-4 | 37 | Y | - | - | - | Y | Y | Y | Y | **F** | skip | **F** | 80 | 80% | IGNORE |
| 1-4 | 40 | Y | - | - | - | Y | Y | Y | Y | Y | 2/3 | **F** | 82 | 82% | IGNORE |
| 1-4 | 43 | Y | - | - | - | Y | Y | Y | Y | Y | 1/3 | Y | 86 | 86% | IGNORE |
| 1-4 | 44 | Y | - | - | - | Y | Y | Y | Y | - | skip | **F** | 93 | 93% | PLATFORM |
| 1-4 | 45 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 46 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 1-4 | 47 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 5 | 48 | Y | - | - | - | Y | Y | Y | Y | **F** | skip | Y | 93 | 93% | IGNORE |
| 5 | 50 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED |
| 5 | 51 | Y | - | - | - | Y | Y | Y | **F** | Y | 1/3 | Y | 73 | 73% | IGNORE |
| 5 | 52 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 5 | 53 | Y | - | - | - | Y | Y | Y | **F** | - | skip | Y | 93 | 93% | IGNORE, HALLUCINATE |
| 5 | 54 | Y | - | - | - | Y | Y | Y | **F** | Y | 2/3 | Y | 82 | 82% | IGNORE |
| 5 | 55 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 5 | 56 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 5 | 57 | Y | - | - | - | Y | Y | Y | **F** | - | skip | Y | 93 | 93% | IGNORE, HALLUCINATE |
| 5 | 58 | Y | - | - | - | Y | Y | Y | Y | Y | 1/3 | **F** | 77 | 77% | IGNORE |
| 5 | 59 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 5 | 60 | Y | - | - | - | Y | Y | Y | **F** | - | skip | Y | 93 | 93% | IGNORE |
| 5 | 61 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 5 | 62 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED, HALLUCINATE |
| 5 | 64 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | **F** | 91 | 91% | IGNORE |
| 5 | 66 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | **F** | 91 | 91% | IGNORE |
| 5 | 67 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED |
| 5 | 68 | Y | - | - | - | Y | Y | Y | **F** | Y | **3/3** | Y | 91 | 91% | IGNORE, HALLUCINATE |
| 5 | 69 | Y | - | - | - | Y | **F** | **F** | Y | Y | **3/3** | Y | 91 | 91% | IGNORE |
| 5 | 70 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | **F** | 95 | 95% | IGNORE, HALLUCINATE |
| 5 | 71 | Y | - | - | - | Y | Y | Y | **F** | Y | 1/3 | **F** | 73 | 73% | IGNORE |
| 5 | 72 | Y | - | - | - | Y | Y | Y | **F** | Y | **3/3** | Y | 95 | 95% | IGNORE, HALLUCINATE |
| 5 | 73 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED |
| 5 | 74 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | **F** | 95 | 95% | IGNORE |
| 6 | 75 | Y | - | - | **F** | **F** | Y | Y | Y | - | skip | **F** | 79 | 79% | IGNORE |
| 6 | 77 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 78 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 79 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 80 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 81 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 82 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 83 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 84 | Y | - | - | - | Y | Y | Y | Y | - | skip | Y | 100 | PASS | CLOSED |
| 6 | 87 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED |
| 6 | 88 | Y | - | - | - | Y | Y | Y | Y | Y | 2/3 | **F** | 73 | 73% | IGNORE |
| 6 | 90 | Y | - | - | - | Y | Y | Y | Y | Y | **3/3** | Y | 100 | PASS | CLOSED |

**Legend:** Y = pass | **F** = fail (data integrity) | `-` = not applicable / not tested | skip = intentionally skipped by test plan | **3/3** = full criteria collection

---

## Classification Scoring

| Class | Count | % | Runs |
|-------|------:|--:|------|
| CLOSED | 31 | 44.9 | R03 (M:40652/A:49884), R04 (M:40653/A:49885), R09 (M:40658/A:49889), R10 (M:40659/A:49890), R17 (M:40665/A:49896), R19 (M:40668/A:49899), R21 (M:40671/A:49902), R22 (M:40672/A:49903), R25 (M:40675/A:49905), R45 (M:40695/A:49921), R46 (M:40696/A:49922), R47 (M:40697/A:49923), R50 (M:47094/A:46080), R52 (M:47097/A:46083), R55 (M:47100/A:46086), R56 (M:47101/A:46087), R59 (M:47104/A:46091), R61 (M:47106/A:46093), R62 (M:47107/A:46094), R67 (M:47112/A:46096), R73 (M:47118/A:46103), R77 (M:47121/A:46106), R78 (M:47122/A:46107), R79 (M:47123/A:46108), R80 (M:47124/A:46109), R81 (M:47125/A:46110), R82 (M:47126/A:46111), R83 (M:47127/A:46112), R84 (M:47128/A:46113), R87 (M:47131/A:46116), R90 (M:47134/A:46119) |
| IGNORE | 37 | 53.6 | 01, 02, 07, 08, 11, 12, 13, 15, 20, 23, 26, 27, 28, 29, 30, 34, 35, 37, 40, 43, 48, 51, 53, 54, 57, 58, 60, 64, 66, 68, 69, 70, 71, 72, 74, 75, 88 |
| HALLUCINATE | 7 | 10.1 | 07, 53, 57, 62, 68, 70, 72 |
| PLATFORM | 7 | 10.1 | 02, 08, 11, 23, 27, 30, 44 |

Runs may carry multiple classifications. CLOSED = fully verified clean pass (Method/Account IDs shown). IGNORE = agent skipped a required instruction. HALLUCINATE = agent invented/fabricated values not provided by user. PLATFORM = card non-render, dead buttons, infrastructure issues.

---

## Severity Breakdown

| Severity | Runs | Count | Root Cause |
|----------|------|------:|------------|
| **P0 HIGH (data integrity)** | 07, 51, 53, 54, 57, 60, 68, 71, 72 | 9 | States prompt skipped -- agent ignored CRITICAL STOP guard before states question, then either hallucinated state values from example text (07, 53, 57, 68, 72) or left states empty (51, 54, 60, 71). GPT-5.4-mini medium effort only; 0 occurrences in Round 6 HIGH effort. |
| **P0 HIGH (data integrity)** | 69 | 1 | Exclusivity AND Order System both skipped -- agent jumped from price directly to states, bypassing two consecutive WAIT points. |
| **P1 HIGH (criteria integrity)** | 01, 02, 08, 11, 15, 20, 23, 26, 27, 28, 29, 30, 34, 35, 40, 43, 51, 54, 58, 71, 88 | 21 | PRD-R2-LOOP: criteria loop exits prematurely before all planned criteria collected. 1/3 in 13 runs, 2/3 in 8 runs. Only 14/35 criteria-attempted runs achieved 3/3. |
| **P3 Medium (gate skip)** | 12, 13, 37, 48 | 4 | PRD-R2-GATESKIP: criteria gate card never displayed. Agent jumped directly to account creation or P6 without asking about additional criteria. Safe default (None) applied. |
| **P3 Medium (P6 context loss)** | 37, 40, 44, 58, 64, 66, 70, 71, 74, 75, 88 | 11 | P6 summary missing fields (price, exclusivity, states, criteria show "Not provided"), pipe-delimited state UIDs instead of abbreviations, or P6 skipped entirely. Context lost during summarize_history between criteria exit and P6. |
| **P5 Cosmetic** | 02, 08, 10, 11, 23, 27, 30, 44 | 8 | PRD-R2-DEADBTN: card buttons render but clicks do not register. Typed fallback always works. P4 text-only rendering in R10, R45-R47, R66, R75. No data impact. |

---

## Cross-Round Observations

### Rounds 1-4 --> Round 5: criteria model swap

| Metric | Rounds 1-4 | Round 5 | Delta |
|--------|-----------|---------|-------|
| Config change | -- | Criteria model: GPT-5.4-mini high --> GPT-5-mini medium | Slower, larger model for criteria |
| Valid runs | 33 | 24 | |
| Average score | 92.2% | 92.8% | +0.6 |
| Clean pass rate | 12/33 (36.4%) | 9/24 (37.5%) | +1.1 |
| 3/3 criteria rate | 1/17 (5.9%) | 11/15 (73.3%) | **+67.4** |
| States skip rate | 1/33 (3.0%) | 8/24 (33.3%) | **+30.3 (regression)** |
| P5 regression (P2 re-entry) | 1 excluded (R41) | 2 excluded (R63, R65) | persistent |
| P6 context loss | 2/33 (6.1%) | 6/24 (25.0%) | **+18.9 (new pattern)** |

**What improved:** Criteria collection jumped from 5.9% to 73.3% full collection. GPT-5-mini reliably completes the 3-criterion loop when given exact field names.

**What regressed:** States skip worsened from 3.0% to 33.3%. This is a GPT-5.4-mini base model issue on the dev environment, not attributable to GPT-5-mini. The P5 regression pattern (agent regresses to P2 lead type after exclusivity answer) persisted. P6 context loss emerged as a new failure mode caused by summarize_history between criteria builder exit and P6 summary rendering.

### Round 5 --> Round 6: HIGH effort base model

| Metric | Round 5 | Round 6 | Delta |
|--------|---------|---------|-------|
| Config change | -- | Base effort: medium --> **HIGH** | Higher compute budget for base model |
| Valid runs | 24 | 12 | |
| Average score | 92.8% | 96.0% | +3.2 |
| Clean pass rate | 9/24 (37.5%) | 10/12 (83.3%) | **+45.8** |
| 3/3 criteria rate | 11/15 (73.3%) | 2/3 (66.7%) | -6.6 (small sample) |
| States skip rate | 8/24 (33.3%) | 0/12 (0.0%) | **-33.3 (eliminated)** |
| P5 regression (P2 re-entry) | 2 excluded | 0 excluded | **eliminated** |
| P6 context loss | 6/24 (25.0%) | 2/12 (16.7%) | -8.3 |
| GPT-5-mini stalls | 0 | 2 excluded (R85, R86) | new (infra) |

**What improved:** States skip dropped to zero. P5 regression dropped to zero. Runs R77-R84 produced 8 consecutive absolute clean passes -- the longest streak in PR2. R87 and R90 also passed with full 3/3 criteria. Overall pass rate nearly doubled.

**What remained:** Criteria loop premature exit (R88 at 2/3). P6 context loss (R75, R88). GPT-5-mini stalls emerged as a new infrastructure concern (2/12 runs timed out during criteria phase).

---

## Per-Round Detail

### Rounds 1-4 (R01-R47, production)
**Config:** GPT-5.4-mini medium effort base + GPT-5.4-mini high effort criteria/webhook  
**Valid runs:** 33 | **Pass rate:** 12/33 (36.4%) | **Avg score:** 92.2%

Established the production baseline. The dominant issue was PRD-R2-LOOP -- only 1/17 criteria-attempted runs achieved 3/3 collection (R22, the first full collection in the dataset). States collection was strong at 97.0% (32/33), with only R07 exhibiting a states skip + hallucination from example text. Gate skips appeared in R12, R13, and R37 (10%). Mapping pipeline performed well across JSON (8-11 fields), URL-encoded, and XML formats. Late runs (R43-R47) showed increasing P4 text-only rendering but no data impact.

### Round 5 (R48-R74, dev mirror)
**Config:** GPT-5.4-mini medium effort base + GPT-5-mini medium effort criteria/webhook  
**Valid runs:** 24 | **Pass rate:** 9/24 (37.5%) | **Avg score:** 92.8%

Criteria collection improved dramatically with GPT-5-mini: 11/15 criteria-attempted runs achieved 3/3 (73.3%). However, states skip regressed severely to 33.3% (8/24 runs) -- attributable to the GPT-5.4-mini base model on the dev environment, not the criteria model swap. Three runs exhibited the P5 regression pattern (agent regresses to P2 after exclusivity answer); R63 and R65 were excluded as corrupted. P6 context loss appeared in 6 runs, a new failure mode at the summarize_history boundary between criteria exit and P6.

### Round 6 (R75-R90, dev mirror)
**Config:** GPT-5.4-mini **HIGH** effort base + GPT-5-mini medium effort criteria/webhook  
**Valid runs:** 12 | **Pass rate:** 10/12 (83.3%) | **Avg score:** 96.0%

Transformative results. Zero states skips. Zero P5 regressions. The first run (R75) showed teething issues (schedule reversed, price defaulted, responseSearch skipped) but R77-R84 produced 8 consecutive absolute clean passes -- the longest streak in PR2. R87 and R90 passed with full 3/3 criteria. Only R88 (2/3 criteria + P6 context loss at 73%) and R75 (first-run anomalies at 79%) did not pass. Two GPT-5-mini stalls (R85, R86) were excluded as infrastructure timeouts.

---

## Remaining Issues

| # | Finding | Frequency | Severity | Classification |
|---|---------|-----------|----------|----------------|
| PRD-R2-LOOP | Criteria loop premature exit -- loop exits before all planned criteria collected, typically after 1st or 2nd criterion | 21/35 criteria runs (60.0%) | HIGH | IGNORE |
| PRD-R2-STATESKIP | States prompt skipped -- CRITICAL STOP guard ignored, states question never asked (medium effort only) | 9/69 (13.0%); 0% in R6 | HIGH | IGNORE |
| PRD-R2-STATEHALL | States hallucinated from example text -- when states skipped, agent copies "CA, AZ, TX" from prompt example or defaults to "CA" | 5/9 states-skip runs (55.6%) | HIGH | HALLUCINATE |
| PRD-R2-P6LOSS | P6 context loss -- P6 summary missing price/states/criteria, pipe-delimited UIDs, or P6 skipped entirely | 11/69 (15.9%) | Medium | IGNORE |
| PRD-R2-GATESKIP | Criteria gate skipped -- gate card never displayed, agent jumps to account creation | 4/69 (5.8%); 0% in R6 | Medium | IGNORE |
| PRD-R2-P5REG | P5 regression (P2 re-entry) -- agent regresses to Phase 2 lead type after exclusivity answer | 3 excluded (R41, R63, R65); 0% in R6 | HIGH | IGNORE |
| PRD-R2-STALL | GPT-5-mini criteria stall -- criteria builder stalls 90s+ after CR1, platform timeout | 2/12 R6 runs (R85, R86) | Medium | PLATFORM |
| PRD-R2-DEADBTN | Dead buttons / card non-render -- buttons render but clicks do not register; typed fallback works | 8/69 (11.6%) | Cosmetic | PLATFORM |
| PRD-R2-HALLPROMPT | Hallucinated TOOL_DEFAULTS prompts -- agent asks for account name/status after criteria exit instead of auto-applying defaults | R62, R70 (2/69) | Low | HALLUCINATE |
| PRD-R2-P4TEXT | P4 text-only summary -- Phase 4 confirmation renders as plain text instead of Adaptive Card | R10, R45, R46, R47, R66, R75 (6/69) | Cosmetic | PLATFORM |
| PRD-R2-CTSKIP | Content type card skipped after typed fallback -- typed "I'll provide instructions" collapses content type selection | R01 (1/69) | Medium | IGNORE |
| PRD-R2-REDETECT | Unnecessary format re-detection after explicit JSON selection | R10 (1/69) | Low | IGNORE |

---

## Dominant Pattern

**The criteria loop premature exit (PRD-R2-LOOP) is the single highest-frequency defect at 60% of criteria-attempted runs**, persisting across all model configurations and effort levels. HIGH effort base model resolves the two most critical base-model failures (states skip and P5 regression) but does not fix criteria loop behavior, which is governed by the GPT-5-mini criteria builder phase.

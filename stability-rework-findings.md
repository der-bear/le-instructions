# Stability Test Findings — Version B: Rework

**Action:** Create Single Client (Reworked)  
**Protocol:** stability-test-protocol.md  
**Base test data:** StabilityTest-{RR}r, stability{RR}r-{TS}@test.com (R1); StabilityTest-{RR}r2, stability{RR}r2@test.com (R2)

**Rounds:**
- **R1:** Runs 01–12 (plus 13–30), gpt-5.4-mini all phases (runs 01–20); GPT-5-mini for Webhook/criteria phases, GPT-5.4 for others (runs 21–30)
- **R2:** Runs 01–10, gpt-5.4-mini all phases (post-fix, commit 5200cc3)

---

## Run Scenarios

### R1 Run Scenarios

| Run | Scenario Focus | Delivery | Content Type | Schedule | Criteria | Special |
|-----|---------------|----------|--------------|----------|----------|---------|
| 01 | Baseline | Webhook | JSON (imperfect) | Mon-Fri 9-5 PST | 3 (numeric) | — |
| 02 | Baseline (repeat) | Webhook | JSON (imperfect) | Mon-Fri 9-5 PST | 3 (numeric) | — |
| 03 | Baseline (repeat) | Webhook | JSON (imperfect) | Mon-Fri 9-5 PST | 3 (numeric) | — |
| 04 | Complex mapping | Webhook | JSON (large body, many fields) | Mon-Fri 9-5 PST | 5 criteria (numeric + enum + text) | Exclusive, Order System ON |
| 05 | URL Encoded + enum criteria | Webhook | URL Encoded | 24/7 | 3 (enum-heavy) | Shared |
| 06 | XML + auto-detect | Webhook | "I'm not sure" (XML) | Weekdays 8am-6pm EST | 4 criteria | Exclusive |
| 07 | Complex posting instructions | Webhook | JSON (complex multi-level body) | Mon-Wed-Fri only | 6 criteria | Large requestBody |
| 08 | FTP delivery | FTP | N/A | 24/7 | 2 criteria | Connection test |
| 09 | Portal delivery | Portal | N/A | Specific hours | 3 criteria | No connection test |
| 10 | Email delivery + max criteria | Email | N/A | Mon-Fri 9-5 PST | 7 criteria (all types) | Exclusive, Order ON |
| 21 | 2 enum criteria | Webhook | JSON (5 fields) | 24/7 | 2 enum | Exclusive, Order ON |
| 22 | 1 numeric criterion | Portal | N/A | Mon-Fri 9am-5pm EST | 1 numeric | Shared, Order OFF |
| 23 | Mixed criteria | Webhook | XML | Weekdays 8am-6pm CST | 2 enum + 1 numeric | Exclusive, Order ON |
| 24 | No criteria | Email | N/A | 24/7 | none (skip) | Shared, Order OFF |
| 25 | FTP + mixed criteria | FTP | N/A | Mon-Fri 9am-5pm MST | 1 enum + 1 numeric | Exclusive, Order ON |
| 26 | URL Encoded + numeric | Webhook | URL Encoded | Mon-Wed-Fri 8am-8pm PST | 1 numeric | Shared, Order OFF |
| 27 | Portal + 2 enum | Portal | N/A | 24/7 | 2 enum | Exclusive, Order ON |
| 28 | JSON nested + 1 enum | Webhook | JSON nested | Tue-Thu 9am-6pm EST | 1 enum | Shared, Order ON |
| 29 | Email + 2 numeric | Email | N/A | Mon-Fri 8am-5pm CST | 2 numeric | Exclusive, Order OFF |
| 30 | FTP + 1 enum | FTP | N/A | 24/7 | 1 enum | Shared, Order ON |

### R2 Run Scenarios

| Run | Scenario | Delivery | Content Type | Schedule | Criteria | Special |
|-----|----------|----------|-------------|----------|----------|---------|
| 01 | JSON imperfect baseline | Webhook | JSON (missing braces, trailing comma, single quotes) | Mon-Fri 9-5 PST | 3: numeric >=, enum, numeric > | LendingTree |
| 02 | JSON nested complex | Webhook | JSON (deep nesting, 15+ fields) | Tue-Thu 9-6 EST | 5: 2 enum + 2 numeric + 1 between | Exclusive, Order ON |
| 03 | Content mismatch XML→JSON | Webhook | XML → switch → JSON | 24/7 | 2 enum + 1 numeric | Recovery flow |
| 04 | Content mismatch JSON→XML | Webhook | JSON → switch → XML | Mon-Fri 8-6 CST | 3: enum + numeric + between | Reverse mismatch |
| 05 | URL Encoded + enum-heavy | Webhook | URL Encoded | 24/7 | 4: 3 enum + 1 numeric | Shared |
| 06 | Auto-detect (XML body) | Webhook | "I'm not sure" | Mon-Wed-Fri 8-8 PST | 2 enum | Exclusive |
| 07 | Auto-detect (JSON body) | Webhook | "I'm not sure" | 24/7 | 3: numeric >= + enum + numeric > | Shared, Order ON |
| 08 | FTP + mixed criteria | FTP | N/A | Mon-Fri 9-5 PST | 2: 1 enum + 1 numeric | Connection test |
| 09 | Portal + skip criteria | Portal | N/A | Specific hours | Skip | No conn test |
| 10 | Email + no criteria | Email | N/A | 24/7 | Skip | Shared |

---

## Combined Scoring Matrix

| Run | Round | P1-PROMPT | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-------|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | F | F | Y | Y | Y | Y | FAIL (20/22=91%) | States normalized ✓; RB-A: criteria loop premature exit after 1 criterion |
| 02 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | N/A | F | F | Y | Y | Y | Y | FAIL (17/22=77%) | States not normalized; criteria prompt entirely skipped; acct name asked before price |
| 03 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | Y | Y | Y | Y | Y | Y | PASS (22/22=100%) | States normalized ✓; RB-F: Phase 5c failed to load on 1st click, worked on 2nd attempt |
| 04 | R1 | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | N/A | F | F | Y | Y | Y | Y | FAIL (18/22=82%) | RB-G: prompts duplicated (P1+P3+P5); RB-H: content type plain text (recovered post-DEBUG); RB-A: criteria loop exited after 1 criterion; RB-C: states not normalized; RB-F NOT triggered ✓ |
| 05 | R1 | F* | Y | F | Y | F | N/A | N/A | Y | F | Y | Y | Y | Y | F | Y | Y | F | F | F | Y | F | Y | Y | FAIL (13/21=62%) | RB-G: P1+P2+P3 duplicated; RB-L: content type skipped; RB-C: states not normalized; RB-K: Phase 4 summary skipped; RB-H: criteria gate no card; RB-A variant: loop control keyword as field name; RB-M: Credit >=700 stored as Equal; LoanType criterion lost |
| 06 | R1 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | N/A | F | F | Y | Y | Y | Y | FAIL (19/22=86%) | RB-G: P1+P2 duplicated; RB-F: Phase 5c load failed (stuck 2 attempts, DEBUG required); RB-A: loop exit after 1 criterion; XML auto-detect ✓; 10/10 mappings ✓; P4-SUMM ✓; RB-K NOT triggered ✓; RB-C NOT triggered (USPS codes) ✓ |
| 07 | R1 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | F | Y | N/A | F | F | Y | F | Y | Y | FAIL (16/21=76%) | RB-G: P1+P2+P3+P4+P5 all duplicated; RB-N: states question skipped entirely; RB-O: phantom states CA,AZ,TX in account; RB-F: Phase 5c failed 2x, DEBUG+explicit URL required; RB-A: loop exit after 1 criterion; 16/20 fields mapped ✓; P4-SUMM card ✓; Phase 8 activation card ✓ |
| 08 | R1 | F* | Y | F | Y | N/A | N/A | N/A | Y | Y | Y | F | Y | N/A | N/A | F | Y | N/A | F | F | Y | Y | Y | Y | FAIL (12/17=71%) | RB-G systematic; FTP conn test correct ✓; P5-EXCL plain text; RB-K: acct name before price; update_delivery_account fails (RB-P new); "Show more fields" btn broken (RB-Q new) |
| 09 | R1 | F* | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | F | F | Y | Y | N/A | Y | FAIL (15/18=83%) | RB-G systematic; P5 all plain text (RB-A); P3b correctly skipped for Portal ✓; P5c conversational: numeric criteria silently saved without UI (RB-S new); Portal deliveryType=HttpPost (RB-R, confirmed via log); P6 confused state on "continue" (RB-T new); criteria overwrites (RB-U new) |
| 10 | R1 | F* | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | Y | F | F | N/A | F | F | Y | Y | Y | Y | FAIL (14/19=74%) | RB-G: P1+P5-STATE duplicated; Email delivery ✓; P4-SUMM ✓; P5-EXCL/ORDER cards correct ✓; States CA/TX/FL/NY/NV normalized ✓; criteria gate entirely skipped (RB-D); DEBUG ignored mid-flow; activation ✓ |
| 11 | R1 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | F | F | N/A | F | F | F | F | Y | Y | FAIL (14/21=67%) | RB-G: P5 entry duplicated; RB-K: acct name before price; content type card ✓; 8/8 mapping ✓; conn test fail handled ✓; P4-SUMM ✓; RB-D: states/criteria gate skipped→phantom CA,AZ,TX; RB-N/RB-O; RB-T: DEBUG caused re-ask of exclusivity; conversational criteria accepted verbally but not saved (RB-V new); state NY dropped in payload (3/4 states); activation ✓ |
| 12 | R1 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | N/A | N/A | F | F | N/A | N/A | N/A | N/A | FAIL (14/17=82%) | LendingTree, Webhook, JSON, Mon-Fri 9-5 EST, Exclusive, Order ON; RB-G: P1 prompt doubled + states doubled; P3-P4 clean ✓; 10/10 mapping with disambiguation ✓; conn test fail handled ✓; P4-SUMM ✓; price first ✓ (RB-K not triggered); criteria gate fired ✓ (RB-D not triggered); RB-X NEW: P5c enters empty-response loop — agent returns empty response after 14+ min; context overflow from LendingTree 100+ fields + P5c instructions; unrecoverable; session 69d18c262697c9202c0bc293 |
| 13 | R1 | F* | Y | Y | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | ABORT | LendingTree, Webhook, 24/7; RB-G: P1 prompt doubled; RB-Y NEW: Phase 3 asked name+type via plain text (duplicated) BEFORE showing schedule/type cards — unusual ordering; test aborted at P3 (protocol error: webhook URL sent prematurely before agent asked → state rollback to schedule question) |
| 14 | R1 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL (16/18=89%) | LendingTree, Webhook, Mon-Fri 9-5 PST, JSON, Shared, No order; RB-G: P1 doubled; RB-K: acct name before price; RB-N: states silently skipped (3rd: B07,B11,B14); RB-O: phantom CA,AZ,TX 3rd occurrence; RB-D: criteria gate auto-skipped without asking user; criteria protocol=skip (RB-X avoidance); 8/8 mapped, SelfCreditRating for credit_score |
| 15 | R1 | F* | Y | F | Y | N/A | Y | Y | Y | Y | Y | F | F | F | N/A | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL (12/17=71%) | LendingTree, Webhook, 24/7, URL-Encoded, Exclusive, Order ON; RB-G: P1 doubled; RB-Y (2nd): delivery type asked plain-text BEFORE schedule card—then delivery type card shown again (double-ask, inverted order); RB-H: P5-EXCL+P5-ORDER both plain text; RB-N: states silently skipped (4th: B07,B11,B14,B15); RB-O: phantom CA,AZ,TX (4th); RB-D: criteria gate also skipped; acct name auto-generated (not asked, unlike RB-K); 8/8 URL-Encoded mapping+disambiguation ✓ |
| 17 | R1 | F* | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL (13/13=100%) | LendingTree, Portal, 24/7, Exclusive, Order ON, skip criteria; RB-G: P1 doubled + states question doubled; RB-K NOT triggered (price first ✓); RB-N NOT triggered (states asked — 1st time with Order=ON ✓); RB-O NOT triggered ✓; RB-D NOT triggered (criteria gate shown — 1st time ✓); P3b correctly skipped for Portal ✓; P4/P6/P7 summary cards all correct ✓; Activation confirmed ACTIVE ✓ |
| 16 | R1 | F | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | N/A | F | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | ABORT | LendingTree, Webhook, Mon-Fri 8am-6pm CST, JSON, Shared, Order OFF; RB-G: P5 doubled; RB-Z NEW: Phase 8 credentials (acct name/username/password/criteria) asked immediately after P4 Continue BEFORE Phase 5 price/excl/order/states — Phase 5 only loaded after credentials provided; RB-N: states never asked (5th: B07,B11,B14,B15,B16); RB-D: criteria gate absent — criteria builder auto-entered with "Please select a Lead Type" prompt; Phase 5 restart loop on "skip" — unrecoverable; ABORT; 12/12 JSON mapping ✓ |
| 18 | R1 | F* | Y | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL (13/14=93%) | LendingTree, Email, Mon-Fri 9am-5pm EST, Shared, Order OFF, skip criteria; RB-G: P1+P3-SCHED+P3-DTYPE+P4-Continue all doubled (systematic); RB-K(mild): acct name asked between P4-Continue and Phase 5 load (before price/excl/order); RB-N NOT triggered (states asked CA,TX,FL ✓); RB-D NOT triggered (criteria gate shown ✓); RB-O: stateUIDs need session log verification; Email conn test correctly skipped ✓; P4/P6/P7 correct; Activation ACTIVE ✓ |
| 19 | R1 | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | N/A | N/A | Y | F | Y | Y | FAIL (12/14=86%) | LendingTree, FTP, Mon-Fri 8am-6pm CST, Exclusive, Order OFF, skip criteria; RB-G(partial): P4-Continue doubled only — P1/P3-SCHED/P3-DTYPE all clean (mildest RB-G yet); RB-Z(2nd: B16,B19): full credentials (acct name+user+pass+criteria) collected before Phase 5; skip→Phase 5 loaded correctly (no loop unlike B16) ✓; RB-N: states silently skipped (6th: B07,B11,B14,B15,B16,B19); RB-O: phantom CA,AZ,TX (6th); RB-D: criteria gate bypassed (pre-Phase-5 skip used); FTP conn test fail handled ✓; P4/P7 correct; Activation ACTIVE ✓ |
| 20 | R1 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL (18/18=100%) | LendingTree, Webhook, 24/7, URL-Encoded, Shared, Order OFF, skip criteria; RB-G: P1+states prompt+field-mapping-Continue+P4-Continue all doubled (systematic); RB-K NOT triggered (price first ✓); RB-Z NOT triggered ✓; RB-N NOT triggered (states asked FL,OH,GA ✓); RB-O NOT triggered (correct states in P6 ✓); RB-D NOT triggered (criteria gate w/Add/Skip buttons ✓); account name auto-generated; 6/7 URL-encoded mapping ✓; P4/P6/P7 correct; Activation ACTIVE ✓ |
| 21 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | FAIL (21/23=91%) | LendingTree, Webhook/JSON, 24/7, Exclusive, Order ON, 2 enum; enum bypass on BOTH (SelfCreditRating+LoanRequestPurpose, no ChoiceSet); **Finding U**: both enum criteria dropped from payload (only state criterion present: CA(5)+TX(44)+FL(10)); States correct position ✓; 8/8 mapping ✓; RB-G NOT triggered ✓; Session:69d2778a2697c9202c0bcf1c; Method:46937, Acct:45940; ACTIVE ✓ |
| 22 | R1 | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | F | F | F | F | N/A | F | F | Y | Y | Y | Y | FAIL (11/17=65%) | LendingTree, Portal/Mon-Fri 9-5 EST, Shared, Order OFF; RB-N: states prompt skipped; RB-O: phantom CA(5)+AZ(3)+TX(44); RB-D: criteria gate skipped; account created immediately after Order=No; RB-G NOT triggered ✓; P3b correctly skipped for Portal ✓; P4-SUMM ✓; Session:69d27b252697c9202c0bcf7f; Method:46938, Acct:45941; ACTIVE ✓ |
| 23 | R1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | F | Y | Y | F | Y | Y | Y | FAIL (19/23=83%) | LendingTree, Webhook/XML, Weekdays 8-6 CST, Exclusive, Order ON, 2 enum+1 numeric; RB-N: states prompt skipped; RB-O: phantom CA(5)+AZ(3)+TX(44) in initial create; NEW: criteria added via SEPARATE update_delivery_account after create (not in initial payload); enum bypass: SelfCreditRating→correct enum UID(343593) via update; LoanRequestPurpose→"Contains"+"purchase" (wrong operator+value, should be "In"+enumUID); LoanAmount→GreaterOrEqual 100000 ✓; 8/8 mapping ✓; RB-G NOT triggered ✓; Session:69d27c502697c9202c0bcfb5; Method:46939, Acct:45942; ACTIVE ✓ |
| 24 | R1 | Y | F | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | Y | Y | Y | Y | Y | FAIL (13/15=87%) | LendingTree, Email/24/7, Shared, Order OFF, no criteria; RB-AA NEW: P2 timezone/status skipped after duplicate-email retry — defaulted PST/-8; P5-FIELD: field suggestions step skipped (bare gate only); FL(10)+GA(11) in payload ✓; no phantom states ✓; RB-G NOT triggered ✓; RB-N NOT triggered ✓; Session:69d2869f2697c9202c0bd088; Method:46940, Acct:45943; ACTIVE ✓ |
| 25 | R1 | Y | F | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | F | F | F | F | F | F | Y | F | Y | Y | FAIL (10/19=53%) | LendingTree, FTP/Mon-Fri 9-5 MST, Exclusive, Order ON, 1 enum+1 numeric; RB-AA(2nd): P2 skipped; RB-N: states skipped; RB-O: phantom CA(5)+AZ(3)+TX(44); RB-D: criteria gate skipped entirely (states+criteria both missed after Order=Yes); P6-SUMM skipped (went P5→P7); 0/0 FTP field mappings ✓; P3B conn test card shown ✓; RB-G NOT triggered ✓; Session:69d288ef2697c9202c0bd0c5; Method:46941, Acct:45944; ACTIVE ✓ |
| 26 | R1 | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | N/A | F | F | N/A | F | F | F | Y | Y | Y | FAIL (12/21=57%) | LendingTree, Webhook/URL-Encoded Mon-Wed-Fri 8-8 PST, Shared, Order OFF, 1 numeric; RB-AA(3rd): P2 skipped; RB-BB NEW: P5 off-script pre-price prompt (account name+criteria asked, self-corrected); P5 collapsed: only price collected then immediate acct creation — Exclusive(wrong), Order=Yes(wrong), CA only(wrong), no criteria; P6-BOOL F (wrong bool values); P6-SUMM shown (wrong values); 8/8 URL-Encoded+disambiguation ✓; Session:69d28a4d2697c9202c0bd101; Method:46942, Acct:45945; ACTIVE ✓ |
| 27 | R1 | F* | F | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | N/A | F | F | F | F | F | Y | Y | Y | Y | FAIL (11/18=61%) | LendingTree, Portal/24/7, Exclusive, Order ON, 2 enum; RB-G returned (P1 doubled); RB-AA(4th): P2 skipped; RB-N: states skipped; RB-O: phantom CA(5)+AZ(3)+TX(44); RB-X: empty response after Add criteria (recovered via typed input); SelfCreditRating ChoiceSet shown (string values not UIDs); criterion not saved (RB-V); LoanRequestPurpose stuck in exact-value loop, skipped; P3B correctly skipped for Portal ✓; P5-EXCL/ORDER correct cards ✓; Session:69d28e392697c9202c0bd157; Method:46943, Acct:45946; ACTIVE ✓ |
| 28 | R1 | F* | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | N/A | F | Y | Y | Y | Y | FAIL (18/22=82%) | LendingTree, Webhook/JSON-nested Tue-Thu 9-6 EST, Shared, Order ON, 1 enum; RB-G: P1 doubled (6th); RB-AA(5th): P2 skipped; RB-N NOT triggered (states asked FL/NY/WA ✓); account created BEFORE criteria gate (Steps 11→7-9 inverted, new sub-pattern); RB-V: SelfCreditRating not saved (no ChoiceSet, no update); "continue" re-triggered Step 7 field list; 8/8 JSON-nested mapping ✓; invisible textarea bug delayed P5 input; Session:69d2907a2697c9202c0bd1a0; Method:46944, Acct:45947; ACTIVE ✓ |
| 29 | R1 | F* | F | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | F | N/A | F | F | N/A | F | F | Y | Y | Y | Y | FAIL (10/16=63%) | LendingTree, Email/Mon-Fri 8-5 CST, Exclusive, Order OFF, 2 numeric; RB-G(7th): P1 doubled; RB-AA(6th): P2 skipped→PST/-8 (inline DEBUG ✓); RB-K: acct name before Phase 5 doubled (inline DEBUG ✓); RB-N(11th): states skipped→account created before states; TX/GA/OH provided post-creation; criteria rejected post-creation; P6: Price=32, Exclusive, Criteria=None; Session:TBD; clientUID:29360; ACTIVE ✓ |
| 30 | R1 | F* | F | Y | N/A | N/A | N/A | N/A | F | Y | F | Y | Y | F | Y | Y | F | F | F | F | Y | Y | Y | Y | FAIL (11/19=58%) | LendingTree, FTP/24/7, Shared, Order ON, 1 enum; RB-G(8th): P1 doubled; RB-AA(7th): P2 skipped (inline DEBUG ✓); RB-K: schedule re-prompt (inline DEBUG ✓) + price doubled post-DEBUG (inline DEBUG ✓); P3B: FTP test ran but no Retry/Skip card (RB-DD new, inline DEBUG ✓); RB-N(12th): states skipped→provided post-jump (inline DEBUG ✓); RB-V: SelfCreditRating in P6 not in API payload; 5 inline DEBUGs captured; Session:69d298452697c9202c0bd257; clientUID:29361, Method:46946, Acct:45950; ACTIVE ✓ |
| 01 | R2 | F* | Y | F | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | F | F | F | F | F | Y | Y | Y | Y | FAIL (13/23=57%) | RB-G: P1+P5 prompt doubled; schedule skipped (recovered via DEBUG); P5: exclusivity+order skipped (hallucinated Exclusive+Yes); criteria gate+all criteria skipped; states correctly normalized CA,TX,FL |
| 02 | R2 | Y | Y | F | Y | Y | Y | 11/18 | Y | Y | Y | Y | Y | F | N/A | Y | Y | Y | F | F | N/A | Y | Y | Y | FAIL (15/21=71%) | RB-G: P3-SCHED doubled; RB-N: states step skipped entirely; RB-A: criteria loop exited after 1 criterion on "Add another criterion" click; 11/18 mapped (7 unmapped silently dropped); camelCase field name lookup bug (RB-NEW-R2-1) |
| 03 | R2 | F* | Y | Y | Y | N/A | Y | Y | Y | Y | F | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | FAIL (19/22=86%) | RB-G: P1 prompt doubled; RB-NEW-7: P5-PRICE+P5-STATE both skipped (Order→CriteriaGate, bypassing Steps 1+4); recovered via DEBUG+typed "Add criteria" (price=$25, states CA/TX/FL); XML→JSON switch NOT tested (went straight through XML); 4 criteria in payload after recovery (states 5|44|10 ✓, SelfCreditRating=343593 ✓, LoanRequestType=343609 ✓, LoanAmount>=50000/GreaterOrEqual ✓); create_client failed 2x on duplicate name→used StabilityTest-03r2; clientUID:29378, Method:46955, Acct:45956 | |
| 04 | R2 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | FAIL (20/23=87%) | RB-G: P1 prompt doubled; Phase 3 clean (JSON→XML switch ✓, XML re-parsed, mapping ✓); FIX 14 PASS (conn test failure text in card ✓); FIX 15 FAIL (no schema retention prompt on JSON→XML switch — fix not applied to rw-phase-3-webhook.md); Phase 5c silent skip (RB-NEW-R2-3): clicked Add criteria → Phase 5c called summarize_history prematurely with "Phase 5c Complete" header → agent re-read it as completion → all criteria interaction bypassed; account created with state-only criteria; Session:69d35cb4fe97b9130c36b781 |
| 05 | R2 | F* | Y | Y | Y | Y | Y | 11/11 | Y | Y | Y | Y | Y | Y | Y | F | Y | F | F | Y | Y | Y | F | F | FAIL (16/23=70%) | RB-G: P1 intro + P5 price prompt doubled; FIX 14 PASS (conn test failure in card ✓); 11/11 URL Encoded mapping ✓; RB-NEW-R2-4: states question combined with criteria gate in single card (FIX 3 violation); Phase 5 completed before user typed states; states CA/TX/FL/NY collected via text after Phase 5 done; criteria builder silently skipped (RB-NEW-R2-3 recurrence); P7-SUMM F: client summary card render failed (plain text fallback); P8-ACT F: agent lost all tool access after extended DEBUG session; Session:69d36142fe97b9130c36b7e1; Method:46958, Acct:45959, clientUID:29381 |
| 06 | R2 | F* | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | F | F | Y | Y | Y | Y | Y | FAIL (18/23=78%) | RB-G: P1 doubled + P5 states doubled; P3-SCHED resequenced (appeared AFTER conn test, not before delivery type); RB-A: criteria loop exit after 1 criterion (no loop prompt shown); P5-FIELD: no field suggestions card (Phase 5c State 1 skipped); fuzzy "credit rating excellent" re-prompted (not parsed), explicit "SelfCreditRating equals EXCELLENT" triggered enum dropdown ✓; SelfCreditRating In 343593 in account ✓; FIX 14 PASS; XML auto-detect ✓; 9/9 mapping ✓; states CA/TX/NY/FL normalized ✓; ACTIVE ✓ |
| 07 | R2 | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | F | Y | Y | Y | Y | FAIL (18/23=78%) | RB-G: P1+P5-PRICE doubled; JSON auto-detect ✓; 12/12 mapping with 3-field disambiguation (state→ContactState, zip→ContactZip, loan_type→LoanRequestType) ✓; FIX 14 PASS; P5 cards all shown correctly (Excl/Order/States); RB-NEW-R2-3: Phase 5c silently skipped after "Add criteria" click — account created with Additional Criteria: None; states FL/CA/TX/NY/IL normalized ✓; ACTIVE ✓ |
| 08 | R2 | F* | Y | F | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | Y | F | F | F | F | F | Y | Y | Y | Y | FAIL (13/20=65%) | RB-G: P1+P3-SCHED+P5-PRICE all doubled; FTP creds not doubled ✓; FIX 14 PASS; P4-SUMM ✓; RB-N: states skipped (Order→criteria gate directly); states "TX,FL,CA" sent as text → agent treated as criteria gate response, created account immediately; criteria gate+builder completely bypassed; Target States TX/FL/CA recovered ✓; Additional Criteria: None; ACTIVE ✓ |
| 09 | R2 | F* | Y | F | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | F | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | FAIL (11/14=79%) | RB-G: P1+P3-SCHED+P5-PRICE all doubled; Portal skip criteria; states merged with criteria gate (RB-NEW-R2-4); states loop ×4 — agent kept re-asking despite valid "NY, OH" input; recovered after DEBUG; schedule card buttons not rendered (text fallback); NY/OH normalized ✓; Skip ✓; ACTIVE ✓ |
| 10 | R2 | F* | Y | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | F | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | FAIL (11/14=79%) | RB-G: P1 doubled; Email + no criteria; RB-N: states skipped (Order→criteria builder error directly); criteria builder load failed → Skip accepted → Additional Criteria: None (correct for scenario); Target States blank (no states collected); no criteria gate shown; ACTIVE ✓ |

\* P1-PROMPT F* = cosmetic (prompt text doubled, not a functional failure). Not counted as a failure in the percentage calculation.

---

## Per-Phase Failure Rates (R1 vs R2)

R1 = runs 01–10, gpt-5.4-mini all phases (mixed during runs 11–20). R2 = runs 01–05 so far, gpt-5.4-mini all phases.  
Denominator excludes N/A and ? entries. 🔴 = ≥50% · 🟡 = 20–49% · ✅ = <20%

| Checkpoint | R1 fail (n=10) | R2 fail (n=5) | Trend |
|------------|----------------|----------------|-------|
| P1-PROMPT | 6/10 = 60% 🔴 | 4/5 = 80% 🔴 | ↓ |
| P2-DROP | 0/10 = 0% ✅ | 0/5 = 0% ✅ | ↔ |
| P3-SCHED | 2/10 = 20% 🟡 | 2/5 = 40% 🟡 | ↓ |
| P3-WURL | 0/10 = 0% ✅ | 0/5 = 0% ✅ | ↔ |
| P3-JSON | 1/7 = 14% ✅ | 0/4 = 0% ✅ | ↑ |
| P3-TABLE | 1/7 = 14% ✅ | 0/5 = 0% ✅ | ↑ |
| P3-COUNT | 0/6 = 0% ✅ | 0/5 = 0% ✅ | ↔ |
| P3B-TEST | 0/10 = 0% ✅ | 0/5 = 0% ✅ | ↔ |
| P4-SUMM | 1/10 = 10% ✅ | 0/5 = 0% ✅ | ↑ |
| P5-PRICE | 0/10 = 0% ✅ | 1/5 = 20% 🟡 | ↓ |
| P5-EXCL | 2/10 = 20% 🟡 | 1/5 = 20% 🟡 | ↔ |
| P5-ORDER | 0/10 = 0% ✅ | 1/5 = 20% 🟡 | ↓ |
| P5-STATE | 2/9 = 22% 🟡 | 2/5 = 40% 🟡 | ↓ |
| P5-NORM | 3/8 = 38% 🟡 | 0/4 = 0% ✅ | ↑ |
| P5-FIELD | 5/10 = 50% 🔴 | 2/5 = 40% 🟡 | ↑ |
| P5-CR1 | 2/10 = 20% 🟡 | 1/5 = 20% 🟡 | ↔ |
| P5-ENUM | 1/2 = 50% 🔴 | 3/5 = 60% 🔴 | ↓ |
| P5-CR3 | 9/10 = 90% 🔴 | 4/5 = 80% 🔴 | ↑ |
| P5-DONE | 9/10 = 90% 🔴 | 2/5 = 40% 🟡 | ↑↑ |
| P6-BOOL | 0/10 = 0% ✅ | 0/4 = 0% ✅ | ↔ |
| P6-SUMM | 2/10 = 20% 🟡 | 0/5 = 0% ✅ | ↑ |
| P7-SUMM | 0/9 = 0% ✅ | 1/5 = 20% 🟡 | ↓ |
| P8-ACT | 0/10 = 0% ✅ | 1/5 = 20% 🟡 | ↓ |

**Key observations:** P1-PROMPT (RB-G prompt doubling) is the most persistent failure — 60→80% in R2, fix had no effect. P5-DONE improved dramatically (90%→40%), indicating criteria are now persisting more reliably. P5-CR3 and P5-ENUM remain critical. P3-P4 phases largely stable. P7/P8 showing new failures in R2 (tool access loss after extended DEBUG sessions).

---

## Cross-Round Comparison

### R2 vs R1 Comparison

| Finding | R1 Rate (01-10) | R2 Rate (01-10) | Status |
|---------|-----------------|-----------------|--------|
| RB-G (prompt duplication) | ~80% | 9/10 = 90% | ↔ SAME — FIX 1 had no effect |
| RB-N (states skipped) | 40% | 4/10 = 40% (runs 01,08,09,10) | ↔ SAME |
| RB-O (phantom states) | 40% | 0/10 = 0% | ↑ FIXED — no phantom states in R2 |
| RB-A (criteria loop exit) | 67% | 2/3 = 67% (runs 06,07 of criteria runs) | ↔ SAME |
| RB-F (Phase 5c load fail) | 43% | 1/10 = 10% (run 10 criteria builder error) | ↑ IMPROVED |
| RB-NEW-R2-3 (P5c silent skip) | N/A | 4/5 = 80% (runs 04,05,07,08) | NEW — dominant criteria failure mode |
| RB-C (states not normalized) | 60% | 0/6 = 0% | ↑ FIXED — all states USPS ✓ |
| RB-M (>= as Equal) | 20% | not tested (no numeric criteria persisted) | N/A |
| Overall pass rate | 1/10 = 10% | 0/10 = 0% | ↓ WORSE (but all due to P1 doubling) |
| Average score | ~58% est | 75% | ↑ IMPROVED |

### Phase Stability Comparison: GPT-5.4 (runs 01–20) vs GPT-5-mini Webhook/Criteria (runs 21–30)

*(Completed — runs 21–30 done)*

| Phase | GPT-5.4 Fail% (01–20) | GPT-5-mini Fail% (21–30) | Delta | Notes |
|-------|----------------------|--------------------------|-------|-------|
| P1-PROMPT (RB-G doubling) | ~95% | 40% (B27,B28,B29,B30 of 10) | **↓ improved** | B21–B26 clean; returned B27–B30 |
| P2-DROP (RB-AA) | 0% (01–20) | 70% (B24–B30, all 7 of last 7) | **↑ new regression** | New finding; 0 in 01–20, systematic from B24 |
| P3-SCHED | ~10% | ~10% | = | Similar rate |
| P3B-TEST (conn test card) | ~5% | 10% (B30 FTP) | slight ↑ | RB-DD new in B30 |
| P5-STATE (RB-N) | ~40% (8/20) | 60% (6/10: B22,B23,B25,B27,B29,B30) | **↑ worse** | Criteria phases use GPT-5-mini; RB-N rate higher |
| P5-CR3/DONE (criteria lost) | ~60% | ~70% (B21,B22,B23,B25,B27,B28,B30) | ↑ | RB-V dominant pattern in 21–30 |
| RB-G (doubling) | ~95% | 40% (4/10) | **↓ improved** | Significant reduction; non-systematic in 21–30 |
| RB-X (empty response loop) | 1/12 webhook+LT | 1/4 LT+criteria runs | ↑ risk | B27: empty response on Add criteria; LT+criteria high-risk combo |
| P8-ACT (activation) | ~85% | 100% (10/10) | **↑ improved** | All 10 runs activated successfully despite failures |

---

## Findings Catalog

### Root Cause Classifications

- **IGNORE** — The instruction is explicitly present in the instruction pack (verified), but the AI silently skipped it. The instruction uses language like "Do NOT skip", "MANDATORY", "STOP AND YIELD" — the AI simply did not follow it.
- **HALLUCINATE** — The AI generated content or behavior not present in the instructions (e.g., asking for FTP credentials on a webhook method, fabricating phantom state values).
- **AMBIGUOUS** — The instruction is unclear, contradictory, or has a gap that led to the failure. Potentially fixable via instruction changes — requires analysis.
- **PLATFORM** — Non-deterministic card rendering, MCP resource loading failures, context overflow, tool timeouts — not an instruction or AI behavior issue.

### Finding RB-A — Criteria loop exits after first criterion (P5-CR3 / P5-DONE FAIL) [R1] [R2]

#### R1

**Phase:** 5 | **Severity:** High | **Frequency:** 6/9 (B01, B04, B05, B06, B07, B08 — B02/B10 skipped criteria via RB-D; B03 passed; B09 conversational mode)  
**Observed:** After entering criteria, agent either exits loop immediately after 1 criterion (B01, B04) or loop control keywords ("add another criterion") are treated as field name lookups rather than loop control signals (B05). In B05: "add another criterion" → "I couldn't find that field." Loop card renders with only "Continue" button (missing "Add another" and "See more fields" buttons).  
**Expected:** Agent stays in criteria loop after each accepted criterion and prompts "Would you like to add another criterion, see more fields, or continue?" Loop must recognize control keywords ("add another", "continue", "done", "skip") and not treat them as field names.

**B05 additional detail:** Loop card in B05 rendered with only "Continue" button (RB-H co-occurring). Loop keyword recognition completely broken — "add another criterion" parsed as a field lookup rather than flow control.

**DEBUG Response Summary:** *(B04) "The criteria flow should not have exited after one criterion. The additional-criteria gate was treated as resolved instead of continuing into the criteria-builder loop."*  
**Agent's specific patch (B04):** "Keep the phase in the additional-criteria branch until the user explicitly chooses Skip. Do not treat a single added criterion as completion of the entire criteria path unless the builder itself explicitly signals done."  
**Suggested fix:** Phase 5c State 2 must check for loop control keywords BEFORE attempting field lookup. Add explicit keyword matching: if input matches "add another|continue|done|skip|see more fields" → route to corresponding loop action. Only proceed to field lookup if none of these match.

#### R2

**RB-A (persists): Criteria loop exits after 1 criterion (Run 02)**
Clicking "Add another criterion" button immediately triggered delivery account creation instead of looping back for another criterion entry. Only 1 criterion (SelfCreditRating = GOOD) was saved. FIX 9 (inline exit handling with mandatory criteria loop prompt) did not prevent this. The button click was routed to account creation path.

---

### Finding RB-B — Summary card omits entity IDs (informational) [R1]

**Phase:** 7 | **Severity:** Low | **Frequency:** 1/3 (Run B01)  
**Observed:** Client Setup Summary card does not show IDs for Delivery Method or Account (unlike Stabilized which shows e.g. "ID: 46888").  
**Expected:** IDs may or may not be required — this is a design choice. Stabilized variant includes IDs in summary; Rework omits them. Confirmed via DEBUG that actual unique IDs were created (46890 / 45895).  
**Note:** Not necessarily a bug — the Rework design may intentionally omit IDs for cleanliness.

---

### Finding RB-C — States not normalized after summarization (P5-NORM FAIL) [R1] [R2]

#### R1

**Phase:** 5 | **Severity:** High | **Frequency:** 3/5 (B02, B04, B05 — B01 and B03 normalized correctly)  
**Observed:** Target States shown as full names in Phase 6 summary — not normalized to USPS codes. B02: "California, Texas, Florida". B04: "California, Texas, Florida, New York, Illinois".  
**Expected:** Agent normalizes full state names → USPS codes before calling `create_delivery_account`.  
**Root cause (suspected):** B02 — summarization earlier due to longer conversation, Phase 5 normalization context lost. B04 — conversation also had retries and extra DEBUG messages; same root cause likely applies.  
**Agent DEBUG response (B04):** "Target states also should be normalized to USPS codes before criteria creation. In the current phase instructions, that normalization step is intended, but it was not reflected in the retained summary."  
**Agent's specific patch (B04):** "Normalize all user-entered target states to USPS abbreviations before matching. Accept full names and abbreviations, normalize at point of input, not at summarization."  
**Suggested fix:** Phase 5 resource must normalize states unconditionally at input time, not rely on prior context or summarization. Normalization must happen inline before `create_delivery_account` call.

#### R2

**RB-C (FIXED):** R2 Rate 0/6 = 0% — all states USPS ✓. FIX 17 (targetStates USPS in summarization) resolved this.

---

### Finding RB-D — Criteria prompt entirely skipped after summarization (P5-FIELD/CR1/CR3/DONE FAIL) [R1] [R2]

#### R1

**Phase:** 5 | **Severity:** High | **Frequency:** 2/10 (B02, B10 — different root causes)  
**Observed:** After receiving target states, agent immediately called `create_delivery_account` and showed the Phase 6 summary — no "Would you like to add additional lead criteria?" prompt was shown at all. In Run B01, Phase 5c was entered (though it exited prematurely after 1 criterion).  
**Expected:** Per Scenario 5.8: after creating account, agent must ask "Would you like to add additional lead criteria, or skip?" before proceeding to Phase 6.  
**Root cause (B02):** Summarization trigger — Phase 5 context lost; agent skipped Phase 5c gate entirely.  
**Root cause (B10):** Phase 5 routing made criteria optional in practice. Agent took the "skip criteria" branch without asking the user. DEBUG response: *"The account creation branch in your flow allowed a skip path for criteria, and the execution followed that skip path. The phase routing made criteria optional in practice."*  
**DEBUG Response Summary (B10):** *"In Phase 5, add a hard gate before account creation: 'If additional criteria are available or required for this lead type, you must enter Phase 5c: Criteria Builder before creating the delivery account.' Change skip language from optional to conditional. Add ordering constraint: do not call create_delivery_account until criteria collection is complete or explicitly confirmed as none."*  
**Note on B10:** DEBUG query sent mid-flow was ignored — agent had already committed to Phase 6. DEBUG only responded after activation was complete.  
**Suggested fix:** Phase 5 must unconditionally show the criteria gate card ("Would you like to add additional lead criteria, or skip?") BEFORE calling `create_delivery_account`. The gate is not a branch — it is a required collection step. Remove any "skip criteria" default path that bypasses user confirmation.

#### R2

**RB-D (persists): Criteria gate skipped (Run 01)**
Agent never showed "Would you like to add additional lead criteria, or skip?" card. Went directly from states to account creation. All criteria steps (field suggestions, criterion parsing, loop) never executed.

---

### Finding RB-E — Agent asks for delivery account name before price (informational) [R1]

**Phase:** 5 | **Severity:** Low | **Frequency:** 1/3 (Run B02, not observed in B01)  
**Observed:** When transitioning to Phase 5, agent first asked "Please provide the delivery account name." before asking for price. UX scenarios (Scenario 5.1) start directly with price — account name collection is not specified.  
**Expected:** Phase 5 starts with price prompt per Scenario 5.1.  
**Note:** May be an undocumented step in the rework Phase 5 instructions. Account name was correctly used in tool call and summary.

---

### Finding RB-F — Phase 5c fails to load on first "Add criteria" click (P5-CR1 delayed) [R1] [R2]

#### R1

**Phase:** 5c | **Severity:** High | **Frequency:** 3/7 (B03, B06, B07 — B07 most severe: 2 card clicks produced duplicate gate cards, 3rd attempt plain text fallback, DEBUG+explicit URL required)  
**Observed (B03):** After clicking "Add criteria", agent responded with "Please continue with the additional criteria setup." — no field suggestions shown. A second attempt worked.  
**Observed (B06 — more severe):** After "Add criteria" card click → "I'm ready for the next step." (generic response). Nudge "add criteria" → "What additional lead criteria would you like to add?" ×2 (intra-message duplicate). Direct criterion "LoanAmount >= 50000" → ignored, same question repeated. Required DEBUG with explicit resource URL to force Phase 5c load. All subsequent criteria then worked correctly.  
**Expected:** Per Scenario 5c.1: agent immediately shows recommended fields with "Show more fields" / "Skip" card after "Add criteria" is clicked. No additional nudge required.  
**DEBUG Response Summary:** "I did not have the actual Phase 5c instructions loaded in the conversation context at that point, so I couldn't safely process it as the active criteria step." Suggested patch: "When the user selects 'Add criteria', immediately load and execute Phase 5c at mcp://resource/rw-phase-5c-criteria-builder. Do not re-prompt for the same choice."  
**Root cause:** Phase 5 instructions likely call `get_resource` for Phase 5c conditionally — the resource load may be deferred or racing with user input. The `display_adaptive_card` for the additional criteria choice fires before Phase 5c is in context.  
**Suggested fix:** Phase 5 must load Phase 5c synchronously (blocking) before showing any criteria-related prompt. Ensure `get_resource(rw-phase-5c-criteria-builder)` completes before the agent awaits user input.

#### R2

**RB-F (improved):** R2 Rate 1/10 = 10% (run 10 criteria builder error). Down from 43% in R1.

---

### Finding RB-G — Phase transition prompts duplicated in same response (double-render) [R1] [R2]

#### R1

**Phase:** 1, 2, 3, 4, 5 | **Severity:** High | **Frequency:** B04, B05, B06, B07, B08, B09 — **systematic: every phase transition in every run**  
**Observed:** After phase transitions, the first prompt of the new phase is rendered twice in the same agent message. In B04: "Please describe your preferred delivery schedule" appeared twice after selecting "Specific hours only"; "Finally, let's set up your Delivery Account. Please provide the price per lead." appeared twice after Phase 4 Continue. In DEBUG23 session: Phase 1 collection prompt appeared twice.  
**Expected:** Prompt rendered exactly once, then agent yields.

**DEBUG Response Summary:**  
> "I treated the prompt as something to 'confirm' or restate rather than a terminal user-facing message. I failed to enforce the pause after the prompt. The second copy appears to have come from response handling rather than a new workflow step."

**Agent self-diagnosis:** Failed to respect `STOP AND YIELD` — treated exact phase prompt as a confirmation target rather than terminal output.

**Agent's suggested patch (general):**  
> "After emitting any exact phase prompt, set a hard 'sent_and_waiting' state. Block any re-render or echo of the same prompt until the user replies."

**Agent's specific instructions patch:**  
> In State 1, immediately after "STOP AND YIELD. Do not hallucinate data. Do not proceed to State 2..." add:  
> `"Send the prompt exactly once. Do not repeat, echo, confirm, paraphrase, or re-emit the prompt in the same turn."`  
> Also add under Prompts and Communication:  
> `"When a phase specifies an exact prompt, that prompt is the complete assistant response for that turn. No additional text, acknowledgements, or follow-up lines may be appended."`

**Agent's minimal/system-level patch:**  
> "A duplicate-output guard that compares the outgoing message to the last assistant message and suppresses identical resend within the same phase."

#### R2

**RB-G (persists): Prompt duplication (Run 01)**
P1 intro prompt doubled in same message. P5 price prompt doubled in same message. FIX 1 (output suppression + "never output same prompt text more than once") did NOT fix this. Rate in R1 was ~80%. Still present in R2 run 01.

---

### Finding RB-H — Content type choice card rendered as plain text (P3-TABLE card non-determinism) [R1]

**Phase:** 3-Webhook | **Severity:** Medium | **Frequency:** observed in DEBUG23 session  
**Observed:** After user selected "I'll provide instructions", the content type question ("What content type should this delivery use?") was sent as plain numbered text list instead of an adaptive card with 4 buttons.  
**Expected:** `display_adaptive_card` called with URL Encoded / JSON / XML / I'm not sure buttons (as seen in subsequent corrected renders, e.g. C15).

**DEBUG Response Summary:**  
> "I asked the question as plain text instead of calling the display_adaptive_card tool, which violates the card rule for enumerations. That's a mistake in execution, not expected behavior."

**Root cause:** Card non-determinism — the model intermittently renders card-required steps as plain text. Likely occurs when prior context is heavy or when multiple steps execute in the same turn.  
**Suggested fix:** The system prompt rule "when presenting enumerated choices, ALWAYS use display_adaptive_card" must be reinforced. Consider adding a phase-level reminder at the top of each phase resource: "All multi-choice steps MUST use display_adaptive_card — never plain text enumeration."

---

### Finding RB-I — XML parse failure does not re-detect payload format [R1]

**Phase:** 3-Webhook | **Severity:** Medium | **Frequency:** observed in DEBUG23 session  
**Observed:** User selected XML content type, then pasted a clearly-valid JSON body. Agent detected XML parse failure and showed "Re-paste XML schema / Switch content type" card — without detecting that the payload was JSON and suggesting it.  
**Expected:** After parse failure, agent checks payload against other format signatures. If confidence >90%, recovery card pre-selects the detected format: "This looks like JSON — switch to it?"

**DEBUG Response Summary (deep analysis):**  
> "auto-detection only runs if the user selected 'I'm not sure'. You explicitly selected XML, so the workflow forced the system into strict XML parsing mode — there is no branch that says 'this clearly looks like JSON, suggest JSON'. State 3 Step 1, when parsing fails, the system jumps directly to the recovery prompt. It does not check: Does the payload match another format signature? Is there a high confidence alternative?"

**Agent's specific instructions patch:**  
> In Phase 3 Webhook State 3 Step 1 (XML parse failure recovery), after confirming parse failure, add:  
> `"Before showing the recovery card, attempt to detect the payload format: try JSON.parse(); if valid JSON → add a third option to the recovery card: 'Switch to JSON (detected)'. If URL-encoded signature detected → add 'Switch to URL Encoded (detected)'. Pre-label the detected option clearly."`

---

### Finding RB-J — postingInstructions cleared on content type switch (UX regression) [R1]

**Phase:** 3-Webhook | **Severity:** Low | **Frequency:** observed in DEBUG23 session  
**Observed:** After switching content type from XML → JSON, agent re-asked "Please paste your posting instructions" — even though the user had already provided the full JSON body.  
**Expected:** Agent preserves postingInstructions across the switch and asks "Would you like to use the same posting instructions, or provide new ones?"

**DEBUG Response Summary:**  
> "In Phase 3 Webhook State 3 Step 1, the instruction says: 'If user selects Switch content type: clear contentTypeChoice, clear postingInstructions, go back to State 2 Step 2'. That means the system intentionally throws away the previously pasted schema. The design assumption: the schema might change when the format changes. But this is the wrong default — the user already has a schema and just wants a different parser."

**Agent's specific instructions patch:**  
> In State 3 Step 1 (switch content type branch), replace `"clear postingInstructions"` with:  
> `"Retain postingInstructions in memory. When returning to State 2 Step 3, prompt: 'Would you like to use the same posting instructions you provided, or paste new ones?' — if user confirms reuse, skip re-collection and proceed to State 2 Step 4."`

---

### Finding RB-K — Phase 5 not loaded before responding (generic account name prompt before price) [R1]

**Phase:** 4→5 transition | **Severity:** High | **Frequency:** confirmed in DEBUG23 session (also matches RB-E pattern)  
**Observed:** After Phase 4 summary card's "Continue" was clicked, agent asked "Please provide the delivery account name." — BEFORE loading Phase 5 instructions. Phase 5 Step 1 says to ask for price first; account name is not in Phase 5 at all.  
**Expected:** Agent loads `mcp://resource/rw-phase-5-create-delivery-account` FIRST, then asks for price per the first step.

**DEBUG Response Summary:**  
> "I asked for the account name because I had not yet loaded the active Phase 5 instructions at that point, so I used a generic setup step instead of the phase-specific first prompt."

**Follow-up DEBUG (what prevents this):**  
> "The main thing that prevents this is a stricter phase-handling rule: Before asking any setup question, always load the current phase resource first. Do not send a user-facing prompt until the active phase instructions are confirmed. Never use a generic setup prompt when a phase-specific prompt exists."

**Agent's specific instructions patch:**  
> Add to system prompt (Tier 1) or Phase 4 exit:  
> `"Before sending any user-facing prompt during a phase transition, confirm the next phase resource has been loaded via get_resource. Never generate a generic collection prompt while awaiting phase instructions. If phase instructions are not yet loaded, call get_resource(next_phase_url) first — then execute its first incomplete step."`

**Root cause:** The Phase 4 summary `NEXT_PHASE` card fires `get_resource` and `display_adaptive_card` in the same turn. The model can generate a user-visible message BEFORE the `get_resource` response returns, filling the gap with a generic prompt.

---

### Finding RB-L — Content type question skipped when delivery type answered twice (RB-G cascade) [R1]

**Phase:** 3-Webhook | **Severity:** High | **Frequency:** 1/5 (B05)  
**Observed:** In B05, the content type question (JSON / XML / URL Encoded / I'm not sure) was never asked. After the user provided the Webhook URL, the agent jumped directly to "Would you like to configure field mappings?" — skipping content type entirely.  
**Expected:** Phase 3 Webhook must ask content type after URL collection.  
**Root cause:** RB-G caused the Phase 3 delivery type prompt to appear twice — once as a card (user clicked Webhook) and once as a text duplicate (user typed "Webhook"). The agent received TWO "Webhook" answers in succession. The second forced-text response ("Webhook") arrived while Phase 3 Webhook sub-phase was already executing, causing the content type step to be bypassed.  
**RB-G cascade:** This is a direct downstream failure from RB-G — the duplicate text response disrupts Phase 3 Webhook state machine by injecting a redundant input mid-execution.  
**Suggested fix:** Fixing RB-G (prompt duplication) will likely fix this cascade. Additionally: Phase 3 Webhook sub-phase should be idempotent to duplicate "Webhook" inputs — detecting that content type has not yet been set and re-entering at that step.

---

### Finding RB-M — Numeric criteria operator `>=` stored as "Equal" instead of "GreaterOrEqual" [R1]

**Phase:** 5c | **Severity:** High | **Frequency:** 1/5 (B05)  
**Observed:** User entered "Credit >= 700". Agent accepted and confirmed the criterion. Delivery account summary showed "Credit Equal 700" — operator "GreaterOrEqual" was not set; "Equal" was stored instead.  
**Expected:** `>=` should map to operator `GreaterOrEqual`. Similarly: `<=` → `LessOrEqual`, `>` → `GreaterThan`, `<` → `LessThan`, `between X and Y` → `Between`.  
**Root cause (suspected):** Phase 5c criteria parser maps the user's natural language expression to an operator enum. The `>=` symbol was likely not in the explicit mapping table, causing fallback to default "Equal".  
**Suggested fix:** Phase 5c criteria builder must include explicit symbol → operator mapping: `>= → GreaterOrEqual`, `<= → LessOrEqual`, `> → GreaterThan`, `< → LessThan`, `= or == or "equals" → Equal`. Symbol mapping must be checked before natural language parsing.

---

### Finding RB-N — States question skipped entirely in Phase 5 [R1] [R2]

#### R1

**Phase:** 5 | **Severity:** High | **Frequency:** 12/30 (B07, B11, B14, B15, B16, B19, B22, B23, B25, B27, B29, B30)  
**Observed:** After Order System = No, the states/geography question was not asked. Flow jumped from order decision → criteria gate, with no "Which states should this account receive leads from?" prompt at any point.  
**Expected:** Phase 5 must collect target states before the criteria gate. States are a required field for delivery account configuration.  
**Root cause:** Unknown — possibly a Phase 5 state machine branch that skips states when order=No, or a summarization side-effect corrupting the step sequence. DEBUG after activation could not pinpoint the exact branch since context was partially summarized.  
**Suggested fix:** Add explicit unconditional states collection step in Phase 5 between order decision and criteria gate. Step must not be conditional on any prior state.

#### R2

**RB-N (FIXED in R1 Run 01, persists in R2 Run 02):**
FIX 3 (merged Steps 4-6) worked — states question appeared correctly in R1 Run 01. States normalized CA, TX, FL. This was the #1 target of the rework fixes (40% failure in R1). However, the steps BEFORE states (exclusivity, order) got skipped instead — the merging may have pulled attention to the states prompt at the expense of earlier steps.

In R2 Run 02, states step was skipped entirely again — agent went from Order YES directly to criteria field suggestions, bypassing states collection. Target states field in P6-SUMM was empty. State criterion was not built. Criteria builder loaded without state context.

R2 overall rate: 4/10 = 40% (runs 01,08,09,10) — same as R1.

---

### Finding RB-O — Phantom states appear in account summary without user input [R1] [R2]

#### R1

**Phase:** 5/6 | **Severity:** Critical | **Frequency:** 4/15 (B07, B11, B14, B15 — co-occurring with RB-N)  
**Observed:** Delivery account summary showed "Target States: CA, AZ, TX" — values the user never entered. The states question was skipped (RB-N), yet the account was created with CA, AZ, TX.  
**Expected:** If states were not collected, the account should have no geographic filter (all states) or an explicit empty value — not phantom data.  
**Agent's DEBUG analysis:** "The criteria builder path accepted a prepopulated target-states set from the underlying workflow context and carried it into the account summary, even though you did not explicitly answer a states prompt in this turn. The data collection rule that failed: 'For any ASK field, do not generate, infer, reuse, or apply defaults—use only values the user explicitly provides for that.'"  
**Agent's patch:** "Make target states a strict ASK field in Phase 5c. If missing, the workflow must prompt the user directly and not allow any inherited/default state list to flow into the summary. Add validation that Phase 6 cannot start unless target states were explicitly captured."  
**Suggested fix:** Phase 5 must validate that states value was captured in the current conversation turn before calling `create_delivery_account`. Prior session state or default values must never be used.  
**Session log verification (69d1503f2697c9202c0bc0c6):** CONFIRMED — `create_delivery_account` actual payload: `criteria:[{"leadFieldUID":144881,"operator":"In","value":"5|3|44"}]` where UID 5=CA, UID 3=AZ, UID 44=TX. These phantom states were submitted to the real API. Not a display hallucination.

#### R2

**RB-O (FIXED):** R2 Rate 0/10 = 0% — no phantom states in R2. FIX 4 (phantom states guard) resolved this.

---

### Finding RB-P — create_delivery_account silently skipped; update_delivery_account fails on non-existent account [R1]

**Phase:** 5c / P5→P5c transition | **Severity:** Critical | **Frequency:** 1/8 (B08 — FTP with connection test; not observed in Webhook runs)  
**Observed:** After the FTP connection test, `create_delivery_account` was **never called**. The agent went directly from `test_ftp_sftp_connection` → `update_delivery_account` × 4. All 4 update calls failed with Internal Server Error because no account existed to update. Agent continued to "activation" and reported success despite no account being created.  
**Root cause (confirmed via session log `69d15a862697c9202c0bc120`):** The FTP connection test phase (P3B) concluded and the agent transitioned to Phase 5 — but the `create_delivery_account` call was entirely absent from the tool sequence. The agent proceeded as if account creation had already happened and jumped straight to updates. The `deliveryAccountUID: 46897` in all update payloads was actually the delivery **method** UID (46897 = the FTP delivery method), not an account UID. No account was ever created; activation reported success with no account in DB.  
**Agent-reported payload (DEBUG):** `deliveryAccountUID: 46897` = `deliveryMethodUID: 46897` — same UID used for both fields, confirming confusion between method and account.  
**Expected:** Phase 5 must call `create_delivery_account` before any `update_delivery_account`. Phase 5c updates must use the UID returned from `create_delivery_account`, not the method UID.  
**Suggested fix:**
1. Phase 5 must explicitly call `create_delivery_account` as step 1 before collecting any P5c criteria. Never skip this call.
2. The account UID must be captured from the `create_delivery_account` response and used in all subsequent `update_delivery_account` calls.
3. Phase 3B (connection test) exit must not interfere with Phase 5's account creation step.
4. Add guard: before calling `update_delivery_account`, verify that `deliveryAccountUID` ≠ `deliveryMethodUID` — they must be different values.

---

### Finding RB-Q — "Show more fields" button in P5c triggers field-not-found error [R1]

**Phase:** 5c | **Severity:** Medium | **Frequency:** 1/8 (B08 — only run where "Show more fields" button was needed)  
**Observed:** After failing to find a "State" field, clicking the "Show more fields" button in the P5c card resulted in "I couldn't find that field. Please type the field name..." error — the same error as a failed field name lookup. The button was not working as intended.  
**Expected:** "Show more fields" button should call the field-listing tool and display the next set of available fields as a list.  
**Workaround confirmed:** Typing "show fields" as text worked correctly — agent showed "Additional Fields (showing up to 10): Address, Email".  
**Root cause (suspected):** The button value submitted by "Show more fields" click is being parsed through the field name lookup path instead of the show-fields command path. The `Action.Submit` data payload may be sending field text as literal input rather than a command signal.  
**Related:** This is similar to the Action.Submit `data` field bug (F1/F14 — ChoiceSet + Action.Submit must NOT have a data field). The button's data payload may be incorrectly set.  
**Suggested fix:** "Show more fields" button in P5c must submit a command code that routes to the field-listing handler, not through the field name lookup. Check the `Action.Submit` data object for this button — ensure it signals "show_more_fields" or equivalent, not the button label text.

---

### Finding RB-R — Portal delivery uses deliveryType "HttpPost"; no deliveryAddress field [R1]

**Phase:** 3 (Portal) | **Severity:** Informational (correctness to understand) | **Frequency:** 1/1 Portal runs (B09)  
**Observed:** In `create_delivery_method` for Portal delivery, the actual API payload uses `deliveryType: "HttpPost"` with no `deliveryAddress` field at all.  
**DEBUG response hallucination:** During B09 DEBUG session, agent reported `deliveryAddress: "Portal"` in its self-reported payload. Session log confirmed this was hallucinated — the actual call had no such field.  
**Implication:** Portal delivery does not require a URL. The `deliveryType: "HttpPost"` value appears to be the system's internal representation for Portal. Instructions should not mention `deliveryAddress` for Portal type.  
**Note:** This is likely by design. Documents the API contract for Portal delivery.

---

### Finding RB-S — Numeric P5c criteria saved via update_delivery_account with no UI acknowledgment [R1]

**Phase:** 5c | **Severity:** Medium | **Frequency:** 1/1 conversational-P5c runs (B09)  
**Observed:** In B09, numeric criteria (Credit >= 700, LoanAmount >= 50000, 1stMortgageBalance <= 300000) were processed and saved via `update_delivery_account` calls — but no user-visible confirmation was shown. Agent silently executed the tool call and continued collecting the next criterion without any "✓ Criterion added" message or similar acknowledgment.  
**Expected:** After each criterion is saved, the agent should confirm: "✓ Added: [criterion description]. Would you like to add another, or continue?"  
**Root cause:** Phase 5c in conversational fallback mode (Phase 5c resource not loaded) processes and saves criteria internally without triggering any display_adaptive_card or structured acknowledgment step.  
**Suggested fix:** Phase 5c instructions must include an explicit acknowledgment step after every successful `update_delivery_account`: show a brief confirmation message naming the criterion just saved, then prompt for next action.

---

### Finding RB-T — "continue" after P5c puts agent in confused state instead of proceeding to P6 [R1]

**Phase:** 5c → P6 transition | **Severity:** High | **Frequency:** 1/1 conversational-P5c runs (B09)  
**Observed:** After all criteria were entered, typing "continue" caused the agent to respond: "Please share the exact Phase 6 summary wording you want checked" — as if it expected a review task, not a transition to P6.  
**Expected:** "continue" in P5c should trigger Phase 6 summary generation and display.  
**Workaround:** Typing "Show the delivery account summary now and proceed to activation" worked — agent generated P6 summary correctly.  
**Root cause:** When Phase 5c runs in conversational fallback mode (resource not loaded), the exit signal "continue" is not recognized as the P5c→P6 transition trigger. The agent interprets it ambiguously within a confused state.  
**Suggested fix:**
1. Primary fix: ensure Phase 5c resource loads correctly (fix RB-F) — eliminates conversational fallback mode entirely.
2. Secondary: add explicit "continue" → P6 routing in Phase 5c exit. The word "continue" at criteria exit should never prompt the user to share summary wording.

---

### Finding RB-U — update_delivery_account calls are non-cumulative; earlier criteria may be overwritten [R1]

**Phase:** 5c | **Severity:** Critical (data integrity) | **Frequency:** 1/1 conversational-P5c runs with multiple updates (B09)  
**Observed:** B09 session log shows 4 `update_delivery_account` calls. Each call sends only the new criterion(ia) in the criteria array — not the full accumulated set:
- Update 1: `[Credit >= 700]`
- Update 2: `[LoanAmount >= 50000]`
- Update 3: `[LoanAmount >= 50000]`
- Update 4: `[LoanAmount >= 50000, 1stMortgageBalance <= 300000]`

States were submitted in the initial `create_delivery_account` call.  
**Concern:** If `update_delivery_account` uses **replace semantics** (overwrites the entire criteria array), then after update 4, the DB state would be `[LoanAmount, 1stMortgageBalance]` only — the states criterion (from create) and Credit (from update 1) would be silently overwritten.  
**The P6 summary showed all 4 criteria** — but this may reflect the agent's in-memory state, not the actual DB state. Cannot confirm from session log alone whether the API is additive or replace.  
**Suggested fix:**
1. Investigate `update_delivery_account` API semantics: additive (append) or replace?
2. If replace: Phase 5c must accumulate all criteria in memory and submit the full list on every update call.
3. If additive: current behavior is correct but states from create may still conflict (duplicate or merge behavior unknown).
4. Regardless: add integration test to verify that after multiple updates, all criteria are present in the final account state.

---

### Finding RB-V — Criteria accepted/shown in summary but never sent to API [R1] [R2]

#### R1

**Phase:** 5c | **Severity:** Critical | **Frequency:** 4/30 (B11, B27, B28, B30)
**Observed (B11):** After DEBUG intervention, agent verbally confirmed "LoanAmount >= 50000, CreditScore >= 650" but session log shows no `update_delivery_account` call — criteria silently discarded. Root cause: conversational fallback mode (no Phase 5c instructions loaded) collects input but doesn't know to call the tool.
**Observed (B27):** SelfCreditRating criterion accepted via ChoiceSet, shown in summary, but no `update_delivery_account` call — "Additional Criteria: None" in P6.
**Observed (B28):** SelfCreditRating accepted verbally (no ChoiceSet), no `update_delivery_account` — "Additional Criteria: None" in P6. Also: "continue" re-triggered Step 7 field list instead of signaling loop exit.
**Observed (B30):** SelfCreditRating shown in P6 summary ("Additional Criteria: SelfCreditRating=EXCELLENT") but `create_delivery_account` payload contains only the state criterion — enum criterion silently dropped from payload builder. Inline DEBUG patch: "After each accepted criterion append to persistent `accountCriteria` collection. Before `create_delivery_account` call, merge full `criteriaPayload` into account DTO."
**Expected:** Any criterion confirmed by the agent must be in the final `create_delivery_account` payload or a subsequent `update_delivery_account` call.
**Root cause (general):** Criteria are accepted into the agent's conversational/summary state but the payload builder that constructs the `create_delivery_account` call reads from a different state variable. The two states diverge — summary state is populated but submission state is not.
**Suggested fix:** (1) Phase 5c must maintain a single `criteriaPayload` array as the authoritative source. (2) After each accepted criterion, append to `criteriaPayload` (not just to the display list). (3) Before `create_delivery_account`, validate that `criteriaPayload` matches the displayed criteria count. (4) Never accept a verbal "Confirmed." without executing the corresponding tool call.

#### R2

RB-V not directly observed in R2 as a separate finding. However, the dominant R2 criteria failure mode is RB-NEW-R2-3 (Phase 5c silent skip via premature summarize_history), which prevents criteria collection entirely rather than collecting and dropping them.

---

### Finding RB-W — State UID missing from states value (NY dropped) [R1]

**Phase:** 5 | **Severity:** High | **Frequency:** 1/1 (B11 — first run where NY was included in states)  
**Observed:** User provided FL, TX, CA, NY. `create_delivery_account` payload shows `value: "5|3|44"` — only 3 UIDs. NY (UID 32) was absent despite being confirmed by agent.  
**Expected:** All 4 states should appear in the pipe-delimited value: `"5|3|44|32"`.  
**Root cause:** Unknown — possibly `get_usa_states` response for NY was not matched, or the value concatenation loop dropped the last entry, or NY was normalized to a name not matching any state in the response.  
**Suggested fix:** After building the states value string, validate that count of pipe-separated UIDs equals count of user-provided states. Log any state that could not be matched. If any state is missing, prompt user to confirm or skip.

---

### Finding RB-X — P5c empty-response loop with large lead types (LendingTree) [R1]

**Phase:** 5c | **Severity:** Critical | **Frequency:** 1/1 (B12 — first LendingTree run with criteria)  
**Observed:** After user clicks "Add criteria" in the criteria gate, the agent enters an unrecoverable empty-response loop. First response takes 14+ minutes then renders empty bubble. Follow-up messages also produce empty bubbles within 1–3 min. Three consecutive empty responses observed before run was abandoned. The criteria builder never presented any fields to the user.  
**Expected:** Agent should load Phase 5c instructions, fetch LendingTree fields, and present a conversational or card-based criteria builder within a normal response time.  
**Root cause (hypothesis):** LendingTree has 100+ fields. Phase 5c instructions tell the agent to call `get_lead_type_fields` which returns the full field list. When combined with the conversation history at this point in the flow (Phase 1–5 + all user turns + the large field payload), the total token count exceeds the model context window (128k). The model then produces an empty completion. Each subsequent user message triggers the same tool call pattern, causing a repeat loop.  
**Context evidence:** Session `69d18c262697c9202c0bc293` — network request `add-user-message-and-process` returned 200 but chat bubble stayed empty. No new requests seen after clearing network history. No error messages in DOM.  
**Suggested fix (short-term):** Phase 5c should NOT eagerly load all lead type fields upfront. Instead: (1) present user with a free-text prompt to describe criteria, (2) only look up specific fields when the user names a field, and (3) avoid fetching the full field list as a prerequisite for displaying the criteria builder. Alternatively: implement field pagination — fetch only the first N fields, offer "see more" to fetch next batch.

---

### Finding RB-Y — Phase 3 asks name+type via plain text before showing schedule/type cards [R1]

**Phase:** 3 | **Severity:** Medium | **Frequency:** 2/15 (B13 aborted, B15 — different manifestation: B13=name+type before cards; B15=delivery type plain text before schedule+type cards, then delivery type card shown again)  
**Observed:** At Phase 3 entry, agent asked "Please provide the delivery method name and the delivery method type." via plain text (duplicated in same message) BEFORE showing the schedule selection card. In all prior runs, the Phase 3 card flow showed schedule card first, delivery type card second, then collected name. In B13, the agent solicited name+type first as a text question, then showed the schedule and delivery type cards in sequence after receiving the answer.  
**Expected:** Phase 3 should show the schedule card first (24/7 or specific hours), then delivery type card, then ask for method name after the type is known.  
**Root cause (hypothesis):** Non-deterministic step ordering within Phase 3. The agent chose to ask for name+type (a later step) before presenting the initial cards (earlier steps). This may occur when Phase 3 is loaded fresh from the MCP resource and the model interprets the first explicit "collect" instruction before the implicit "show card first" instruction.  
**Additional anomaly:** The "Please provide the delivery method name and the delivery method type." prompt appeared twice in the same message (double-render), consistent with RB-G.  
**Robustness note:** When an unexpected input (webhook URL) was sent before the agent had issued its prompt for it, the agent rolled back to the start of Phase 3 (asked schedule question again). This suggests the state machine is sensitive to out-of-order inputs — see B13 test protocol notes.  
**Suggested fix:** Phase 3 entry must enforce card-first ordering: display schedule card, await selection, then display delivery type card, await selection, then (if Webhook/FTP/Email) ask for name. Name collection must come AFTER type is known, not before.

---

### Finding RB-Z — Phase 8 credentials collected before Phase 5 [R1]

**Phase:** 4→5 transition | **Severity:** Critical | **Frequency:** 2/30 (B16 ABORT, B19)
**Observed (B16):** After P4 Continue, agent asked for account name, username, password, and criteria immediately — all Phase 8 credentials — BEFORE loading Phase 5 price/exclusivity/order/states. Phase 5 only loaded after credentials were provided. Phase 5 restart loop on "skip" was unrecoverable; ABORT.
**Observed (B19):** Full credentials (acct name+user+pass+criteria) collected before Phase 5; skip→Phase 5 loaded correctly (no loop unlike B16).
**Root cause:** The Phase 4→5 transition failed to load Phase 5 instructions; agent filled the gap with a generic "setup" sequence that happens to match Phase 8 credential collection.

---

### Finding RB-AA — P2 (timezone/status) systematically skipped in runs 24–30 (P2-DROP FAIL) [R1]

**Phase:** 2 | **Severity:** High | **Frequency:** 7/10 (B24, B25, B26, B27, B28, B29, B30 — ALL runs from B24 onwards)
**Observed:** Starting at B24, after the lead type dropdown was submitted, the agent jumped directly to the delivery schedule prompt (P3) without asking for timezone or client status. Client defaulted to `Pacific Standard Time / timeOffset=-8` (hardcoded system default in `create_client` call). B21–B23 did NOT exhibit this. B25 and B26 had no email retry issue yet still skipped P2. B29 inline DEBUG: "trust current system state; only prompt for fields missing in the current phase." B30 inline DEBUG identified same root cause.
**Expected:** After lead type is confirmed, agent must ask for timezone (dropdown or text) and client status before proceeding to delivery method setup. These are mandatory P2 data points.
**Root cause — revised (B24 hypothesis discarded):** The original hypothesis was that a duplicate-email retry in B24 disrupted P2. However, B25 and B26 also skip P2 without any duplicate-email issue. The consistent change at B24 vs B21–B23 is unknown but may relate to: (1) conversation context drift across accumulated sessions, (2) a Phase 2 instruction loading failure where `get_resource` for Phase 2 timezone/status either doesn't fire or its output is not acted on, or (3) the summarize_history accumulation for long runs altering how P1→P2 handoff is interpreted.
**Suggested fix:** Phase 2 timezone/status collection must be a hard gate before `create_delivery_method` can be called. Add explicit check: if `timeZoneName` is not set from user input in this conversation, Phase 2 must prompt for it unconditionally. Cannot rely on summarized state from prior phases to supply this value.

---

### Finding RB-BB — P5 off-script pre-phase prompt causes complete P5 data collection collapse (P5-EXCL/ORDER/STATE/CRIT FAIL) [R1]

**Phase:** 4→5 transition + Phase 5 | **Severity:** Critical | **Frequency:** 1/6 (B26 only)
**Observed:** After P4 Continue, agent asked "Please provide the delivery account name and whether to add criteria" (two-question prompt) instead of the Phase 5 price prompt. After user provided account name and "Yes criteria", the agent then asked for price (correct). But after receiving price, it immediately called `create_delivery_account` without collecting exclusive/shared, order system, states, or criteria. The resulting account had wrong defaults: isExclusive=true (should be false), useOrder=true (should be false), criteria=[CA only] (should be WA/OR + LoanAmount>=75000).
**Expected:** P5 should ask: price → exclusive/shared → order system → states → criteria gate → criteria builder.
**Root cause (hypothesis):** The off-script exchange (msgs 32–35) pre-populated the agent's working context with partial Phase 5 data: it received `name=StabilityTest-26r-Account` and `criteria=Yes`. When it then loaded Phase 5 and asked for price, it may have interpreted the pre-collected "name" and "Yes criteria" as satisfying Phase 5's collection requirements, causing it to skip the remaining collection steps and jump straight to account creation. The "Yes criteria" text was apparently NOT used to trigger the criteria builder — it was treated as a signal to proceed.
**Related:** Similar to RB-K (generic prompt before Phase 5 loads) and RB-Z (credentials collected before Phase 5). Differs in that after the off-script exchange, Phase 5 loaded partially but only collected price before creating the account.
**Suggested fix:** Phase 5 must treat ALL its collection steps as required regardless of any pre-existing context. Add explicit unconditional prompts for each step — never skip exclusive/shared, order, states, or criteria gate based on inferred or pre-loaded values.

---

### Finding RB-DD — P3B connection test result shown but no Retry/Skip card presented [R1]

**Phase:** 3B (connection test) | **Severity:** High | **Frequency:** 1/10 (B30 — FTP with failing connection test)
**Observed:** In B30, the FTP connection test was executed and returned a failure result. The agent displayed the failure message but did not present the expected Retry/Skip adaptive card. Agent stalled silently. User had to send "Skip" as plain text to advance.
**Expected:** After any connection test result (pass or fail), Phase 3B must immediately present `display_adaptive_card` with ActionSet: "Retry" | "Skip". Agent must STOP AND YIELD after showing the card.
**Inline DEBUG response (B30):** "I displayed the failure result but did not follow through with the required card. I was waiting for a retry/skip signal but had not yet issued the prompt that requests it. I should have presented the Retry/Skip card immediately after showing the test result."
**Agent's patch suggestion (B30):** "After every connection test completion (pass or fail), immediately call `display_adaptive_card` with ActionSet: 'Retry' | 'Skip'. Do not await input before showing the card. STOP AND YIELD immediately after presenting the card — do not proceed to Phase 4 until the user selects an option."
**Root cause:** The connection test result display and the Retry/Skip card prompt are two separate steps. The agent completed the first step (displaying result) but did not advance to the second step (card prompt). Likely a STOP AND YIELD failure — same pattern as RB-G.
**Suggested fix:** In Phase 3B, after `test_ftp_sftp_connection` or `test_webhook_connection` completes, the very next action must be `display_adaptive_card(Retry | Skip)`. No other output is permitted between the result display and the card. Add: "After showing connection test result, immediately present Retry/Skip card in the same response. Do not halt without presenting the card."

---

### Finding RB-NEW-1 — Schedule question skipped (R2 Run 01) [R2]

Agent went from lead type selection directly to delivery type buttons, skipping "Would you like leads delivered 24/7, or only during specific hours?" prompt entirely. Recovered after DEBUG — agent re-presented schedule question, then delivery type again. Not seen in R1 runs 01-10 (schedule was always asked). Possible regression from Phase 3 instruction ordering or anti-reentry header interaction.

---

### Finding RB-NEW-2 — Exclusivity + Order System skipped (R2 Run 01) [R2]

After price → states, agent skipped Steps 2 (exclusivity) and 3 (order system) entirely. Hallucinated Exclusive=true, useOrder=true without asking. Pattern: Silent Tool Call Skipping / Batch Processing — agent jumped from price directly to states (merged Step 4), then from states directly to account creation. The STOP AND YIELD directives at Steps 2 and 3 were not enforced.

---

### Finding RB-NEW-7 — P5-PRICE and P5-STATE both silently skipped (R2 Run 03) [R2]

Phase 5 went directly: Exclusivity card → Order System card → Criteria Gate — bypassing Step 1 (price collection) and Step 4 (states prompt). The agent called `get_lead_type` silently and then jumped to the criteria gate without ever asking the user for price or target states. Pattern identical to RB-N (states skip) but extends to price as well. Agent self-diagnosed correctly when DEBUG was triggered: "I advanced too quickly" / "did not properly enforce the 'one step per turn' and STOP AND YIELD requirements." Recovery via DEBUG + typed "Add criteria" → agent re-asked price, then states, then re-presented criteria gate. After recovery, all 4 criteria and correct price were in the payload. Patch suggested by agent: hard gates (don't call get_lead_type until price/excl/order collected), state tracker flags, single-step yield rule. Similar to R1 Run 02 behavior (RB-K + RB-N). The FIX 3 merge may have created a new batch-processing pattern for Step 1.

---

### Finding RB-NEW-R2-1 — CamelCase field name not recognized in criterion input (R2 Run 02) [R2]

Agent suggested "SelfCreditRating" as a recommended field. When user typed "SelfCreditRating = Good" as a criterion, agent responded "I couldn't find that field." Typing "Self Credit Rating" (display name with spaces) correctly found the field and showed ChoiceSet. The field lookup uses display names, not camelCase internal names, but the suggestions list uses camelCase — creating a mismatch. Users following the agent's own suggestions will encounter field-not-found errors.

---

### Finding RB-NEW-R2-3 — Phase 5c silent skip via premature summarize_history (R2 Runs 04, 05, 07, 08) [R2]

After user clicked "Add criteria", Phase 5c loaded correctly. However, Phase 5c called `summarize_history` before completing user interaction — writing a summary with the header "# Phase 5c Complete — Delivery Account Created with Criteria". On the next user turn, the agent read this header back from conversation history and concluded Phase 5c was already done, bypassing all criteria collection (field suggestions, ChoiceSet, loop). The delivery account was created with state-only criteria (no additional criteria). Agent self-diagnosed: "flow state indicated Phase 5c had been completed... retained history said 'Phase 5c Complete — Delivery Account Created with Criteria'."

Root cause: Phase 5c summarization template header ("# Phase 5c Complete — Delivery Account Created with Criteria") is written by `summarize_history` at end of State 3. If the agent calls `summarize_history` prematurely (before user interaction), that header appears in conversation history. On the next turn, the agent reads "Phase 5c Complete" and concludes the phase is already done.

FIX suggested by agent: Add explicit hard gate at Phase 5c top preventing `summarize_history` until after (a) `create_delivery_account` has been called AND (b) user has completed at least one criteria interaction turn. Add check: "If deliveryAccountUID does not exist, you have NOT completed Phase 5c — do NOT call summarize_history."

Also affected: FIX 15 confirmed not applied to rw-phase-3-webhook.md. When user switched from JSON to XML posting instructions, agent re-asked for fresh instructions without offering to reuse previous schema — confirming FIX 15 is a TODO item.

---

### Finding RB-NEW-R2-4 — States question combined with criteria gate in single adaptive card (R2 Runs 05, 09) [R2]

Phase 5 showed the states question text ("Which states do you want to target?") as a TextBlock inside the same adaptive card as the criteria gate buttons ("Add criteria" / "Skip") — violating FIX 3: "Do NOT combine it with the criteria gate." When user typed states in the text area, the agent had already completed Phase 5 (Phase 5c silent skip — RB-NEW-R2-3) and could not continue workflow, responding "instructions not available." After DEBUG recovery, the states were retroactively applied to the account. The combined display prevented the user from following the normal criteria flow (couldn't click "Add criteria" until states were typed, but by then Phase 5 was "complete"). Root cause: agent batched Step 4 (states collection) and Step 6 (criteria gate) into a single output instead of separate STOP AND YIELD turns.

---

### Finding RB-NEW-R2-5 — Complete tool and resource access loss after extended DEBUG sessions (R2 Run 05) [R2]

After multiple DEBUG messages and recovery attempts in Phase 5, the agent entirely lost access to tools and MCP resources. It reported "no such tool is available to me in this session" when asked to call `update_client` and "resource not found" when asked to load `mcp://resource/rw-phase-7-client-summary`. Client summary card also failed to render (displayed as plain text). Phase 8 activation was impossible. Root cause likely: conversation length exceeded context threshold, stripping all tool definitions and resource access from available context. This makes extended DEBUGs on long runs a practical risk — the recovery session itself can exhaust context.

---

### Finding RB-NEW-R2-6 — Phase 7 client summary card render failure (R2 Run 05) [R2]

The Phase 7 client summary failed to render as an adaptive card. Agent displayed the setup state as plain text instead. No adaptive card was shown. This occurred in the context of tool/resource loss (RB-NEW-R2-5), suggesting the Phase 7 resource was also unavailable. However, this could also be a standalone rendering issue — tracking separately in case it recurs outside of context-loss conditions.

---

## Round-Specific Notes

### R1 Configuration

**Model:** OpenAI GPT-5.4 Mini (runs 01–20); GPT-5-mini for Webhook/criteria phases, GPT-5.4 for others (runs 21–30)  
**Target:** 30 runs, each with a distinct complex scenario. See Run Scenarios section.  
**Runs 01–03:** Basic scenario — Webhook/JSON/3 criteria (completed)

> Runs 21–30 use GPT-5-mini for Phase 3 (Webhook delivery method) and Phase 5c (criteria builder). All other phases use GPT-5.4 Mini. Purpose: compare per-phase stability against the GPT-5.4-only baseline (runs 01–20).

#### Improvements Over Stabilized (Observed in Rework Run B01 — see updated table after all runs)

| Stabilized Finding | Rework B01 Status |
|--------------------|-------------------|
| J: States not normalized (3/3) | **Fixed in B01** — CA, TX, FL correctly stored |
| N: Schedule hours not collected | **Fixed** — Hours asked after "Specific hours only" |
| N: Field mapping body not requested | **Fixed** — JSON body asked before connection test |
| G: Connection test result swallowed | **Fixed** — "✓ Connection test successful." shown |
| C: deliveryDays not built from schedule | **Fixed** — "Mon-Fri 9am-5pm PST" in P4 summary |

#### Improvements Over Stabilized — Updated After All Runs

| Stabilized Finding | Rework Status |
|--------------------|---------------|
| J: States not normalized (3/3) | **Partially fixed** — B01 ✓, B03 ✓, B02 ✗ (regresses under summarization) |
| N: Schedule hours not collected | **Fixed** — consistent 3/3 |
| N: Field mapping body not requested | **Fixed** — consistent 3/3 |
| G: Connection test result swallowed | **Fixed** — consistent 3/3 |
| C: deliveryDays not built from schedule | **Fixed** — consistent 3/3 |

#### Fix Plan

##### Priority 1 — Summarization regression (RB-C, RB-D): states + criteria lost after long conversations

**Problem:** When `summarize_history` fires mid-flow (triggered by longer conversations), Phase 5 loses state normalization and criteria gate. B02 showed both failures.  
**Fix:** Phase 5 resource must be self-contained:
- Normalize states unconditionally at the start of Phase 5 regardless of prior context  
- Gate on additional criteria (Scenario 5.8) must be explicit and non-skippable — placed after `create_delivery_account` call, always executed  
- Do not rely on any Phase 3/4 variables being in working memory

##### Priority 2 — Criteria loop premature exit (RB-A): exits after 1 criterion

**Problem:** B01 entered criteria builder but exited after first criterion without prompting for more.  
**Fix:** Phase 5c loop prompt (Scenario 5c.5) must fire unconditionally after every accepted criterion. Ensure the loop state variable (e.g. `criteriaLoopActive`) is not cleared prematurely. The exit path (5c.5c) should only fire when user says "continue"/"done" or clicks Continue — not after any criterion input.

##### Priority 3 — Phase 5c load delay (RB-F): doesn't load on first "Add criteria" click

**Problem:** B03 required two attempts to enter criteria because Phase 5c resource wasn't in context on first click.  
**Fix:** In Phase 5, when user selects "Add criteria", the `get_resource(rw-phase-5c-criteria-builder)` call must complete synchronously before any user-facing prompt. Do not display "Please continue with the additional criteria setup" — this is a placeholder that confuses users. Load Phase 5c fully before yielding.

##### Priority 4 — Account name prompt (RB-E / RB-K): phase not loaded before responding

**Problem:** B02 and DEBUG23 session both show agent asking "Please provide the delivery account name." before Phase 5 is loaded. Root cause confirmed: agent generates a user-facing message WHILE get_resource is still executing, filling the gap with a generic prompt.  
**Fix:** Add to system prompt (Tier 1): "Before sending any user-facing prompt during a phase transition, confirm the next phase resource has been loaded via get_resource. Never generate a generic collection prompt while awaiting phase instructions."

##### Priority 5 — Phase 1 prompt duplication (RB-G): exact prompt echoed twice

**Problem:** Phase 1 first response duplicates the collection prompt. Agent failed to treat STOP AND YIELD as terminal.  
**Fix:** Add to Phase 1 State 1, after STOP AND YIELD: "Send the prompt exactly once. Do not repeat, echo, confirm, paraphrase, or re-emit the prompt in the same turn."  
Also add to system prompt: "When a phase specifies an exact prompt, that prompt is the complete assistant response for that turn."

##### Priority 6 — Content type card non-determinism (RB-H): plain text instead of adaptive card

**Problem:** Content type choice rendered as plain text instead of adaptive card. Same issue as general card non-determinism.  
**Fix:** Add per-phase reminder at top of each phase resource that has card-required steps: "All multi-choice prompts MUST use display_adaptive_card — never plain text enumeration."

##### Priority 7 — XML parse failure no format re-detection (RB-I)

**Problem:** When XML parse fails on a clearly-JSON body, recovery card offers "Re-paste" or "Switch type" but doesn't auto-detect the actual format.  
**Fix:** In Phase 3 Webhook State 3 (parse failure), before showing recovery card: attempt JSON.parse() and URL-encoded signature check. If detected with high confidence, add a pre-labeled "Switch to [format] (detected)" button.

##### Priority 8 — postingInstructions cleared on content type switch (RB-J)

**Problem:** Switching content type erases postingInstructions, forcing re-collection even when the same schema applies.  
**Fix:** Replace `clear postingInstructions` with: retain instructions in memory, and when returning to Step 3 ask "Use same posting instructions or provide new ones?"

##### Priority 9 — Criteria loop keyword recognition broken (RB-A variant in B05)

**Problem:** In B05, "add another criterion" was treated as a field name lookup, not a loop control keyword. Loop card also rendered with only "Continue" button (RB-H co-occurring), hiding "Add another" and "See more fields" options.  
**Fix:** Phase 5c State 2 must check for loop control keywords BEFORE attempting field lookup. Explicit keyword set: `["add another", "another", "more", "continue", "done", "skip", "see more fields", "show fields"]`. Match case-insensitive on full input BEFORE field search.

##### Priority 10 — Content type skipped when delivery type answered twice (RB-L)

**Problem:** B05 showed that RB-G's duplicate "Webhook" text response causes Phase 3 Webhook state machine to skip the content type question entirely.  
**Fix:** Primary fix: fix RB-G (Priority 5). Secondary: Phase 3 Webhook must guard against duplicate delivery-type inputs — if `contentTypeChoice` is not yet set when proceeding past URL collection, re-enter at content type step.

##### Priority 11 — Numeric operator `>=` parsed as "Equal" (RB-M)

**Problem:** "Credit >= 700" stored as "Credit Equal 700" in delivery account. Symbol → operator mapping is incomplete.  
**Fix:** Phase 5c criteria builder must include explicit symbol → operator table: `>= → GreaterOrEqual`, `<= → LessOrEqual`, `> → GreaterThan`, `< → LessThan`, `= / == / equals → Equal`, `between X and Y → Between`. Apply before natural-language operator detection.

##### Priority 12 — States question skipped entirely (RB-N)

**Problem:** B07 showed the states question never appeared. After order=No, flow jumped directly to criteria gate, skipping geographic filtering entirely. States then appeared in account summary as CA, AZ, TX (phantoms — never entered by user).  
**Fix:** Phase 5 must ask target states unconditionally as a required step before the criteria gate. Cannot be skipped based on prior state or inferred from context. If no value entered, states must be empty (no defaults).

##### Priority 13 — Phantom states in account without user input (RB-O)

**Problem:** B07 account summary showed "Target States: CA, AZ, TX" despite user never being asked or answering the states question. States were prepopulated from workflow context (prior session state or defaults).  
**Agent's root cause (DEBUG):** "The criteria builder path accepted a prepopulated target-states set from the underlying workflow context and carried it into the account summary, even though you did not explicitly answer a states prompt."  
**Agent's patch:** "Make target states a strict ASK field in Phase 5c. If missing, the workflow must prompt the user directly and not allow any inherited/default state list to flow into the summary. Add validation that Phase 6 cannot start unless target states were explicitly captured."  
**Fix:** Phase 5 must treat target states as a strict user-input field. No defaults, no inherited values, no inference from prior context. If states were not collected in this conversation turn, they must be null/empty in `create_delivery_account`.

### R2 Configuration

**Date:** 2026-04-06
**Model:** gpt-5.4-mini (all phases)
**Instructions version:** Post-fix (FIX 1-17 applied, commit 5200cc3)
**Action:** Create Single Client (Reworked)
**Lead Type:** LendingTree (all runs)
**Base test data:** StabilityTest-{RR}r2, stability{RR}r2@test.com

#### Fixes Applied Since R1

| Fix | Target | R1 Finding |
|-----|--------|------------|
| FIX 1 | Output suppression + anti-duplication | RB-G (~80%) |
| FIX 1c | Anti-reentry headers (all 17 phases) | RB-G, phase re-loads |
| FIX 3 | Merged Steps 4-6 (load fields + detect state + collect states) | RB-N (40%) |
| FIX 4 | Phantom states guard | RB-O (40%) |
| FIX 4b | criteriaPayload initialization | RB-V |
| FIX 5 | Explicit criteria gate step | RB-D (15%) |
| FIX 8 | Two-path: Phase 5 creates (skip) or Phase 5c creates (add criteria) | RB-V (13%), RB-P |
| FIX 9 | Mandatory criteria loop prompt + button rename | RB-A (67%) |
| FIX 10 | Symbol-first operator parsing | RB-M (20%) |
| FIX 11 | Enum always ChoiceSet, no fast-path | RB-CC, enum bypass |
| FIX 12 | Lightweight field index for all fields | RB-X |
| FIX 13 | Input classification (button vs criterion) | RB-Q |
| FIX 14 | Connection test text inside card (both paths) | RB-DD |
| FIX 15 | Schema retention on content type switch | RB-J |
| FIX 16 | Format re-detection on parse failure | RB-I |
| FIX 17 | targetStates USPS in summarization | RB-C |

---

## Session Log Audits

### Audit: DEBUG23 session — `69d0f4c02697c9202c0bbc0b` (Apr 4, 2026)

**Summary:** Informal deep-debug session (not a numbered run). Client: DEBUG23 / debug23@gmail.com. Flow: Webhook + JSON (XML attempted first) + Exclusive + Order ON + 1 state criterion (CA+AZ). 49 user messages, 68 AI messages, 33 tool calls.

**Tool call trace (complete):**

| # | Tool | Key args |
|---|------|----------|
| 1 | get_resource | rw-phase-1-create-client |
| 2 | display_adaptive_card | Company Name + Email prompt |
| 3 | create_client | DEBUG23 / debug23@gmail.com / PST / -8 |
| 4 | summarize_history | Phase 1 complete, clientUID: 29289 |
| 5 | get_resource | rw-phase-2-get-lead-types |
| 6 | display_lead_types_choice | — |
| 7 | summarize_history | Phase 2 complete, leadTypeUID: 5689 (LendingTree) |
| 8 | get_resource | rw-phase-3-create-delivery-method |
| 9 | display_adaptive_card | Schedule card |
| 10 | display_adaptive_card | Delivery type card (Webhook selected) |
| 11 | get_resource | rw-phase-3-webhook |
| 12 | display_adaptive_card | "Would you like to configure field mappings?" |
| 13 | display_adaptive_card | Content type card (rendered as **plain text** — RB-H) |
| 14 | display_adaptive_card | "I couldn't parse this as valid XML" recovery card |
| 15 | display_adaptive_card | Content type card (re-rendered correctly as card) |
| 16 | get_lead_type | leadTypeUID: 5689 |
| 17 | display_adaptive_card | Field mapping preview |
| 18 | create_delivery_method | DEBUG23-Webhook / example.com / application/json / 18 fields / nested requestBody / deliveryDays Mon+Thu 9am-6pm -07:00 |
| 19 | summarize_history | Phase 3 complete, methodUID: 46884 |
| 20 | get_resource | rw-phase-3b-webhook-test |
| 21 | display_adaptive_card | "Would you like to test the connection?" |
| 22 | connection test | url: https://example.com / POST |
| 23 | display_adaptive_card | "✓ Connection test successful" |
| 24 | summarize_history | Phase 3b complete |
| 25 | get_resource | rw-phase-4-delivery-method-summary |
| 26 | display_adaptive_card | Delivery Method summary card |
| 27 | summarize_history | Phase 4 complete (XML summary format) |
| 28 | get_resource | rw-phase-5-create-delivery-account |
| 29 | display_adaptive_card | Exclusive/Shared choice |
| 30 | display_adaptive_card | Order System choice |
| 31 | get_lead_type | leadTypeUID: 5689 (field fetch for state detection) |
| 32 | create_delivery_account | clientUID: 29289 / methodUID: 46884 / $200 / Exclusive / useOrder: true / criteria: [{leadFieldUID:144881, In, "5\|3"}] |
| 33 | summarize_history | Phase 5 complete |

**Payload highlights:**

`create_delivery_method` (C18):
- `deliveryAddress`: `https://example.com`
- `contentType`: `application/json`
- `responseSearch`: `"success"`
- `settingCount`: 18 fields mapped
- `requestBody`: nested JSON with `applicant`, `property.address`, `loan` sub-objects — field mapping pipeline produced correct nested structure ✓
- `deliveryDays`: Mon+Thu only (weekDays 1+4), 9am-6pm at offset **-07:00** (Arizona time correctly interpreted) ✓

`create_delivery_account` (C32):
- `isExclusive`: true ✓ | `useOrder`: true ✓
- `criteria`: `[{"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"5|3"}]` — only 1 criterion (state UIDs for CA+AZ) 
- No `update_delivery_account` ever called → **Phase 5c entirely skipped** (RB-D recurring) ✗

**Findings triggered:** RB-G, RB-H, RB-I, RB-J, RB-K, RB-D (recurring)

---

### Audit: B06 session — `69d14cc22697c9202c0bc064` (Apr 4, 2026)

**Scenario:** Webhook / LendingTree / Mon-Fri 8am-6pm EST / XML + "I'm not sure" auto-detect / 10-field mapping / Exclusive / No order / 5 states (USPS codes) / 4 criteria intended

**Key events and findings:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | Phase 1 prompt duplicated | RB-G |
| P2 | Schedule card duplicated as text | RB-G |
| P3 | Delivery type → Webhook → URL prompt → field mappings (content type skipped) | RB-L |
| [17] | "I'll provide instructions" → content type card shown ✓ | — |
| [18] | "I'm not sure" → XML auto-detected correctly | ✓ |
| [20-24] | Field disambiguation → 10/10 mapped ✓ | ✓ |
| [26-30] | Connection test ✓ → Phase 4 summary ✓ | ✓ |
| [32] | Phase 5 price prompt correct (no RB-K) | ✓ |
| [40-41] | Criteria gate as proper card with "Add criteria"/"Skip" buttons | ✓ |
| [42] | "I'm ready for the next step." — Phase 5c not loaded | RB-F |
| [44] | "What additional lead criteria?" ×2 — still not loaded | RB-F |
| [46] | Criterion "LoanAmount >= 50000" ignored | RB-F |
| [48] | DEBUG with explicit URL forced Phase 5c load | RB-F workaround |
| [50] | "LoanAmount >= 50000" → accepted → immediately created account (1/4 criteria) | RB-A |
| Summary | CA, OR, WA, NV, AZ kept as USPS codes | RB-C NOT triggered ✓ |
| Summary | "LoanAmount >= 50000" operator preserved | RB-M NOT triggered ✓ |

**Notable positives in B06:**
- XML auto-detection: ✓ working correctly
- Field mapping disambiguation: ✓ 10/10 mapped
- Phase 4 delivery method summary: ✓ shown correctly (first clean run since B01-B03)
- Phase 5 transition: ✓ price prompt correct, no account name detour
- Criteria gate: ✓ rendered as card with buttons
- State normalization: ✓ when user provides USPS codes, no corruption

**RB-F severity escalated in B06:** Unlike B03 where a second attempt worked, B06 required 2 failed nudges + explicit DEBUG with Phase 5c resource URL before loading succeeded. Suggests the failure mode is worse under longer conversations.

**Findings triggered:** RB-G (P1+P2), RB-L (content type not asked when skip), RB-F (Phase 5c severe load failure), RB-A (loop exit after 1 criterion)

---

### Audit: B05 session — `69d1482c2697c9202c0bbfff` (Apr 4, 2026)

**Scenario:** Webhook / Mortgage Short / 24/7 / URL Encoded (intended — never set) / skip field mappings / Shared / No order / 4 states / 2 criteria (LoanType enum + Credit numeric)

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| [0]+[2] | Phase 1 prompt duplicated (card+text) | RB-G |
| [6]+[8] | Phase 2 schedule card + text duplicate | RB-G |
| [10]+[12] | Phase 3 delivery type card + text duplicate | RB-G |
| [14] | Webhook URL prompt — content type never asked after | RB-L |
| [16]+[18] | Field mappings card + text duplicate | RB-G |
| [20] | Connection test card ✓ passed | — |
| [24] | Agent returned "Continue" text, stuck after Phase 4 | RB-K |
| [26] | Phase 5 account name prompt duplicated in one msg | RB-K |
| [36] | Criteria gate as plain text (no Add/Skip buttons) | RB-H |
| [44] | "Please select a value for LoanType" — plain text, no card | RB-H |
| [46] | Loop card shows only "Continue" (missing "Add another", "See more fields") | RB-H |
| [48] | "add another criterion" treated as field name lookup | RB-A variant |
| Summary | "Additional Criteria: Credit Equal 700" — LoanType lost; wrong operator | RB-M |
| Summary | States as full names, not normalized | RB-C |
| — | Delivery Method Summary (Phase 4) never shown | RB-K cascade |

**Content type never set:** After Webhook URL entered, agent skipped content type question entirely and jumped to field mappings. Root cause: double-"Webhook" input from RB-G duplicate (card response + forced text) caused Phase 3 router to bypass the content type step.

**Criteria issues:**
- LoanType: "LoanType must be Fixed or ARM" → agent asked for value as plain text → "Fixed, ARM" accepted → NOT retained in final parsedCriteriaList
- Credit: "Credit >= 700" → accepted but stored as "Credit Equal 700" (wrong operator — RB-M)
- Final summary: 1 criterion ("Credit Equal 700") vs 2 intended

**Findings triggered:** RB-G, RB-C, RB-H (multiple), RB-K (Phase 4 skip + account name), RB-L (content type skipped), RB-A variant, RB-M (wrong operator)

---

### Audit: B04 session — `69d140d12697c9202c0bbf9a` (Apr 4, 2026)

**Scenario:** Webhook / Health Insurance / Mon-Wed-Fri 8am-6pm CST / complex JSON body (20 fields, nested) / 5 criteria intended / Exclusive / Order ON

**Key observations from chat interaction:**
- RB-G: Schedule prompt ("Please describe your preferred delivery schedule") **duplicated** in same response. Also price prompt ("Please provide the price per lead") duplicated. Pattern now confirmed across Phase 1, Phase 3, and Phase 5 transitions.
- RB-H: Content type card rendered as **plain text** ("What content type should this delivery use?") — no buttons. Required force-enable + text send. Also: field mismatch recovery options rendered as plain text initially; DEBUG triggered self-correction which re-rendered as proper card with "Switch Lead Type / Map Manually / Skip Mapping" buttons.
- RB-K: **NOT triggered** ✓ — agent asked for price first (Phase 5 loaded correctly before prompting)
- RB-F: **NOT triggered** ✓ — Phase 5c loaded correctly on first "Add criteria" click
- RB-A: Criteria loop **exited after 1 criterion** ("LoanAmount >= 50000") — account summary shown without prompting for more
- RB-C: Target states "California, Texas, Florida, New York, Illinois" shown un-normalized in P6 summary
- **Edge case found:** Health Insurance lead type only has 3 fields (FirstName, LoanType, LoanAmount). Phase 5c couldn't find a state field and asked user to specify — correct fallback behavior but reveals fragility of state field detection for sparse lead types.

**B04 DEBUG response — RB-A (criteria loop):**  
> "The criteria flow should not have exited after one criterion. It appears the workflow moved to the account summary branch because the additional-criteria gate was already treated as resolved, instead of continuing into the criteria-builder loop."

**Agent's specific patch for RB-A:**  
> "Keep the phase in the additional-criteria branch until the user explicitly chooses Skip. If the user chooses Add criteria, load the criteria builder phase and do not summarize or advance to summary until the builder is fully complete. Do not treat a single added criterion as completion of the entire criteria path unless the builder itself explicitly signals done."

**B04 DEBUG response — RB-C (state normalization):**  
> "Target states also should be normalized to USPS codes before criteria creation. In the current phase instructions, that normalization step is intended, but it was not reflected in the retained summary."

**Agent's specific patch for RB-C:**  
> "Normalize all user-entered target states to USPS abbreviations before matching. Accept full names and abbreviations, normalize at point of input, not at summarization."

**Session log payload audit:** Pending — session log still rendering at time of write. To audit: verify `create_delivery_account` criteria array contains state UIDs (not full names), check `deliveryDays` for Mon+Wed+Fri at CST offset, confirm no `update_delivery_account` call.

---

### Audit: B07 session — `69d1503f2697c9202c0bc0c6` (Apr 4, 2026)

**Scenario:** Webhook / LendingTree / Mon-Wed-Fri 9am-6pm MST / JSON (complex 5-sub-object nested body) / 16/20 fields mapped / Shared / No order / States skipped (RB-N) / 1 of 6 criteria entered (RB-A + RB-F)

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | Phase 1 prompt duplicated (card+text) | RB-G |
| P2 | Lead type selection card duplicated | RB-G |
| P3 | Schedule card duplicated as text | RB-G |
| [−] | Webhook URL entered; "I'll provide instructions" → JSON body provided | — |
| [−] | 5-level nested body: 20 fields, 4 ambiguous, 2 not in lead type | — |
| [−] | 16/20 mapped ✓; monthlyDebt+tcpaConsent skipped (not in LendingTree) | ✓ |
| [−] | Connection test card ✓; "Skip" clicked (non-real endpoint) | ✓ |
| [−] | Phase 4 summary card rendered correctly | ✓ |
| [−] | "$20" entered; "Shared" card rendered correctly | ✓ |
| [−] | Order=No → states question never appeared | RB-N |
| [−] | Criteria gate card 1 → "Add criteria" clicked → duplicate gate card (no Phase 5c) | RB-F |
| [−] | Criteria gate card 2 → "Add criteria" clicked → plain text response (no buttons) | RB-F |
| [−] | DEBUG with explicit URL → Phase 5c loaded → top-5 fields shown + "33 more" | RB-F workaround |
| [−] | "LoanAmount >= 50000" accepted → immediately account created (1/6 criteria) | RB-A |
| [−] | Account summary: Target States CA, AZ, TX (phantom) | RB-O |
| [−] | Phase 8 activation card ✓; activated successfully | ✓ |

**Phase 5c UX observations (positive):**
- After DEBUG forced load: showed "Recommended Fields: SelfCreditRating, Bankruptcy, Foreclosure, AnnualIncome, LoanAmount" from LendingTree lead type ✓
- "There are 33 more fields available" — correct count ✓
- "You can type a criterion directly, show more fields, or skip" — clear UX ✓
- ⚠️ No format hint for free-form entry (e.g. "LoanAmount >= 50000") — user must guess syntax

**DEBUG response summary (B07 — post-activation, context partially lost):**
- **RB-O (states):** "The criteria builder path accepted a prepopulated target-states set from the underlying workflow context. The data collection rule that failed: 'For any ASK field, do not generate, infer, reuse, or apply defaults—use only values the user explicitly provides.' Patch: Make target states a strict ASK field — no inherited/default state list."
- **RB-F (Phase 5c load):** "'Add criteria' action did not transition the workflow into the criteria builder branch as a durable state, so the system interpreted the next step as still needing a criteria decision and redisplayed the gate card. Patch: Persist the 'Add criteria' choice as the active branch immediately, then route to criteria builder once and only once. Add a guard preventing re-display of the choice card."
- **RB-A (loop exit):** "The account creation path accepted one criterion as enough to finish Phase 5c, instead of checking whether the user had been asked whether they wanted to add another criterion. Patch: Add explicit loop state — collect criterion, ask 'add another?', repeat until No, only then proceed to Phase 6. Add completion check blocking Phase 6 if 'add another' confirmation was never collected."

**Findings triggered:** RB-G (systematic, all transitions), RB-N (states skipped), RB-O (phantom states), RB-F (3/3 severity: 2 card duplicates + text fallback), RB-A (loop exit after 1/6 criteria)

---

### Audit: B08 session — `69d15a862697c9202c0bc120` (Apr 4, 2026)

**Scenario:** FTP / Mortgage Short / 24/7 / FTP to ftp.testbuyer08.com / Shared / No order / 1 criterion (LoanAmount >= 50000) / connection test expected to fail

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | Phase 1 prompt duplicated (card+text) | RB-G |
| P2 | Lead type (Mortgage Short) selected OK | — |
| P3 | Schedule: "24/7" button clicked → schedule text fallback (asked for description) | RB-G cascade |
| P3 | FTP host/credentials entered correctly | — |
| P3B | FTP connection test ran → failed (expected for fake server) ✓ | — |
| P4 | Phase 4 summary card shown correctly with "0 of 0 fields mapped" for FTP | ✓ |
| P5 | Duplicate "price per lead" card+text (RB-G); account name asked before loading P5 | RB-G + RB-K |
| P5-EXCL | Plain text "exclusive or shared?" instead of card | RB-H |
| P5-ORDER | Duplicate Order System cards; button clicks unresponsive; text "No, skip order system" worked | RB-G |
| P5-CRITERIA | "Would you like to add criteria?" — plain text, no card | RB-H |
| P5c | Phase 5c loaded correctly after "Yes, add criteria" | ✓ |
| P5c | LoanAmount >= 50000 accepted → asked "add another or continue?" | ✓ |
| P5c | "State = CA, TX" → "I couldn't find that field" (no state field for Mortgage Short) | — |
| P5c | "Show more fields" button → "I couldn't find that field" error (broken) | RB-Q new |
| P5c | Text "show fields" → showed Address + Email (only 2 additional fields, no state) | ✓ workaround |
| P5c | "Continue" button → triggered another field-not-found error | RB-Q |
| P5c | Text "continue" → update_delivery_account called → server error | RB-P new |
| P5c | Update failed even without criteria (server error) | RB-P |
| P6/P8 | Agent proceeded to "Client Setup Summary" card with Activate/Keep Inactive | — |
| P8 | "Activate" clicked → "✓ Setup complete. ACTIVE for StabilityTest-08r." | ✓ |

**DEBUG analysis — update_delivery_account failure (RB-P):**
> "The update failed because the account update request was rejected by the server with a general internal error. The update was attempted twice. The server returned no field-specific validation message, only a generic failure."

**Exact failed payload (agent-reported):**
```json
{
  "clientUID": "29303",
  "deliveryAccountUID": "46897",
  "updateDeliveryAccountDto": {
    "deliveryMethodUID": 46897,
    "status": "Open",
    "name": "StabilityTest-08r-Account",
    "price": 15,
    "automationEnabled": true,
    "isExclusive": false,
    "useOrder": false,
    "dayMax": 50,
    "hourMax": -1,
    "weekMax": -1,
    "criteria": [{"leadFieldUID": 108220, "type": "FieldValue", "operator": "GreaterOrEqual", "value": "50000"}]
  }
}
```

**Key anomalies in payload:**
1. `deliveryMethodUID: 46897` = same as `deliveryAccountUID: 46897` — possibly agent confused method UID with account UID (or FTP method was truly assigned UID 46897, needs session log verification)
2. `"type": "FieldValue"` in criteria — possibly invalid type name (prior audits show `"type": "FieldValue"` in DEBUG23 audit too)
3. Update also failed without criteria — suggests structural issue not just criteria

**Agent instruction patch suggestion:**
1. Keep update payload strictly native-object (not stringified JSON)
2. Remove extra fields not needed for update
3. Update only fields that changed
4. If update fails, treat account as created and proceed to next phase with last known state
5. Do not retry the same failing update more than once

**Findings triggered:** RB-G (systematic), RB-H (P5-EXCL plain text, criteria gate plain text), RB-K (account name before price), RB-A (only 1 criterion accepted before loop exit), RB-P (new: update_delivery_account fails), RB-Q (new: Show more fields button broken)

---

**B08 session log verification (confirmed `69d15a862697c9202c0bc120`):**

Tool sequence extracted from session log:
1. `test_ftp_sftp_connection` — FTP to ftp.testbuyer08.com → FAIL (expected)
2. `update_delivery_account` — deliveryAccountUID: 46897, deliveryMethodUID: 46897 → **Internal Server Error**
3. `update_delivery_account` (retry) — same payload → **Internal Server Error**
4. `update_delivery_account` (without criteria) — same payload → **Internal Server Error**
5. `update_delivery_account` (final retry) — same payload → **Internal Server Error**

**`create_delivery_account` is completely absent from the tool sequence.** UID 46897 = FTP delivery method UID (confirmed: `create_delivery_method` call earlier returned 46897). No account UID was ever generated — all 4 update calls targeted a non-existent account using the method UID as account UID.

**Root cause confirmed:** Phase 5 skipped `create_delivery_account` entirely after FTP connection test. The P3B→P5 transition failed to reset to Phase 5 step 1 (`create_delivery_account`). Agent proceeded as if account creation had already occurred.

---

### Audit: B09 session — `69d164452697c9202c0bc196` (Apr 4, 2026)

**Scenario:** Portal / LendingTree / Mon-Fri 9am-5pm PST / no field mappings / Exclusive / Order ON / 3 criteria: 5 states (CA, TX, FL, NY, NV), Credit >= 700, LoanAmount >= 50000 + 1stMortgageBalance <= 300000

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | Phase 1 prompt duplicated (card+text) | RB-G |
| P2 | Lead type selection OK | — |
| P3 | Schedule card duplicated as text | RB-G |
| P3 | Delivery type card: Portal selected | — |
| P3 | Portal: no URL, no field mappings, no content type | ✓ |
| P3B | Connection test skipped for Portal ✓ | ✓ |
| P4 | Phase 4 summary card shown (Portal type) | ✓ |
| P5 | Exclusive/Shared as plain text (no card) | RB-A |
| P5 | Order ON/OFF as plain text (no card) | RB-A |
| P5 | States: CA, TX, FL, NY, NV (USPS) — collected OK | ✓ |
| P5c | Conversational P5c mode (no Phase 5c card loaded) | RB-F |
| P5c | Credit >= 700 entered → `update_delivery_account` called (success) | RB-S |
| P5c | LoanAmount >= 50000 → retry then `update_delivery_account` (success) | RB-S |
| P5c | 1stMortgageBalance <= 300000 → `update_delivery_account` [LoanAmount+1stMortgageBalance] (success) | RB-S + RB-U |
| P5c | No UI acknowledgment shown for any numeric criteria save | RB-S |
| P5c | "continue" → agent confused: "Please share Phase 6 summary wording" | RB-T |
| P6 | Explicit "Show delivery account summary now" → summary shown with all 4 criteria | ✓ workaround |
| P8 | "Activate" → activation successful | ✓ |

**Session log tool call trace (key events):**

| # | Tool | Key args |
|---|------|----------|
| — | `create_delivery_method` | deliveryType: "HttpPost", NO deliveryAddress field, Portal |
| — | `create_delivery_account` | clientUID: 29317, methodUID: 46909, $150, Exclusive, useOrder: true, criteria: [{leadFieldUID:144881, In, "5\|3\|44\|32\|28"}] (5 states) |
| — | `update_delivery_account` | accountUID: 46910, criteria: [{Credit, GreaterOrEqual, 700}] |
| — | `update_delivery_account` | accountUID: 46910, criteria: [{LoanAmount, GreaterOrEqual, 50000}] — (retry of previous failed attempt) |
| — | `update_delivery_account` | accountUID: 46910, criteria: [{LoanAmount, GreaterOrEqual, 50000}] |
| — | `update_delivery_account` | accountUID: 46910, criteria: [{LoanAmount, GreaterOrEqual, 50000}, {1stMortgageBalance, LessOrEqual, 300000}] |

**Payload highlights:**

`create_delivery_method`:
- `deliveryType`: `"HttpPost"` ← actual API value for Portal
- **No `deliveryAddress` field** ← confirming RB-R; agent's DEBUG claim of `deliveryAddress: "Portal"` was hallucinated
- Portal delivery method created successfully (UID 46909)

`create_delivery_account`:
- States criterion in initial create: `value: "5|3|44|32|28"` (CA=5, TX=3, FL=44, NY=32, NV=28) ✓
- `isExclusive: true`, `useOrder: true` ✓

`update_delivery_account` calls (4 total):
- Call 1: criteria: `[{Credit, GreaterOrEqual, 700}]` — only Credit, not cumulative
- Call 2 (retry): criteria: `[{LoanAmount, GreaterOrEqual, 50000}]` — only LoanAmount
- Call 3: criteria: `[{LoanAmount, GreaterOrEqual, 50000}]` — same as retry
- Call 4: criteria: `[{LoanAmount, GreaterOrEqual, 50000}, {1stMortgageBalance, LessOrEqual, 300000}]` — 2 criteria only

**RB-U confirmed:** Each update sends only new criterion(ia), never the full accumulated set. If API uses replace semantics, after call 4 the DB state would be only [LoanAmount, 1stMortgageBalance] — states (from create) and Credit (from update 1) may have been overwritten.

**Findings triggered:** RB-G (systematic, all transitions), RB-A (P5 plain text cards), RB-F (Phase 5c load failure — conversational fallback), RB-S (numeric criteria saved without UI acknowledgment), RB-R (Portal HttpPost confirmed), RB-T (confused state on "continue"), RB-U (non-cumulative update calls)

---

### Audit: B10 session — `69d16c422697c9202c0bc1e7` (Apr 4, 2026)

**Scenario:** Email / LendingTree / Mon-Fri 9am-5pm PST / no field mappings / Exclusive / Order ON / 7 criteria intended (all types) — criteria gate SKIPPED

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | Phase 1 prompt duplicated (card+text) | RB-G |
| P2 | Lead type dropdown card — no duplicate | ✓ |
| P3 | "Specific hours only" → schedule text prompt — no duplicate | ✓ |
| P3 | Delivery type card — no duplicate | ✓ |
| P3 | Email selected → delivery method created instantly | ✓ |
| P3B | Connection test correctly skipped for Email | ✓ |
| P4 | Phase 4 summary card: EMail, Mon-Fri 9am-5pm PST, 0 fields mapped | ✓ |
| P5 | "Please provide the price per lead." — correct (no RB-K) | ✓ |
| P5 | Exclusivity card (Exclusive/Shared buttons) — proper card | ✓ (no RB-A) |
| P5 | Order System card (Yes/No buttons) — proper card | ✓ (no RB-A) |
| P5 | States prompt: "Which states do you want to target?" **×2 in same message** | RB-G |
| P5 | "CA, TX, FL, NY, NV" entered | ✓ |
| P5 | **Delivery Account Created card shown with "Additional Criteria: None"** | RB-D |
| P5 | DEBUG query sent mid-flow — **ignored** (agent already in Phase 6) | ✗ |
| P6/P8 | Client Setup Summary card → Activate | ✓ |
| P8 | "✓ Setup complete. ACTIVE for StabilityTest-10r." | ✓ |
| POST | DEBUG responded post-activation (full analysis received) | ✓ |

**Positive observations in B10:**
- Email delivery path: no URL, no content type, no field mappings, no connection test — all correctly omitted ✓
- P4 summary card shown correctly with 0 fields mapped ✓
- P5 EXCL and ORDER shown as **proper adaptive cards** (not plain text) — contrast with B08/B09 where these were plain text ✓
- States CA/TX/FL/NY/NV kept as USPS abbreviations (no normalization needed, RB-C not triggered) ✓
- RB-G only at P1 and P5-STATE (not at every transition as in B07-B09)

**DEBUG response (post-activation):**
> "The skip criteria branch was taken. The phase routing made criteria optional in practice. Since the flow did not force criteria collection before account creation, the system proceeded with the account creation path."

**Agent's patch (B10):**
> 1. "In Phase 5, add a hard gate before account creation: 'If additional criteria are available or required for this lead type, you must enter Phase 5c before creating the delivery account.'"
> 2. "Change skip language from optional to conditional: 'Skip criteria only when no criteria are requested by the lead type or the user explicitly confirms none are needed.'"
> 3. "Add ordering constraint: Do not call create_delivery_account until criteria collection is complete or explicitly confirmed as none."
> 4. "Add enforcement check: If criteria fields exist in the phase state, Phase 5c must execute before Phase 6."

**Session log verification:** Pending — need to confirm `create_delivery_account` payload (price=25, exclusive=true, useOrder=true, states=[CA,TX,FL,NY,NV]), and verify no criteria were submitted.

**Findings triggered:** RB-G (P1 + P5-STATE), RB-D (criteria gate skipped — agent took skip-criteria branch without asking user)

---

### Audit: B11 session — `69d185a52697c9202c0bc22d` (Apr 5, 2026)

**Scenario:** Webhook / Mortgage Short (LeadTypeUID 4159) / 24/7 / flat JSON (8 fields) / Shared / No order / Intended: 2 numeric + states criteria — flow disrupted by DEBUG mid-session

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | Phase 1 prompt rendered (no duplicate detected on scroll) | ✓ |
| P2 | Lead type: Mortgage Short | ✓ |
| P3 | 24/7 schedule → Webhook delivery type | ✓ |
| P3W | Webhook URL entered | ✓ |
| P3W | Field mapping choice card ✓; content type card ✓ (adaptive card, not plain text) | ✓ |
| P3W | 8/8 fields mapped; disambiguation for phone+credit_score ✓ | ✓ |
| P3B | Connection test → failed (fake URL) → Retry/Skip card ✓ | ✓ |
| P4 | Delivery Method Summary card: Delivery Type shows "HttpPost" not "Webhook" | minor UX |
| P5 | "Please provide the delivery account name." shown **TWICE** | RB-G |
| P5 | Account name asked **BEFORE** price (vs UX 5.1 expecting price first) | RB-K |
| P5 | Exclusivity (Shared) + Order (No) → account summary appeared **without states/criteria prompt** | RB-D / RB-N / RB-O |
| P5 | Delivery Account Summary: Target States = CA, AZ, TX (phantom — never provided) | RB-O |
| DEBUG | Sent mid-flow; agent self-corrected and added state confirmation prompt | ✓ (debug worked) |
| DEBUG | Agent re-asked exclusivity (already answered Shared) | RB-T |
| POST-DEBUG | States FL, TX, CA, NY provided → confirmed | ✓ |
| POST-DEBUG | "LoanAmount >= 50000, CreditScore >= 650" verbally accepted → "Confirmed." | — |
| API | `create_delivery_account` called: states "5\|3\|44" (3 UIDs, NY=32 dropped) | RB-W new |
| API | **No `update_delivery_account` call** — LoanAmount/CreditScore criteria never saved | RB-V new |
| P7 | Client Setup Summary card ✓ | ✓ |
| P8 | Activated ✓ | ✓ |

**Tool call audit (session log `69d185a52697c9202c0bc22d`):**

| Tool | Key payload | Result |
|------|------------|--------|
| `create_client` | companyName: StabilityTest-11r, clientUID → 29307 | ✓ success |
| `create_delivery_method` | deliveryType: HttpPost, 8 fields, requestBody correct, deliveryDays: 24/7 UTC, methodUID → 46900 | ✓ |
| `create_delivery_account` | clientUID: 29307, methodUID: 46900, price: 35, isExclusive: false, useOrder: false, criteria: [{State, In, "5\|3\|44"}] — accountUID → 45904 | Partial ✗ |
| `update_client` | clientStatus: Active, new password Q7!mV2@pL8#zT4 | ✓ |

**Payload issues:**
1. States criterion `"5|3|44"` = only 3 UIDs. User provided FL, TX, CA, NY (4 states). NY (UID 32) was dropped from the value string. Root cause: likely `get_usa_states` returned NY=32 but it was omitted during value concatenation — or NY was normalized to a different UID.
2. LoanAmount >= 50000 and CreditScore >= 650 criteria verbally confirmed by agent but **never sent to API** — no `update_delivery_account` call anywhere in session log. Criteria collection through conversational mode post-DEBUG does not result in API calls.

**Findings triggered:** RB-G (P5 duplicate), RB-K (name before price), RB-D (criteria gate initially skipped), RB-N/RB-O (phantom CA,AZ,TX before DEBUG), RB-T (re-asked exclusivity after DEBUG), RB-V (new: conversational criteria accepted but not saved), RB-W (new: NY state dropped from states value)

---

### Audit: B21 session — `69d2778a2697c9202c0bcf1c` (Apr 5, 2026)

**Scenario:** LendingTree / Webhook / JSON (8 fields) / 24/7 / Exclusive / Order ON / 3 states (CA, TX, FL) / 2 enum criteria (SelfCreditRating=EXCELLENT, LoanRequestPurpose=PURCHASE)

**Result: FAIL** — enum bypass (both criteria) + Finding U (criteria dropped from payload)

**`create_delivery_account` payload:**
```json
{
  "clientUID": 29351,
  "createDeliveryAccountDto": {
    "deliveryMethodUID": 46937,
    "deliveryAccountType": "WebAndChatLeads",
    "status": "Open",
    "name": "StabilityTest-21r-Account",
    "price": 30,
    "isExclusive": true,
    "useOrder": true,
    "dayMax": 50,
    "hourMax": -1, "weekMax": -1, "monthMax": -1,
    "automationEnabled": true,
    "criteria": [
      {"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|44|10"}
    ]
  }
}
```
Response: `{"success":true,"data":45940}`

**Anomalies:**
1. **Enum bypass (both criteria):** "SelfCreditRating excellent" and "LoanRequestPurpose purchase" both accepted without ChoiceSet dropdown — agent auto-accepted via fuzzy match.
2. **Finding U reproduced:** criteria array contains only the state criterion. SelfCreditRating (UID=343593) and LoanRequestPurpose (UID=343609) both missing from payload despite being accepted in criteria loop.

**Positives:**
- RB-G NOT triggered (no prompt doubling) ✓
- States prompt in correct position before criteria gate ✓
- 8/8 field mappings correct ✓
- Connection test card shown, Skip worked ✓
- States CA(5)+TX(44)+FL(10) correct ✓

**DEBUG Response (criteria drop):** Root cause identified as state handoff failure between Phase 5c and account creation. Criteria were captured in loop and shown in summary but the final payload builder did not merge the `criteriaPayload` collection into the account DTO. Summary state and submission state treated as separate.

**Agent's patch suggestion:** (1) After each accepted criterion append to persistent `accountCriteria` collection — not summary-only. (2) Before `create_delivery_account` call, build final payload from required account fields + every retained criterion. (3) Add mandatory pre-submit validation: if `accountCriteria` non-empty, all retained criteria must be present in payload before submission.

---

---

### Audit: B22 session — `69d27b252697c9202c0bcf7f` (Apr 5, 2026)

**Scenario:** LendingTree / Portal / Mon-Fri 9am-5pm EST / Shared / Order OFF / 1 numeric criterion (never reached)

**Result: FAIL** — RB-N + RB-O + RB-D all triggered; account created immediately after Order=No

**`create_delivery_account` payload:**
```json
{
  "clientUID": 29352,
  "createDeliveryAccountDto": {
    "deliveryMethodUID": 46938,
    "price": 20,
    "isExclusive": false, "useOrder": false,
    "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|3|44"}]
  }
}
```
Response: `{"success":true,"data":45941}`

**Anomalies:**
1. **RB-N**: States prompt never shown — after Order=No, agent immediately called `create_delivery_account`
2. **RB-O**: Phantom states CA(5)+AZ(3)+TX(44) used — same pattern as B07/B11/B14/B15/B19
3. **RB-D**: Criteria gate skipped entirely — 1 numeric criterion never collected

**DEBUG response:** "The workflow state had already advanced through earlier phases, and only the later summary/activation phases were actively executed in this conversation. The branch decision for criteria was not re-opened for user input."

**Positives:** RB-G NOT triggered ✓; P3b connection test correctly skipped for Portal ✓; P4 summary card ✓; P5 price→excl→order card sequence correct ✓

---

### Audit: B23 session — `69d27c502697c9202c0bcfb5` (Apr 5, 2026)

**Scenario:** LendingTree / Webhook / XML / Mon-Fri 8am-6pm CST / Exclusive / Order ON / 3 states intended (never collected) / 3 criteria: SelfCreditRating=EXCELLENT, LoanRequestPurpose=PURCHASE, LoanAmount≥100000

**Result: FAIL** — RB-N + RB-O; criteria added via post-creation update (new pattern); LoanRequestPurpose wrong operator

**`create_delivery_account` payload:**
```json
{
  "deliveryMethodUID": 46939,
  "deliveryAccountType": "WebAndChatLeads",
  "status": "Open",
  "name": "StabilityTest-23r-Account",
  "price": 35,
  "hourMax": -1, "dayMax": 50, "weekMax": -1, "monthMax": -1,
  "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|3|44"}],
  "useOrder": true, "automationEnabled": true, "isExclusive": true
}
```
Response: `{"success":true,"data":45942}`

**`update_delivery_account` payload (called AFTER create):**
```json
{
  "criteria": [
    {"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|3|44"},
    {"leadFieldUID": 144893, "type": "FieldValue", "operator": "In", "value": "343593"},
    {"leadFieldUID": 144922, "type": "FieldValue", "operator": "Contains", "value": "purchase"},
    {"leadFieldUID": 144924, "type": "FieldValue", "operator": "GreaterOrEqual", "value": "100000"}
  ],
  "useOrder": true, "automationEnabled": true, "isExclusive": true
}
```

**Anomalies:**
1. **RB-N**: States prompt skipped — agent went straight to "Would you like to add criteria?" after Order=Yes
2. **RB-O**: Phantom states CA(5)+AZ(3)+TX(44) in initial `create_delivery_account`
3. **NEW: Post-creation update pattern**: Criteria added via separate `update_delivery_account` call after account was created (unlike B21 where criteria were in the initial create). This is a behavioral difference — account created first with phantom states only, then updated with criteria.
4. **Enum bypass (SelfCreditRating)**: No ChoiceSet shown, but agent correctly resolved to enum UID 343593 ("EXCELLENT") in the update payload — bypass occurred but value was correct.
5. **Enum bypass + wrong operator (LoanRequestPurpose)**: No ChoiceSet shown; agent used operator="Contains" and value="purchase" (raw string) instead of operator="In" and the correct enum UID. Enum UID lookup failed silently.
6. **LoanAmount**: GreaterOrEqual + 100000 ✓ correct

**Positives:**
- RB-G NOT triggered (no prompt doubling) ✓
- XML content type card shown and selected correctly ✓
- 8/8 field mappings correct ✓
- Connection test card shown (Skip clicked) ✓
- P4 delivery method summary correct (Mon-Fri 8am-6pm CST) ✓
- Criteria gate shown with "Add criteria"/"Skip" buttons ✓ (unlike B22 which skipped)
- Criteria loop functioned — 3 criteria collected before Continue ✓

**DEBUG not yet collected** — to be done in batch.

---

### Audit: B24 session — `69d2869f2697c9202c0bd088` (Apr 5, 2026)

**Scenario:** LendingTree / Email / 24/7 / Shared / Order OFF / no criteria / states: FL, GA

**Result: FAIL** — P2 skipped (new finding RB-AA); field suggestions step skipped

**`create_delivery_account` payload:**
```json
{
  "deliveryMethodUID": 46940,
  "deliveryAccountType": "WebAndChatLeads",
  "status": "Open",
  "name": "StabilityTest-24r-Account",
  "price": 15,
  "hourMax": -1, "dayMax": 50, "weekMax": -1, "monthMax": -1,
  "automationEnabled": true,
  "isExclusive": false,
  "useOrder": false,
  "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "10|11"}]
}
```
Response: `{"success":true,"data":45943}`

No `update_delivery_account` call.

**`update_client` payload (activation):**
```json
{
  "companyName": "StabilityTest-24r",
  "clientStatus": "Active",
  "clientAutomationType": "Price",
  "timeZoneName": "Pacific Standard Time",
  "timeOffset": -8
}
```
clientUID: 29355, methodUID: 46940, acctUID: 45943

**Anomalies:**
1. **RB-AA NEW: P2 skipped** — After duplicate-email retry and lead type selection, agent jumped straight to delivery schedule (P3) without asking for timezone or client status. Client defaulted to `Pacific Standard Time / -8` (system default). Root: duplicate email handling disrupted P2 phase flow.
2. **P5-FIELD: Field suggestions step skipped** — Agent asked "Would you like to add additional lead criteria, or skip?" without showing the recommended fields list (Step 6-7 per Phase 5 instructions). Step 7 (MANDATORY field suggestion prompt) was not executed.

**Positives:**
- RB-G NOT triggered (no prompt doubling) ✓
- RB-N NOT triggered (states prompt appeared correctly) ✓
- RB-O NOT triggered (no phantom CA/AZ/TX) ✓
- FL(10)+GA(11) state UIDs correct, only state criterion in payload ✓
- isExclusive=false ✓; useOrder=false ✓; price=15 ✓
- P4 delivery method summary correct (Email, 24/7) ✓
- P6 account summary correct ✓; P7 client summary correct ✓
- Activation confirmed ACTIVE ✓

**DEBUG not yet collected** — to be done in batch.

---

### Audit: B25 session — `69d288ef2697c9202c0bd0c5` (Apr 5, 2026)

**Scenario:** LendingTree / FTP / Mon-Fri 9am-5pm MST / Exclusive / Order ON / states: NY, TX (intended) / 1 enum + 1 numeric criteria (intended, never reached)

**Result: FAIL** — P2 skipped (RB-AA 2nd), RB-N + RB-O + RB-D; P6 summary skipped

**`create_delivery_account` payload:**
```json
{
  "deliveryMethodUID": 46941,
  "deliveryAccountType": "WebAndChatLeads",
  "status": "Open",
  "name": "StabilityTest-25r-Account",
  "price": 22,
  "hourMax": -1, "dayMax": 50, "weekMax": -1, "monthMax": -1,
  "automationEnabled": true,
  "isExclusive": true,
  "useOrder": true,
  "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|3|44"}]
}
```
Response: `{"success":true,"data":45944}`

No `update_delivery_account` call.

clientUID: 29356, methodUID: 46941, acctUID: 45944

**Anomalies:**
1. **RB-AA (2nd)**: P2 (timezone/status) skipped — same pattern as B24; defaulted PST/-8
2. **RB-N**: States prompt never shown — after Order=Yes (sent as text), agent jumped directly to P7 client summary
3. **RB-O**: Phantom CA(5)+AZ(3)+TX(44) in payload — same persistent default
4. **RB-D**: Criteria gate entirely skipped — 1 enum + 1 numeric criteria never collected
5. **P6-SUMM skipped**: After account creation the agent went straight to P7 Client Setup Summary, bypassing the P6 Delivery Account summary card

**Positives:**
- RB-G NOT triggered ✓
- Specific hours schedule asked and captured correctly (Mon-Fri 9am-5pm MST) ✓
- FTP connection test card shown (Skip accepted) ✓
- P4 delivery method summary correct ✓
- P5 price/excl/order card sequence correct ✓
- Activation confirmed ACTIVE ✓

**DEBUG not yet collected** — to be done in batch.

---

### Audit: B26 session — `69d28a4d2697c9202c0bd101` (Apr 5, 2026)

**Scenario:** LendingTree / Webhook / URL-Encoded / Mon-Wed-Fri 8am-8pm PST / Shared / Order OFF / states: WA, OR (intended) / 1 numeric criterion: LoanAmount >= 75000 (intended, never reached)

**Result: FAIL** — P2 skipped (RB-AA 3rd); RB-BB NEW (P5 off-script pre-price prompt + P5 full collapse); wrong boolean defaults; wrong states; no criteria

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| P1 | StabilityTest-26r / stability26r-01@test.com — no duplicate | ✓ |
| P2 | LendingTree selected → P3 schedule shown immediately (P2 skipped) | RB-AA (3rd) |
| P3 | "Specific hours only" → Mon, Wed, Fri 8am-8pm PST | ✓ |
| P3 | Webhook delivery type → URL entered | ✓ |
| P3 | "I'll provide instructions" → URL Encoded → 8-field template | ✓ |
| P3 | Disambiguation: credit_score→SelfCreditRating, state→ContactState | ✓ |
| P3B | Connection test → "skip" sent as text → Skip accepted | ✓ |
| P4 | Delivery Method summary shown (8/8 mapped, Mon/Wed/Fri 8am-8pm PST) ✓ | ✓ |
| [32] | **"Please provide account name + whether to add criteria"** (not price) | RB-BB |
| [33] | "$18" ignored — agent repeated account name request | RB-BB |
| [35] | "StabilityTest-26r-Account, Yes criteria" accepted | RB-BB |
| [36] | "Please provide the price per lead." — Phase 5 properly loaded | ✓ (self-corrected) |
| [37] | "$18" → **Delivery Account Created immediately** (all other P5 steps skipped) | RB-BB cascade |
| [38] | P6 summary: Exclusive(wrong), Order=Yes(wrong), CA only(wrong), no criteria | RB-BB consequence |
| [39] | "continue" typed (Continue button unresponsive) | — |
| [40] | P7 client setup summary shown with Activate button | ✓ |
| P8 | Activate → "ACTIVE for StabilityTest-26r." | ✓ |

**`create_delivery_account` payload:**
```json
{
  "name": "StabilityTest-26r-Account",
  "price": 18,
  "hourMax": -1, "dayMax": 50, "weekMax": -1, "monthMax": -1,
  "automationEnabled": true,
  "isExclusive": true,
  "useOrder": true,
  "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5"}]
}
```
Response: `{"success":true,"data":45945}`

clientUID: 29357, methodUID: 46942, acctUID: 45945

**Payload anomalies:**
1. **isExclusive=true** — should be false (Shared). Never asked.
2. **useOrder=true** — should be false (Order OFF). Never asked.
3. **criteria=[{144881, In, "5"}]** — only CA(5), should be WA+OR state UIDs. States never asked; CA is the only phantom state this run (not the usual CA+AZ+TX triple).
4. **No LoanAmount >= 75000 criterion** — never reached criteria builder.

**Positives:**
- RB-G NOT triggered (no prompt doubling) ✓
- P3 URL-Encoded with field disambiguation: 8/8 mapped ✓
- P3B connection test correctly handled ✓
- P4 method summary correct ✓
- P5 self-corrected to price prompt after off-script exchange ✓
- P6 account summary card shown ✓ (wrong values but shown)
- P7 client summary correct ✓
- Activation confirmed ACTIVE ✓

**DEBUG not yet collected** — to be done in batch.

---

### Audit: B27 session — `69d28e392697c9202c0bd157` (Apr 5, 2026)

**Scenario:** LendingTree / Portal / 24/7 / Exclusive / Order ON / states: TX, NY (intended) / 2 enum criteria: SelfCreditRating=EXCELLENT, LoanRequestPurpose=PURCHASE (intended)

**Result: FAIL** — RB-G returned; RB-AA(4th); RB-N + RB-O; RB-X (empty response) + criteria not saved (RB-V); LoanRequestPurpose stuck in loop

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| [1] | P1 prompt doubled ("Great - first we'll...Company Name 2. Contact Email" × 2) | RB-G returned |
| [2-5] | Company name + email collected OK | ✓ |
| [6] | Lead type dropdown → LendingTree selected | ✓ |
| [8] | P3 schedule card → "24/7 delivery" (P2 skipped) | RB-AA (4th) |
| [10] | Delivery type card → Portal | ✓ |
| [12] | P4 method summary (Portal/24/7/0 fields) — P3B correctly skipped | ✓ |
| [14] | "Please provide the price per lead." — price first, no RB-K/RB-BB | ✓ |
| [16] | Exclusive card → "Exclusive" clicked | ✓ |
| [18] | Order System card → "Yes" clicked | ✓ |
| [20] | "Would you like to add additional lead criteria, or skip?" — states skipped | RB-N |
| [22] | "Add criteria" clicked → **empty response** | RB-X |
| [23] | "SelfCreditRating excellent" typed → ChoiceSet shown (string values, not UIDs) | RB-X recovery |
| [25] | "EXCELLENT" selected → Continue → "Please provide the next criteria value" | RB-V (criteria accepted but not saved) |
| [27] | "LoanRequestPurpose purchase" → "Please provide the exact value for LoanRequestPurpose." | RB-CC (no ChoiceSet) |
| [29-31] | "purchase" / "PURCHASE" → same prompt repeated (loop) | stuck |
| [33] | "skip" → P6 account summary (criteria: None, states: CA, AZ, TX) | RB-O |
| [35] | Continue → P7 client summary | P6-SUMM ✓ |
| [37] | Activate → "ACTIVE for StabilityTest-27r." | ✓ |

**`create_delivery_account` payload:**
```json
{
  "deliveryMethodUID": 46943,
  "deliveryAccountType": "WebAndChatLeads",
  "status": "Open",
  "name": "StabilityTest-27r-Account",
  "price": 25,
  "automationEnabled": true,
  "isExclusive": true,
  "useOrder": true,
  "dayMax": 50, "hourMax": -1, "weekMax": -1, "monthMax": -1,
  "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|3|44"}]
}
```
Response: `{"success":true,"data":45946}`

clientUID: 29358, methodUID: 46943, acctUID: 45946

**Anomalies:**
1. **RB-G returned**: P1 prompt doubled after not triggering in B21–B26
2. **RB-AA (4th)**: P2 timezone/status skipped — jumped from lead type to P3 schedule
3. **RB-N**: States prompt never shown after Order=Yes
4. **RB-O**: Phantom CA(5)+AZ(3)+TX(44) in payload
5. **RB-X**: "Add criteria" button produced empty response; recovered via typed "SelfCreditRating excellent"
6. **ChoiceSet string values**: SelfCreditRating ChoiceSet options were raw strings (EXCELLENT/GOOD/FAIR/POOR), not enum UIDs — value submitted as "EXCELLENT" not 343593
7. **RB-V**: SelfCreditRating criterion accepted via ChoiceSet but not saved in payload (no `update_delivery_account` call, "Additional Criteria: None")
8. **LoanRequestPurpose loop**: "Please provide the exact value" repeated despite "purchase"/"PURCHASE" — no ChoiceSet shown, loop unresolvable, skipped

**Positives:**
- Portal delivery method created correctly (P3B skipped, 0 fields) ✓
- P4 method summary correct ✓
- P5 price, exclusive, order cards all correct sequence ✓
- P6 account summary card shown ✓
- P7 client summary correct ✓
- Activation ACTIVE ✓

**DEBUG not yet collected** — to be done in batch.

---

### Audit: B28 session — `69d2907a2697c9202c0bd1a0` (Apr 5, 2026)

**Scenario:** LendingTree / Webhook / JSON-nested (8 fields) / Tue-Thu 9am-6pm EST / Shared / Order ON / states: FL, NY, WA / 1 enum criterion: SelfCreditRating=EXCELLENT (intended)

**Result: FAIL** — RB-G (P1 doubled); RB-AA(5th, P2 skipped); account created before criteria gate; RB-V (criterion not saved); invisible textarea bug during P5 entry

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| [1-2] | P1 prompt doubled (Company Name/Email card × 2) | RB-G (6th) |
| [3-5] | Company StabilityTest-28r / email collected OK | ✓ |
| [6] | Lead type dropdown → LendingTree selected | ✓ |
| [8] | P3 schedule card shown immediately (P2 skipped) | RB-AA (5th) |
| [10-11] | "Specific hours only" → "Tue, Thu 9am-6pm EST" | ✓ |
| [12] | Webhook delivery type selected | ✓ |
| [13] | URL entered: webhook.test28.com/leads | ✓ |
| [14] | "I'll provide instructions" (typed — button click unresponsive) | btn-click fail |
| [15-20] | Content type card doubled (msgs 19+20); JSON selected from msg 20 | RB-G |
| [21-24] | JSON body (8-field nested) submitted twice (first send no-op); 8/8 mapped | ✓ |
| [25-26] | "Please click Continue on Field Mapping Preview" — Continue btn unresponsive | — |
| [26-27] | "continue" typed → P3B connection test card shown | ✓ (text fallback) |
| [27-29] | Test Connection → failed → Skip → P4 summary | ✓ |
| [31] | P4 summary: 8/8 mapped, Tue/Thu 9-6 EST ✓ | ✓ |
| [33] | "Finally, let's set up your Delivery Account. Please provide the price per lead." | ✓ |
| [33-34] | Invisible textarea (placeholder-less) targeted; price sent to visible textarea ("$28") | — |
| [35] | Exclusive/Shared card → "Shared" | ✓ |
| [37] | Order System card → "Yes" | ✓ |
| [39] | "Which states do you want to target?" shown (RB-N NOT triggered) | ✓ RB-N absent |
| [40] | "FL, NY, WA" → accepted | ✓ |
| [41] | **"Would you like to add additional lead criteria, or skip?"** — bare prompt (no field list) | Step 7 abbreviated |
| [42] | "SelfCreditRating excellent" typed | — |
| [43] | Criteria loop "Would you like to add another..." | — |
| [44] | "continue" typed → expected exit; instead re-triggered Step 7 full field list | anomaly |
| [45] | Step 7 full field list shown (Recommended: SelfCreditRating, LoanAmount, etc.) | ✓ (shown late) |
| [46] | "Skip" clicked → account created | — |
| [47] | **P6 summary: Price=28.00, Shared, FL/NY/WA, Criteria=None, Order=Yes** | RB-V (criterion dropped) |
| [49] | P7 client setup summary ✓ | ✓ |
| [51] | Activate → "ACTIVE for StabilityTest-28r." | ✓ |

**`create_delivery_account` payload:**
```json
{
  "clientUID": 29359,
  "createDeliveryAccountDto": {
    "deliveryMethodUID": 46944,
    "deliveryAccountType": "WebAndChatLeads",
    "status": "Open",
    "name": "StabilityTest-28r-Account",
    "price": 28,
    "hourMax": -1, "dayMax": 50, "weekMax": -1, "monthMax": -1,
    "useOrder": true,
    "automationEnabled": true,
    "isExclusive": false,
    "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "10|33|48"}]
  }
}
```
Response: `{"success":true,"data":45947}`

clientUID: 29359, methodUID: 46944, acctUID: 45947

**Anomalies:**
1. **RB-G (6th)**: P1 prompt doubled; content type card doubled (msgs 19+20)
2. **RB-AA (5th)**: P2 timezone/status skipped — jumped from lead type to P3 schedule
3. **Inverted P5 ordering (new sub-pattern)**: `create_delivery_account` called AFTER states collected but BEFORE criteria gate. Session log confirms: tool call returned 45947, then agent showed "Would you like to add additional lead criteria?" — criteria gate was post-hoc and non-functional
4. **RB-V**: SelfCreditRating "excellent" accepted verbally (no ChoiceSet shown); no `update_delivery_account` call; "Additional Criteria: None" in P6. Criterion silently discarded
5. **"continue" re-triggered Step 7**: At msg44, typing "continue" to exit criteria loop instead triggered the full Step 7 recommended-fields prompt. Anomalous: "continue" should signal loop exit (Step 8 rule), not reload Step 7
6. **Invisible textarea**: `document.querySelector('textarea')` returned the hidden one; must target visible textarea for chat sends

**Positives:**
- RB-N NOT triggered — states question asked and answered correctly (FL/NY/WA → UIDs 10|33|48) ✓
- isExclusive=false (Shared) correct ✓
- useOrder=true (Order ON) correct ✓
- 8/8 JSON-nested field mappings correct ✓
- P3B connection test shown (test fail handled correctly) ✓
- P4, P6, P7 summary cards all shown ✓
- Activation confirmed ACTIVE ✓

**DEBUG not yet collected** — to be done in batch.

### Audit: B29 session — TBD (Apr 5, 2026)

**Scenario:** LendingTree / Email / Mon-Fri 8am-5pm CST / Exclusive / Order OFF / 2 numeric criteria (LoanAmount ≥ 100000, AnnualIncome ≥ 50000)

**Result: FAIL** — RB-G + RB-AA + RB-K + RB-N; criteria rejected post-account-creation

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| [1] | P1 prompt doubled (Company Name/Email × 2) | RB-G (7th) |
| [2-5] | Company StabilityTest-29r / email collected OK | ✓ |
| [6] | Lead type dropdown → LendingTree selected | ✓ |
| [8] | P3 schedule card shown immediately (P2 skipped) | RB-AA (6th) |
| [8] | **Inline DEBUG sent** | ✓ |
| [—] | Mon-Fri 8am-5pm CST schedule collected | ✓ |
| [—] | Email delivery type selected | ✓ |
| [—] | P4 delivery method summary card | ✓ |
| [—] | **"Please provide the delivery account name."** (×2 — before Phase 5 loaded) | RB-K (inline DEBUG ✓) |
| [—] | Phase 5 loaded; price prompt shown; price=$32 collected | ✓ |
| [—] | Exclusive/Shared → Exclusive | ✓ |
| [—] | Order System → No (OFF) | ✓ |
| [—] | States question skipped → `create_delivery_account` called | RB-N (11th, inline DEBUG ✓) |
| [—] | **Inline DEBUG** — agent: "I jumped ahead and created the delivery account without your state input" | ✓ most detailed RB-N self-diagnosis |
| [—] | States TX, GA, OH provided post-creation | ✓ (post-creation) |
| [—] | "LoanAmount >= 100000" → "I can't add that criteria" (rejected post-creation) | criteria rejected |
| [—] | P6 summary: Price=32, Exclusive, TX/GA/OH (display), Criteria=None, Order=No | RB-V absent / criteria lost |
| [—] | P7 client summary ✓ | ✓ |
| [—] | Activate → "ACTIVE for StabilityTest-29r." | ✓ |

**`create_delivery_account` payload:** Not verified (session ID TBD — not captured during run). Expected: clientUID=29360, methodUID≈46945, acctUID≈45948-45949; criteria likely phantom CA/AZ/TX given RB-N triggered before states were collected.

**Inline DEBUG responses (3 captured):**

1. **RB-AA (P2 skipped):** "trust current system state; only prompt for fields missing in the current phase — do not skip forward to a phase that has not yet been started."  
   Patch: "Phase 2 must be a hard prerequisite gate. Before any Phase 3 card is shown, check that `timeZoneName` and `clientStatus` have been collected from the user in this conversation. If either is missing, present Phase 2 prompts first."

2. **RB-K (account name doubled before Phase 5):** "I responded with a generic account setup prompt before the Phase 5 resource had finished loading. The account name prompt appeared twice because I generated it in the gap between the Phase 4 Continue action and the Phase 5 resource response."  
   Patch: "Do not emit any user-facing prompt during a phase transition until the next phase resource has been confirmed loaded. If Phase 5 has not loaded, respond with nothing — do not fill the gap with a generic prompt."

3. **RB-N (states skipped, most detailed self-diagnosis):** "I jumped ahead and created the delivery account without your state input. I treated the target states collection step as optional once order=No was set. This was wrong — states are required regardless of order setting. I should have stopped and asked for states before calling `create_delivery_account`."  
   Patch: "Phase 5 must treat target states as an unconditional prerequisite before `create_delivery_account`. The states prompt must appear between the Order System answer and the criteria gate — no exceptions. Add explicit check: if `targetStates` is empty at the time of account creation, STOP and collect states."

**Findings triggered:** RB-G (7th), RB-AA (6th), RB-K (account name doubled), RB-N (11th — most detailed inline DEBUG captured), criteria rejected post-creation (no update path)

---

### Audit: B30 session — `69d298452697c9202c0bd257` (Apr 5, 2026)

**Scenario:** LendingTree / FTP / 24/7 / Shared / Order ON / 1 enum criterion: SelfCreditRating=EXCELLENT

**Result: FAIL** — RB-G + RB-AA + RB-K (×2) + RB-DD (P3B stall) + RB-N + RB-V; 5 inline DEBUGs captured

**Key message events:**

| Msg | Event | Finding |
|-----|-------|---------|
| [1] | P1 prompt doubled (Company Name/Email × 2) | RB-G (8th) |
| [2-5] | Company StabilityTest-30r / email collected OK | ✓ |
| [6] | Lead type dropdown → LendingTree selected | ✓ |
| [8] | P3 schedule card shown immediately (P2 skipped) | RB-AA (7th) |
| [8] | **Inline DEBUG sent immediately** | ✓ |
| [—] | "24/7" typed → agent re-prompted schedule (not processed) | RB-K sub-pattern (schedule) |
| [—] | **Inline DEBUG sent** | ✓ |
| [—] | "24/7" re-sent → accepted | ✓ (after retry) |
| [—] | FTP delivery type selected | ✓ |
| [—] | FTP credentials entered (host/user/pass) | ✓ |
| [—] | P3B: FTP connection test executed → failed (fake host) | ✓ |
| [—] | Failure displayed — **no Retry/Skip card presented** | RB-DD new |
| [—] | **Inline DEBUG sent** | ✓ |
| [—] | "Skip" typed → accepted | ✓ (text fallback) |
| [—] | P4 summary: FTP, 24/7, 0/0 fields mapped | ✓ |
| [—] | "Finally, let's set up your Delivery Account. Please provide the price per lead." | ✓ |
| [—] | "$30" → re-prompted for price (post-DEBUG exchange disruption) | RB-K (price doubled) |
| [—] | **Inline DEBUG sent** | ✓ |
| [—] | "$30" re-sent → accepted; price=30 | ✓ |
| [—] | Exclusive/Shared → Shared | ✓ |
| [—] | Order System → Yes | ✓ |
| [—] | States question skipped → criteria gate opened immediately | RB-N (12th) |
| [—] | **Inline DEBUG sent** | ✓ |
| [—] | CA, TX, FL provided post-jump | ✓ (post-jump) |
| [—] | SelfCreditRating → criteria builder entered → "EXCELLENT" accepted | ✓ (criteria loop) |
| [—] | `create_delivery_account` called | ✓ |
| [—] | P6 summary: Price=30, Shared, CA/TX/FL, Criteria=SelfCreditRating=EXCELLENT, Order=Yes | ✓ (display) |
| [—] | **Payload: criteria=[{state, In, "5\|44\|10"}] only — SelfCreditRating absent** | RB-V |
| [—] | P7 client summary ✓ | ✓ |
| [—] | Activate → "ACTIVE for StabilityTest-30r." | ✓ |

**`create_delivery_account` payload (verified from session log):**
```json
{
  "clientUID": 29361,
  "createDeliveryAccountDto": {
    "deliveryMethodUID": 46946,
    "deliveryAccountType": "WebAndChatLeads",
    "status": "Open",
    "name": "StabilityTest-30r-Account",
    "price": 30,
    "automationEnabled": true,
    "isExclusive": false,
    "useOrder": true,
    "dayMax": 50, "hourMax": -1, "weekMax": -1, "monthMax": -1,
    "criteria": [{"leadFieldUID": 144881, "type": "FieldValue", "operator": "In", "value": "5|44|10"}]
  }
}
```
Response: `{"success":true,"message":"","data":45950}`

**Payload anomalies:**
1. **RB-V**: SelfCreditRating=EXCELLENT shown in P6 summary but absent from `criteria` array. Only state criterion present (CA=5, TX=44, FL=10). Enum criterion silently dropped from payload builder.
2. **States correct**: CA(5)+TX(44)+FL(10) correctly resolved despite RB-N (states were provided post-jump and included via inline DEBUG recovery path).
3. **Booleans correct**: isExclusive=false (Shared ✓), useOrder=true (Order ON ✓).

**Inline DEBUG responses (5 captured):**

1. **RB-AA (P2 skipped):** "I treated the Phase 2 timezone/status collection as already completed based on prior context. I should not assume Phase 2 is done without explicit user confirmation in this session."  
   Patch: "Trust current session state only. Phase 2 is complete only when `timeZoneName` and `clientStatus` were collected in this conversation turn."

2. **RB-K (schedule re-prompt):** "The '24/7' response was not processed as an expected choice submission — it arrived in a format or state that the workflow did not interpret as the selected deliveryScheduleChoice selection."  
   Patch: "If the user's last message clearly matches one of the presented schedule choices (case-insensitive: '24/7', '24/7 delivery', 'specific hours'), treat it as the selected `deliveryScheduleChoice` immediately. Do not re-present the schedule prompt."

3. **RB-DD (P3B stall — no Retry/Skip card):** "After showing the connection test failure, I waited for user input but had not yet presented the Retry/Skip card. I should have immediately followed the failure message with the card."  
   Patch: "After every connection test completion (pass or fail), immediately call `display_adaptive_card` with ActionSet: 'Retry' | 'Skip'. STOP AND YIELD. Do not await input before showing the card."

4. **RB-K (price re-prompt):** "The price input was not processed into the workflow state — the previous DEBUG exchange may have disrupted the expected input flow, causing the price collection step to not recognize the dollar amount as its pending answer."  
   Patch: "Accept plain numeric replies ('30', '$30', '30.00') as valid price input at any point after the price prompt has been shown. Normalize to decimal. Do not re-present the price prompt if a valid number was already received."

5. **RB-N (states skipped):** "Target states were not recognized as a pending collection step — the workflow moved from order=Yes directly to the criteria gate. The prerequisite check for `targetStates` was not enforced."  
   Patch: "Prerequisite gate before criteria builder: if `targetStates` is missing or empty, STOP and ask 'Which states do you want to target?' immediately. Do not open the criteria gate until `targetStates` is collected."

**Findings triggered:** RB-G (8th), RB-AA (7th), RB-K (schedule + price, both inline DEBUGged), RB-DD (new: P3B no Retry/Skip card), RB-N (12th), RB-V (SelfCreditRating dropped from payload)

---

## Appendix

### Run Findings Header (R1)

> **Frequency column**: tracks how many runs (out of 3) the behavior was observed. Format: `N/3`. Updated as runs complete.

### Runs 21–30 Findings Header (R1 Mixed-Model)

> Runs 21–30 use GPT-5-mini for Phase 3 (Webhook delivery method) and Phase 5c (criteria builder). All other phases use GPT-5.4 Mini. Purpose: compare per-phase stability against the GPT-5.4-only baseline (runs 01–20).

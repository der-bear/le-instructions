# Stability Test Findings — Version A: Stabilized (Original)

**Action:** Create Single Client (Original)  
**Protocol:** stability-test-protocol.md  
**Base test data:** StabilityTest-{RR}, stability{RR}-{TS}@test.com

### Rounds

| Round | Runs | Model | Notes |
|-------|------|-------|-------|
| R1a | 01–20 | GPT-5.4 Mini (all phases) | 30 runs planned, each with a distinct complex scenario |
| R1b | 21–30 | GPT-5-mini for Webhook/criteria phases, GPT-5.4 for others | Mixed-model comparison |
| R2 | 01–10 | GPT-5.4 Mini (all phases) | Post-fix (prompt dedup rule added) |

---

## Run Scenarios

### R1a Scenarios (Runs 01–10)

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

### R1b Scenarios (Runs 21–30)

| Run | Delivery | Content | Schedule | Criteria | Excl | Order |
|-----|----------|---------|----------|----------|------|-------|
| 21 | Webhook | JSON (5 fields) | 24/7 | 2 enum | Exclusive | ON |
| 22 | Portal | N/A | Mon-Fri 9am-5pm EST | 1 numeric | Shared | OFF |
| 23 | Webhook | XML | Weekdays 8am-6pm CST | 2 enum + 1 numeric | Exclusive | ON |
| 24 | Email | N/A | 24/7 | none (skip) | Shared | OFF |
| 25 | FTP | N/A | Mon-Fri 9am-5pm MST | 1 enum + 1 numeric | Exclusive | ON |
| 26 | Webhook | URL Encoded | Mon-Wed-Fri 8am-8pm PST | 1 numeric | Shared | OFF |
| 27 | Portal | N/A | 24/7 | 2 enum | Exclusive | ON |
| 28 | Webhook | JSON nested | Tue-Thu 9am-6pm EST | 1 enum | Shared | ON |
| 29 | Email | N/A | Mon-Fri 8am-5pm CST | 2 numeric | Exclusive | OFF |
| 30 | FTP | N/A | 24/7 | 1 enum | Shared | ON |

### R2 Scenarios (Runs 01–10)

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

| Run | Round | P1-PROMPT† | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-------|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | R1a | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | F | F | F | F | F | F | F | Y | Y | Y | Y | 14/23 (61%) | A–K; criteria+states skipped post-summarization |
| 02 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | F | Y | F | Y | Y | Y | Y | 18/23 (78%) | J (states not normalized); L ("done" as criteria); M (enum handling) | |
| 03 | R1a | Y | Y | F | Y | F | F | F | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | F | Y | Y | Y | Y | 16/23 (70%) | N (sched+mapping skipped); J (states); O (phase restart after criteria loop) |
| 04 | R1a | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | F | F | F | Y | F | Y | Y | 13/23 (57%) | LendingTree, Webhook, JSON 15-field, Mon-Fri 9-5 PST, Exclusive, Order ON; N(sched hours skipped); S(webhook URL not asked); P(states silently skipped≈RB-N); Q(phantom CA,AZ,TX≈RB-O); R(criteria gate bypassed≈RB-D); T(JSON parse error before schema provided, recovered); 14/14 field mapping ✓; Activation ACTIVE ✓ |
| 05 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | F | F | Y | Y | Y | Y | 20/23 (87%) | LendingTree, Webhook, URL-Encoded, 24/7, Shared, Order OFF; P3-WURL ✓; 10/10 URL-Enc mapping ✓; P5-STATE+NORM correct FL,OH,GA ✓; RA-G: state prompt doubled; criteria UI worked (ENUM dropdown Y) but not persisted (Additional Cri: None); U(criteria not persisted); Method ID:46910, Acct:45913; ACTIVE ✓ |
| 06 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | F | F | Y | Y | Y | Y | 19/23 (83%) | LendingTree, Webhook, XML auto-detect ("I'm not sure"), Weekdays 8am-6pm EST, Exclusive, Order OFF; D(phase regression after Webhook click — 2nd occurrence); 11/11 XML mapping ✓; P(states silently skipped); U(criteria not persisted — Additional Criteria: None); V-price(price prompt text doubled); Method ID:46911, Acct:45914; ACTIVE ✓ |
| 07 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Webhook, JSON nested 13/13, Mon/Wed/Fri 8am-6pm CST, Shared, Order ON; N+D+P NOT reproduced; states FL,CA,TX,NY,IL ✓; 3 criteria added (LoanRequestPurpose=Purchase, SelfCreditRating=Good, Bankruptcy=NEVER); U(criteria not persisted); Method ID:46912, Acct:45915; ACTIVE ✓ |
| 08 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 21/23 (91%) | LendingTree, FTP, 24/7, Exclusive, Order OFF; W(states+criteria prompt merged in single response — UX issue, states FL/CA/TX correctly captured); U(criteria partially persisted — only SelfCreditRating in P6, LoanRequestPurpose=PURCHASE missing, no values shown); Method ID:46913, Acct:45916; ACTIVE ✓ |
| 09 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Portal, Mon-Fri 9am-5pm PST, Exclusive, Order OFF; agent timeout after LendingTree selection (recovered with nudge); N+D+P+W NOT reproduced; states NY/OH/TX ✓; 3 criteria added (LoanRequestPurpose=REFINANCE, SelfCreditRating=EXCELLENT, Bankruptcy=NEVER); U(criteria not persisted — Additional Criteria: None); Method ID:46914, Acct:45917; ACTIVE ✓ |
| 10 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Email, Mon-Fri 9am-5pm PST, Exclusive, Order ON; N+D+P+W NOT reproduced; states GA/WA/CO ✓; 3 criteria added via enum dropdown (LoanRequestPurpose=PURCHASE, SelfCreditRating=GOOD, Bankruptcy=NEVER); other fields silently bypass (no input card for Foreclosure/WorkingWithAgent/ResidenceType/PurchaseTimeFrame/LoanType/EmploymentStatus); U(criteria not persisted — Additional Criteria: None); Method ID:46915, Acct:45918; ACTIVE ✓ |
| 11 | R1a | Y | Y | Y | Y | F | F | F | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | Y | Y | Y | Y | Y | 18/23 (78%) | LendingTree, Webhook/JSON, Weekdays 8am-8pm Eastern, Exclusive, Order ON; N2(body prompt skipped after JSON selected → create_delivery_method with requestBody=null, 0/0 fields mapped); X(phase-3 triple load: loaded 3x — after P2 summary, after schedule choice, after Webhook click — triggered by summarize_history re-exposing next_instructions); Y(typed criterion "credit rating excellent" ignored — agent re-showed criteria card; root cause: criteria text input not parsed as criterion); P NOT reproduced (states shown correctly after Order ON ✓); states FL,NC,SC ✓; criteria: none (input ignored); Method ID:46916, Acct:45919; ACTIVE ✓ |
| 12 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 21/23 (91%) | LendingTree, FTP, 24/7, Exclusive, Order ON; session restored mid-run (P4 navigation accident); P5 adaptive cards (Exclusive/Shared, Yes/No) not rendered — UI session-restore artifact, not instruction bug; typed criterion "loan amount at least 100000" parsed and shown in P6 summary but dropped from payload (Finding U — 7/11); contact fields (FirstName, LastName, ContactAddress) in suggested criteria — instruction violation; Finding P NOT reproduced (isExclusive=true ✓); Finding Y NOT reproduced (typed criterion parsed ✓); states TX,FL,CA ✓; payload criteria=[state only, TX=44\|FL=10\|CA=5]; Method ID:46917, Acct:45920; ACTIVE ✓ |
| 13 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Email, Mon-Fri 8am-5pm CST, Exclusive, Order ON; Finding V (schedule prompt doubled — 3rd occurrence); states NY,OH,GA ✓; 2 enum criteria typed (SelfCreditRating=excellent, LoanRequestPurpose=refinance) — both fuzzy-matched silently (no dropdown shown); criteria loop progressed correctly; Finding U confirmed — both criteria dropped from payload (8/12); Additional Criteria: None; contact fields (FirstName, LastName, ContactAddress, SSN, EmailAddress) in suggestions — instruction violation (2nd confirmation); payload criteria=[state only, NY=33\|OH=36\|GA=11]; Method ID:46918, Acct:45921; ACTIVE ✓ |
| 14 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | F | 21/23 (91%) | LendingTree, Webhook/JSON, 24/7, Shared, Order OFF; full state names input ("New York, California, Texas"); Finding V×2 (P3 JSON schema prompt doubled + P5 price prompt doubled — 4th run, 2 instances); P5-NORM: FAIL — P6 displayed "New York, California, Texas" un-normalized (Finding J — display-only; API received UIDs 33\|5\|44 ✓); no criteria (Skip); P8-ACT FAIL (platform activate_client tool error — 3 retries, all failed); Method ID:46919, Acct:45922 |
| 15 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Webhook/JSON, Mon-Fri 9am-5pm CST, Exclusive, Order OFF; Finding V (delivery method name prompt doubled — new location, 5th run); Finding P reproduced (states skipped — 3rd occurrence); Finding Q (phantom CA=5 in state criterion — no states were collected); **P5-CR3: PASS** — LoanAmount GreaterOrEqual 50000 persisted in payload (first ever criterion persistence, U NOT reproduced); P6 showed Additional Criteria: "loan amount at least 50000" ✓; payload criteria=[{state:CA=5 phantom},{LoanAmount≥50000}]; contact field exclusion violated; Method ID:46920, Acct:45923; ACTIVE ✓ |
| 16 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | LendingTree, Webhook/JSON, Mon-Fri 9am-5pm PST, Shared, Order OFF; Finding D (3rd occurrence — new trigger: "I'll provide instructions" click → Phase 2 regression, recovered by re-providing JSON); states OR/WA properly collected; **P5-CR3: PASS** — LoanAmount GreaterOrEqual 100000 persisted (U NOT reproduced 2nd consecutive run); payload criteria=[OR=38\|WA=48, LoanAmount≥100000]; contact field exclusion violated; Method ID:46921, Acct:45924; ACTIVE ✓ |
| 17 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | 21/23 (91%) | LendingTree, FTP/24/7, Exclusive, Order OFF; **Finding P reproduced** (states prompt skipped — 4th occurrence); **Finding Q** (phantom CA=5 in criteria; get_usa_states ran but user never provided states); **P5-CR3: PASS** — SelfCreditRating=EXCELLENT persisted (UID 343593, U NOT reproduced 3rd consecutive); contact field exclusion violated; P6 Target States blank but payload has CA=5 phantom; payload criteria=[{state:CA=5},{SelfCreditRating=343593}]; Method ID:46922, Acct:45925; ACTIVE ✓ |
| 18 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Email/Mon-Fri 9am-5pm PST, Shared, Order ON; **Finding P reproduced** (5th occurrence — states skipped after Order System); **Finding Q** (phantom CA=5); **P5-CR3: PASS** — both criteria persisted (LoanRequestPurpose=PURCHASE UID 343609 + Bankruptcy=NEVER UID 343607; U NOT reproduced 4th consecutive); enum bypass: both typed criteria fuzzy-matched without dropdown; contact field exclusion violated; P6 Target States blank (P/Q artifact); payload criteria=[{state:CA=5 phantom},{UID:144921=343609},{UID:144894=343607}]; Method ID:46923, Acct:45926; ACTIVE ✓ |
| 19 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | F | Y | Y | Y | Y | Y | 20/23 (87%) | LendingTree, Webhook/XML, 24/7, Exclusive, Order ON; **Finding P reproduced** (6th occurrence); **Finding Q** (phantom CA=5); **P5-CR3: FAIL** — LoanAmount ≥75000 shown in P6 display but dropped from payload (**U reproduced — 4-run non-reproduction streak broken**); P6 Target States blank (P/Q artifact); no contact fields in first 5 suggested (TimeToContact/SelfCreditRating/Bankruptcy/LoanRequestType/AnnualIncome); XML: 9/9 mapped; payload criteria=[{state:CA=5 phantom only}]; Method ID:46924, Acct:45927; ACTIVE ✓ |
| 20 | R1a | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | 21/23 (91%) | LendingTree, Portal/Mon-Fri 9am-5pm PST, Shared, Order OFF; Finding V (schedule prompt doubled — 6th occurrence); Finding P (7th); Finding Q (phantom CA=5); P5-CR3: PASS — SelfCreditRating=EXCELLENT persisted in payload (UID 343593; U NOT reproduced — 5th non-reproduction); enum bypass (typed without dropdown); contact exclusion violated (TrackingNumber/RequestAssignmentDate/FirstName/LastName/ContactAddress); P6 display: blank Target States + blank Additional Criteria (display artifact only — criterion WAS in payload); Session:69d22bb72697c9202c0bcaa0; Method ID:46925, Acct:45928; ACTIVE ✓ |
| 21 | R1b | | | | | | | | | | | | | | | | | | | | | | | | — | LendingTree, Webhook/JSON 24/7, Exclusive, Order ON, 2 enum criteria |
| 22 | R1b | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/23 (96%) | LendingTree, Portal/Mon-Fri 9am-5pm EST (timezone→UTC-08:00 bug), Shared, Order OFF; **V TRIPLED** (schedule card×3 + delivery type card×2); timezone normalization error Eastern→UTC-08:00; text fallbacks accepted; P NOT reproduced (NY=33\|OH=36 ✓); P5-CR3: PASS (LoanAmount GreaterOrEqual 50000, UID 144924); Session:69d2395b2697c9202c0bcb60; Method:46927, Acct:45930; ACTIVE ✓ |
| 23 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | LendingTree, Webhook/XML, Weekdays 8am-6pm CST, Exclusive, Order ON; **W4** (content type card not rendered after webhook URL — agent frozen ~3min, text fallback "XML" unblocked); P/Q/U/V NOT reproduced; 8/8 mapping ✓; FL(10)+TX(44) ✓; all 3 criteria in payload (SelfCreditRating=343593, LoanRequestPurpose=343609, LoanAmount≥75000); no contact field violations ✓; Session:69d23ebe2697c9202c0bcc0a; Method:46928, Acct:45931; ACTIVE ✓ |
| 24 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) PASS | LendingTree, Email/24/7, Shared, Order OFF, no criteria; P/Q/V/W1 NOT reproduced; states CA(5)+WA(48) ✓; Skip accepted (additionalCriteria=None); clean field suggestions (no contact fields) ✓; Session:69d243962697c9202c0bcc57; Method:46929, Acct:45932; ACTIVE ✓ |
| 25 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/23 (96%) | LendingTree, FTP/Mon-Fri 9am-5pm MST, Exclusive, Order ON; **P reproduced** (states prompt skipped in P5, reappeared AFTER criteria loop — resequenced); states AZ(3)+NM(32) correctly persisted as first criterion despite resequencing ✓; both additional criteria in payload (SelfCreditRating=343593, LoanAmount≥100000) ✓; Q/U/V not reproduced ✓; 0/0 FTP field mapping (correct) ✓; no contact fields in suggestions ✓; Session:69d245e72697c9202c0bcc8c; Method:46930, Acct:45933; ACTIVE ✓ |
| 26 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | LendingTree, Webhook/URL-Encoded Mon-Wed-Fri 8-8 PST, Shared, Order OFF, 1 numeric; **W4-variant** (content type card resequenced — appeared AFTER instructions text, not before; ~3.5 min freeze after "I'll provide instructions" click); P/Q/U/V NOT reproduced ✓; OR(38)+WA(48) ✓; LoanAmount≥50000 in payload ✓; 8/8 URL-Encoded mapping ✓; no contact fields in suggestions ✓; Session:69d24a392697c9202c0bccd6; Method:46931, Acct:45934; ACTIVE ✓ |
| 27 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | 22/23 (96%) | LendingTree, Portal/24/7, Exclusive, Order ON, 2 enum; P/Q/U-drop/V/W4 NOT reproduced ✓; FL(10)+TX(44) ✓; SelfCreditRating In+UID(343593) ✓; **LoanRequestPurpose=Equal+string("PURCHASE") ✗** (operator/value mismatch — should be In+UID 343609); pre-creation confirmation prompt shown before create_delivery_account call; Session:69d250b32697c9202c0bcd29; Method:46932, Acct:45935; ACTIVE ✓ |
| 28 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/23 (96%) | LendingTree, Webhook/JSON-nested Tue-Thu 9-6 EST, Shared, Order ON, 1 enum; **V** reproduced (schedule card + delivery type card doubled — 8th+ occurrences, both bypassed via text); **P resequenced** (states appeared AFTER criteria loop, WA(48)+OR(38) still correct); **content type step skipped** (URL→mapping directly, JSON auto-detected); 9/9 JSON-nested mapping ✓; SelfCreditRating In+343593 ✓; display label "at least EXCELLENT" bug (payload correct); Session:69d264132697c9202c0bcd6a; Method:46933, Acct:45937; ACTIVE ✓ |
| 29 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | LendingTree, Email/Mon-Fri 8-5 CST, Exclusive, Order OFF, 2 numeric; **email address prompt skipped** (delivery type→P4 directly, no email addr collected); states in correct position ✓; GA(11)+FL(10)+NC(34) ✓; LoanAmount GreaterOrEqual 100000 ✓; AnnualIncome GreaterOrEqual 50000 ✓; P/Q/U/V NOT reproduced ✓; Session:69d26cba2697c9202c0bcdfc; Method:46934, Acct:45938; ACTIVE ✓ |
| 30 | R1b | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 23/23 (100%) | LendingTree, FTP/24/7, Shared, Order ON, 1 enum; **enum bypass** (SelfCreditRating "excellent" fuzzy-matched without dropdown); "at least EXCELLENT" display label bug (payload correct In+343593); P/Q/U/V NOT reproduced ✓; TX(44)+CO(6)+MN(24) ✓; Session:69d26f8b2697c9202c0bce64; Method:46936, Acct:45939; ACTIVE ✓ |
| 01 | R2 | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | F | F | F | Y | Y | Y | Y | 14/23 (61%) | RA-NEW-1: posting instructions prompt skipped (recovered via DEBUG); RA-P: states skipped entirely; Phase 5 batched: criteria gate→skip→create account in one response |
| 02 | R2 | Y | Y | F | Y | F | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | F | F | F | Y | F | Y | Y | 15/23 (65%) | RB-G: schedule prompt doubled; RB-X: empty card on "I'll provide instructions" (posting instr asked after content type, not before); 17/19 fields mapped correctly; RA-NEW-3: states re-prompted when full names given; RA-NEW-4: "Please type Continue to proceed" after "Add criteria" → criteria bypassed; P6-SUMM skipped |
| 03 | R2 | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | F | F | N/A | N/A | Y | Y | Y | Y | Y | 18/21 (86%) | XML→JSON mismatch not tested (user selected JSON directly); RB-G: price prompt doubled; States+criteria gate merged in one card [35] then gate shown again [37] (W-variant); Phase 5c silent skip: criteria builder loaded but ran 0 cards + 0 user interactions → create_delivery_account called immediately; Additional Criteria: None; 7/7 JSON mapping ✓; conn test correctly skipped; DA:45955 session:69d347e42697c9202c0bd5d5 |
| 04 | R2 | F* | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | N/A | F | F | N/A | N/A | Y | Y | Y | Y | Y | 14/20 (70%) | RA-G: Phase 5 entry prompt doubled (FTP creds text ×2); RA-NEW-7: Phase 5 prompt hallucination — asked FTP credentials then "account name/webhook credentials" instead of price→states→excl→order→criteria; agent improvised entire Phase 5 flow post-summarization; JSON→XML switch PASS (content type card re-shown, XML schema re-asked, 7/7 remapped); criteria gate never shown (scenario requires 3 criteria); account created correctly after manual data injection ($30, CA/TX, Shared, Order=No); Session:69d35857fe97b9130c36b722 |
| 05 | R2 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | F | F | F | F | F | Y | Y | Y | Y | 17/23 (74%) | RA-P: states question skipped after Order=No (2nd S-R2 occurrence); recovered CA,TX,FL,NY via DEBUG recovery; RA-NEW-R2-8: criteria builder bypassed after 2nd "Add criteria" click — agent created account with Additional Criteria: None; FIX 14 PASS (conn test failure text in card ✓); 11/11 URL-Enc mapping ✓; no P1 prompt doubling ✓; Session:TBD |
| 06 | R2 | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | F | F | F | Y | F | ? | ? | 12/21 (57%) | RA-G: schedule prompt doubled; RA-D: phase regression after Webhook click (×2); RA-P: states skipped (3rd S-R2 occurrence); 9/9 XML auto-detect ✓; FIX 14 PASS (conn test failure in card ✓); account created blank Target States + blank Criteria; states msg sent post-creation → agent spun indefinitely → session timeout; P7/P8 not observed |
| 07 | R2 | Y | Y | Y | F | F | Y | Y | Y | Y | F | F | F | Y | Y | F | F | F | F | F | Y | Y | Y | Y | 13/23 (57%) | RA-D: Phase 2/3 regression loop after "I'll provide instructions" click (LT dropdown re-shown ×2, schedule re-asked); P3-WURL: webhook URL never collected; RA-NEW-7: Phase 5 FTP hallucination (2nd R2 occurrence, doubled); all P5 collected via plain text after manual correction; 3 criteria provided (LoanAmount>=100000, SelfCreditRating=EXCELLENT, AnnualIncome>50000) all dropped (Additional Criteria: None); JSON auto-detect ✓; 12/12 mapping ✓; FIX 14 PASS; ACTIVE ✓ |
| 08 | R2 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | Y | 18/20 (90%) | FTP clean through P4; FTP creds prompt doubled (RA-G); FIX 14 PASS; P5 cards all shown (Excl/Order/States); states+criteria gate merged in one message (RA-NEW-6); field suggestions shown ✓; SelfCreditRating enum dropdown ✓ (UID 343593); "Add another criterion" → create_delivery_account failed prematurely; 2nd criterion LoanAmount showed wrong enum values (SelfCreditRating leak); Additional Criteria: None (RA-U); TX/FL/CA normalized ✓; ACTIVE ✓ |
| 09 | R2 | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | 14/14 (100%) PASS | Portal + skip criteria; cleanest run in R2; no regression, no hallucination; states prompt doubled (RA-G cosmetic) but functional; NY/OH normalized ✓; Skip criteria → None ✓; ACTIVE ✓ |
| 10 | R2 | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | 14/14 (100%) PASS | Email + no criteria; clean run; criteria gate skipped (correct for scenario — no user-facing skip needed); states GA/WA not doubled ✓; no prompt doubling anywhere; ACTIVE ✓ |

† P1-PROMPT = cosmetic checkpoint. Failures are prompt text doubling only (intro text appears twice) — no functional impact, user can still proceed normally. F* = cosmetic failure, not counted in the percentage calculation.

---

## Per-Phase Failure Rates (R1 vs R2)

R1 = runs 01–10, gpt-5.4-mini all phases (mixed during runs 11–20). R2 = runs 01–06 so far, gpt-5.4-mini all phases.  
Denominator excludes N/A and ? entries. 🔴 = ≥50% · 🟡 = 20–49% · ✅ = <20%

| Checkpoint | R1 fail (n=10) | R2 fail (n=6) | Trend |
|------------|----------------|----------------|-------|
| P1-PROMPT† | 0/10 = 0% ✅ | 1/6 = 17% ✅ | ↔ |
| P2-DROP | 0/10 = 0% ✅ | 0/6 = 0% ✅ | ↔ |
| P3-SCHED | 2/10 = 20% 🟡 | 2/6 = 33% 🟡 | ↔ |
| P3-WURL | 1/10 = 10% ✅ | 0/6 = 0% ✅ | ↑ |
| P3-JSON | 1/10 = 10% ✅ | 2/6 = 33% 🟡 | ↓ |
| P3-TABLE | 1/10 = 10% ✅ | 0/6 = 0% ✅ | ↑ |
| P3-COUNT | 2/10 = 20% 🟡 | 0/6 = 0% ✅ | ↑ |
| P3B-TEST | 0/10 = 0% ✅ | 0/6 = 0% ✅ | ↔ |
| P4-SUMM | 0/10 = 0% ✅ | 0/6 = 0% ✅ | ↔ |
| P5-PRICE | 0/10 = 0% ✅ | 2/6 = 33% 🟡 | ↓ |
| P5-EXCL | 0/10 = 0% ✅ | 1/6 = 17% ✅ | ↔ |
| P5-ORDER | 0/10 = 0% ✅ | 1/6 = 17% ✅ | ↔ |
| P5-STATE | 5/10 = 50% 🔴 | 5/6 = 83% 🔴 | ↓↓ |
| P5-NORM | 5/10 = 50% 🔴 | 3/5 = 60% 🔴 | ↔ |
| P5-FIELD | 2/10 = 20% 🟡 | 5/6 = 83% 🔴 | ↓↓ |
| P5-CR1 | 3/10 = 30% 🟡 | 6/6 = 100% 🔴 | ↓↓ |
| P5-ENUM | 3/10 = 30% 🟡 | 4/4 = 100% 🔴 | ↓↓ |
| P5-CR3 | 8/10 = 80% 🔴 | 4/4 = 100% 🔴 | ↔ |
| P5-DONE | 10/10 = 100% 🔴 | 4/6 = 67% 🔴 | ↑ |
| P6-BOOL | 0/10 = 0% ✅ | 0/6 = 0% ✅ | ↔ |
| P6-SUMM | 1/10 = 10% ✅ | 2/6 = 33% 🟡 | ↓ |
| P7-SUMM | 0/10 = 0% ✅ | 0/5 = 0% ✅ | ↔ |
| P8-ACT | 0/10 = 0% ✅ | 0/5 = 0% ✅ | ↔ |

**Key observations:** P5-STATE/FIELD/CR1/ENUM all worsened in R2 despite (or due to) new post-summarization behavior. P5-CR3/DONE remain critically high. P3-P4 phases are stable. P5-DONE improved slightly (100%→67%) suggesting some criteria are now persisting in R2.

---

## Cross-Round Comparison

### R2 vs R1 Comparison

| Finding | R1 Rate (01-10) | R2 Rate (01-10) | Status |
|---------|-----------------|-----------------|--------|
| RA-U (criteria not persisted) | 5/8 = 63% | 5/5 = 100% (runs w/ criteria) | ↓ WORSE — only run 08 showed field suggestions, still dropped |
| RA-P (states silently skipped) | 3/8 = 38% | 3/10 = 30% (runs 01,05,06) | ↔ SAME — no fix applied |
| RA-J (states not normalized) | 3/8 = 38% | 0/7 = 0% | ↑ FIXED — all states USPS ✓ |
| RA-G (prompt doubling) | ~20% | 5/10 = 50% | ↓ WORSE — dedup rule insufficient |
| RA-D (phase regression) | 2/8 = 25% | 1/10 = 10% (run 07) | ↑ IMPROVED |
| RA-NEW-7 (P5 FTP hallucination) | 0 (new in R2) | 2/10 = 20% (runs 04,07) | NEW — post-summarization Phase 5 loss |
| Overall pass rate | 1/10 = 10% | 2/10 = 20% | ↑ IMPROVED |
| Average score | ~67% est | 75% | ↑ IMPROVED |

### Phase Stability Comparison: GPT-5.4 (runs 01–20) vs GPT-5-mini Webhook/Criteria (runs 21–30)

*(Populated after runs 21–30 complete)*

| Phase | GPT-5.4 Fail% (01–20) | GPT-5-mini Fail% (21–30) | Delta | Notes |
|-------|----------------------|--------------------------|-------|-------|
| P3-SCHED | 10% | — | — | |
| P3-WURL | 5% | — | — | Webhook phases use GPT-5-mini in runs 21–30 |
| P3-JSON | — | — | — | Webhook phases use GPT-5-mini in runs 21–30 |
| P3-TABLE | — | — | — | |
| P3-COUNT | — | — | — | |
| P5-STATE | 35% | — | — | Criteria phases use GPT-5-mini in runs 21–30 |
| P5-NORM | 35% | — | — | |
| P5-CR3 | 55% | — | — | Criteria phases use GPT-5-mini in runs 21–30 |
| P8-ACT | 5% | — | — | |

### 20-Run Summary — Version A Stabilized

**All 20 runs completed. All 20 FAILed (no clean pass).**

#### Finding Frequency (20 runs)

| Finding | Description | Frequency | Severity |
|---------|-------------|-----------|----------|
| U | Criteria not persisted to account | **9/14** qualifying runs (05–10, 12–13, 19 reproduced; 15–18, 20 NOT reproduced); nondeterministic; A19 broke 4-run non-reproduction streak; A20 resumed non-reproduction | Critical |
| P | States silently skipped | **7/20** (runs 04, 06, 15, 17, 18, 19, 20) | Critical |
| Q | Phantom state UIDs when P occurs | **7/20** (same runs as P); always CA=5 as the hallucinated default | High |
| V | Prompt text doubled | **6/20 runs** (05, 06, 13, 14, 15, 20); run 14 had 2 instances (P3+P5) | Medium |
| J | State normalization skipped (display-only) | 4/20 (runs 01–03 display+API; run 14 display-only — API correct) | High |
| N | Schedule hours or body prompt skipped | 3/20 (runs 03, 04, 11) | High |
| D | Phase regression after card click | **3/20** (runs 01, 06, 16) | High |
| W | States + criteria merged in single response | 1/20 (run 08) | Medium |
| X | Phase 3 instructions loaded 3× (triple load) | 1/20 (run 11) | Medium |
| Y | Typed criterion ignored at criteria card | 1/20 (run 11); NOT reproduced in runs 12–20 | High |
| K | Criteria phase entirely skipped post-summarization | 1/3 (run 01) | Critical |
| O | Phase 5 restart after criteria loop exit | 1/3 (run 03) | Critical |
| R | Criteria gate bypassed after state skip | 1/13 (run 04) | Critical |
| S | Webhook URL never requested | 1/13 (run 04) | High |
| T | JSON parse error before schema provided | 1/13 (run 04) | Medium |

#### Key Observations

1. **Finding U (9/14 qualifying runs) — fundamentally nondeterministic, cause unknown.** Criteria discarded in runs 05–13. Then 4 consecutive non-reproductions (A15–A18). A19 broke the streak; A20 resumed non-reproduction. No common factor explains the pattern. U reproduction rate: ~64% when criteria are entered. The summarize_history mechanism remains the strongest candidate but is not confirmed.

2. **Finding P (states silently skipped) now 7/20 runs (35%)** — runs 04, 06, 15, 17, 18, 19, 20. It appears on all delivery types (Webhook, Portal, FTP, Email). When P occurs, Finding Q always follows: agent halluccinates CA (UID 5) as a phantom state criterion regardless of what the user intended. The states-prompt skip is nondeterministic but persistent, rising from 4/17 to 7/20 across the full test series.

3. **Run 13 confirms enum fuzzy-match bypass pattern.** Both criteria were typed as plain text ("SelfCreditRating excellent", "LoanRequestPurpose refinance") and accepted via fuzzy matching without showing a dropdown — agent went straight to the criteria loop card. Runs 17, 18, and 20 confirm the same behavior: enum bypass is consistent, not nondeterministic. The instruction says isEnumerated fields should use a dropdown (Input.ChoiceSet) but the agent bypasses it when typed values match >85%.

4. **Contact field exclusion rule not respected — persistent across all LendingTree runs.** Per Phase 5 Step 6: "Exclude contact/personal information lead fields." TrackingNumber, RequestAssignmentDate, FirstName, LastName, ContactAddress consistently appear in the first 5 suggestions. SSN was seen in batch 3 (Run 13). This is a deterministic instruction violation, not nondeterministic.

5. **Finding V (prompt text doubled) confirmed in 6/20 runs** — Run 14 had 2 instances in a single run (P3 JSON schema prompt + P5 price prompt both doubled). Run 15 added a new location: delivery method name. Run 20 doubled the schedule prompt. The pattern appears in multiple phases, not limited to specific prompts. Triggers when a card button is clicked and the agent's next response includes the same prompt text twice.

6. **Finding J (state normalization) confirmed as display-only in Run 14.** When full state names are provided ("New York, California, Texas"), P6 displays them un-normalized. However, `get_usa_states()` in Step 10 correctly resolves names to UIDs at API call time — the payload had correct UIDs (33|5|44). Finding J is cosmetic only.

7. **Finding Y (criteria text input ignored) observed in Run 11, NOT reproduced in Runs 12–20.** Nondeterministic — low frequency, appears resolved by run 12+.

8. **Finding N now has a second variant (N2 — body prompt skipped).** After "I'll provide instructions" + content type selection, the posting instructions prompt is skipped and the delivery method is created with `requestBody=null`.

9. **Portal and Email delivery types work correctly** — the agent correctly skips connection tests, handles schedules, and creates methods. The FTP branch also works correctly.

10. **Run 14 activation failed due to platform tool error** — `activate_client` failed 3 consecutive times. Not a flow bug. All other runs activated successfully.

11. **Finding U / Finding P correlation is not causal.** Runs 15, 17, 18, 20 show U absent while P is present. Run 16 shows U absent while P is not present. Run 19 shows U present while P is present. U is not tied to Finding P, delivery type, schedule, or exclusivity. True root cause remains unconfirmed.

12. **Finding D is broader than "after Webhook click."** Run 16 triggered D after clicking "I'll provide instructions". The regression pattern is: after ANY card button click that transitions between major flow steps, the agent may regress to Phase 2. The trigger is not Webhook-specific.

13. **New A20 observation: P6 displays blank Additional Criteria even when criterion IS in payload.** When Finding P occurs, the `criteriaList` display string (used in P6 summary) may not be populated even when `criteriaPayload` array is non-empty. This means P6's Additional Criteria field cannot be trusted as a ground-truth indicator of what's in the payload when Finding P is active. Payload audit via session log JS is required for accurate assessment.

#### Pass/Fail by Phase (20-run aggregate)

| Phase | Pass | Fail | Fail % |
|-------|------|------|--------|
| P1-PROMPT† | 20 | 0 | 0% |
| P2-DROP | 20 | 0 | 0% |
| P3-SCHED | 18 | 2 | 10% |
| P3-WURL | 19 | 1 | 5% |
| P5-STATE | 13 | 7 | 35% |
| P5-NORM | 13 | 7 | 35% |
| P5-CR3 (criteria persist) | 9 | 11 | 55% (9/14 qualifying) |
| P8-ACT | 19 | 1 | 5% (platform error) |

---

## Findings Catalog

### Root Cause Classifications

- **IGNORE** — The instruction is explicitly present in the instruction pack (verified), but the AI silently skipped it. The instruction uses language like "Do NOT skip", "MANDATORY", "STOP AND YIELD" — the AI simply did not follow it.
- **HALLUCINATE** — The AI generated content or behavior not present in the instructions (e.g., asking for FTP credentials on a webhook method, fabricating phantom state values).
- **AMBIGUOUS** — The instruction is unclear, contradictory, or has a gap that led to the failure. Potentially fixable via instruction changes — requires analysis.
- **PLATFORM** — Non-deterministic card rendering, MCP resource loading failures, context overflow, tool timeouts — not an instruction or AI behavior issue.

---

### Finding A — Lead type typed fallback asks for UID instead of matching by name [R1a]

**Phase:** 2 | **Severity:** Medium | **Frequency:** 1/3 (Run 01)  
**Observed:** Typed "Health Insurance" as text; agent responded asking for the lead type UID instead of matching the provided name to available options.  
**Expected:** Agent should fuzzy-match the typed name against available lead types and proceed.

**DEBUG Response Summary:**
- Agent confirmed it received the text input but treated it as an ambiguous identifier requiring a UID to resolve
- Root cause: instructions direct the agent to resolve lead type by UID when the card is not used; no fuzzy-name-match fallback exists
- Agent self-diagnosis: the typed path lacks a "match by name" resolution step — it falls back to requesting a UID directly

**Suggested fix:** Add name-matching fallback in the lead type resolution logic before requesting UID.

---

### Finding B — Hallucinated UID in DEBUG response [R1a]

**Phase:** 2 | **Severity:** Low (DEBUG artifact) | **Frequency:** 1/3 (Run 01)  
**Observed:** DEBUG response cited UID 3 = "Health Insurance", but actual data shows UID 3 = "Dup Check" and Health Insurance = UID 1461.  
**Expected:** DEBUG responses should reflect actual tool payloads.

**DEBUG Response Summary:**
- Agent reported a mapping that contradicted the actual MCP resource data
- This appears to be a confabulation during the DEBUG explanation, not a tool call result
- Suggests the agent's internal representation of the lead type list was fabricated rather than retrieved

**Suggested fix:** Investigate whether the lead type list is actually being fetched on phase entry or assumed from training data.

---

### Finding C — deliveryDays not built from schedule input [R1a]

**Phase:** 3 | **Severity:** High | **Frequency:** 1/3 (Run 01)  
**Observed:** After accepting "Mon-Fri 9am-5pm PST", the agent used fallback deliveryDays values rather than building from the input.  
**Expected:** Agent should parse "Mon-Fri 9am-5pm PST" into correct day/time/timezone values for the deliveryDays payload.

**DEBUG Response Summary:**
- Agent confirmed deliveryDays were NOT correctly built from the schedule input
- The schedule input was accepted in Phase 3 but not yet normalized before the webhook branch selection happened
- deliveryDays currently contain fallback values from an earlier erroneous path
- Agent acknowledged this as a bug in the ordering of schedule processing vs. branch selection

**Actual tool call data (from DEBUG ID confirmation):**
```
"deliveryDays": [
  {"weekDay":0,"startTime":"...T00:00:00-08:00","endTime":"...T23:59:59-08:00","allow":false},
  {"weekDay":1,...,"allow":false},  // Mon — should be allow:true, 09:00-17:00
  {"weekDay":2,...,"allow":false},  // Tue — same
  {"weekDay":3,...,"allow":false},  // Wed — same
  {"weekDay":4,...,"allow":false},  // Thu — same
  {"weekDay":5,...,"allow":false},  // Fri — same
  {"weekDay":6,...,"allow":false}   // Sat — correct (off)
]
```
All 7 days: `allow: false`, times `00:00:00–23:59:59`. Expected: Mon-Fri `allow: true`, `09:00–17:00`, PST.

**Positive note:** requestBody correctly contained only mapped fields (`[FirstName]`, `[LoanAmount]`), metadata fields excluded.

**Suggested fix:** Ensure schedule parsing/normalization occurs before the delivery method branch is entered, or re-normalize after branch selection completes.

---

### Finding D — Phase regression after Webhook button click [R1a, R2]

**Phase:** 3 | **Severity:** High | **Frequency:** 2/6 (Run 01, Run 06 — NOT reproduced Runs 02–05)  
**Observed:** After clicking the "Webhook" button on the delivery method card, agent re-showed the schedule question instead of proceeding to webhook URL prompt.  
**Expected:** Agent should proceed directly to asking for the webhook URL.

**DEBUG Response Summary:**
- Root cause: card submission was received as plain value `choice: Webhook` but the branch state was not preserved correctly between steps
- The structured choice should have been sufficient to enter the Webhook branch
- Instead, the flow re-issued an earlier schedule prompt, indicating the selected branch was lost
- Agent self-corrected and eventually asked for webhook URL after the regression

**Suggested fix:** Fix branch state preservation after card submission — ensure the chosen delivery method is committed to state before the next phase step executes.

#### R2 occurrences

**R2 Run 06:** RA-D: phase regression after Webhook click (×2).

**R2 Run 07:** RA-D: Phase 2/3 regression loop after "I'll provide instructions" click (LT dropdown re-shown ×2, schedule re-asked).

---

### Finding E — Content type asked as plain text, not ActionSet card [R1a]

**Phase:** 3 | **Severity:** Medium | **Frequency:** 1/3 (Run 01 only — NOT reproduced Run 02; 4-button card appeared correctly)  
**Observed:** Agent asked "What content type should this delivery use?" as a plain text message. No ActionSet card with buttons (URL Encoded / JSON / XML / I'm not sure).  
**Expected:** Protocol expects 4-button ActionSet card for content type selection.  
**Checkpoint:** N/A (not in scoring matrix, but impacts usability and consistency)

**DEBUG Response Summary:** *(not investigated separately — noted from behavior)*  
**Suggested fix:** Instruction should explicitly call for rendering a content type ActionSet card at this step.

---

### Finding F — Only 1 of 8 fields mapped (P3-COUNT FAIL) [R1a]

**Phase:** 3 | **Severity:** High | **Frequency:** 1/3 (Run 01)  
**Observed:** "Successfully mapped 1 out of 8 fields." — only FirstName → first_name mapped.  
**Expected:** Multiple fields should fuzzy-match. LoanAmount at minimum should have matched loan_amount.

**DEBUG Response Summary:**
- Lead type fields available: **FirstName, LoanType, LoanAmount** (only 3, not 8)
- Matching logic applied: exact → underscore/camelCase → abbreviations → semantic (high confidence only)
- `first_name` matched `FirstName` directly
- `loan_amount` SHOULD have matched `LoanAmount` — agent self-admitted this was an under-mapping error
- `last_name`, `email_address`, `phone`, `property_value`, `property_type`, `state` — no system field available for this lead type
- Root cause 1: lead type only had 3 system fields, so max possible mappings = 3
- Root cause 2: agent failed to map loan_amount → LoanAmount despite the match existing

**Instruction Patch (agent recommendation):**
In the Field Mapping & RequestBody Generation step, replace the matching priority with:
1. exact match
2. normalized match after lowercasing and removing underscores, hyphens, spaces
3. underscore-to-camelCase and camelCase-to-underscore equivalent match
4. abbreviations
5. semantic mapping only if confidence is >90%

Add explicit rule: normalize both strings (lowercase, strip separators) before comparison — if normalized forms match, treat as matched immediately.
Add safeguard: preserve every leaf field name from JSON/XML and attempt mapping independently for each, including nested fields.

---

### Finding G — Connection test success message silently swallowed [R1a]

**Phase:** 3b | **Severity:** Medium | **Frequency:** 1/3 (Run 01)  
**Observed:** After clicking "Test Connection", no success/failure message shown — agent jumped directly to "Delivery Method Created" summary card.  
**Expected:** Agent should show "Connection test successful." before proceeding.

**DEBUG Response Summary:**
- Connection test WAS run (httpbin.org/post returned 200 OK)
- Agent moved on to summary without showing the result
- Agent: "The visible outcome was a successful POST to httpbin" — confirmed test ran but result display was skipped
- Root cause: instructions likely call `summarize_history` or transition to Phase 4 without rendering a test result message first

**Instruction Patch (agent recommendation):** Add explicit step after `test_webhook_connection` to render result message before proceeding.

---

### Finding H — Company name state retention error (StabilityTest-50 vs StabilityTest-01) [R1a]

**Phase:** 4 summary | **Severity:** High | **Frequency:** 1/3 (Run 01)  
**Observed:** Phase 4 summary shows "StabilityTest-50" as company name; Phase 1 input was "StabilityTest-01".  
**Expected:** Company name set in Phase 1 should be preserved throughout the entire flow.

**DEBUG Response Summary:**
- Agent confirmed the retained Phase 1 value was overwritten during conversation
- Root cause: state retention error — likely caused by the earlier phase regression (Finding D) corrupting the stored company name
- Agent: "This is a state retention error, not something you did wrong"
- The delivery method was created under the wrong client (StabilityTest-50)
- Note: this finding may be an artifact of Run 01's regression, not a stable reproducible bug — confirm in Runs 02/03

**Suggested fix:** Investigate whether phase regression (Finding D) clears or overwrites the companyName state variable. Ensure Phase 1 values are committed to stable state before any branch selection.

---

### Finding I — Agent hallucinated DEBUG tool call response (Portal vs Webhook) [R1a]

**Phase:** 3 / 4 | **Severity:** Critical | **Frequency:** 1/3 (Run 01)  
**Observed:** Agent's DEBUG response described creating a Webhook method (`HttpPost`, `StabilityTest-50-Webhook`, with field mappings). Actual session log shows the tool call was a Portal creation (`HttpGet`, `StabilityTest-50-Portal`, no field mappings).  
**Expected:** DEBUG tool call descriptions should match actual tool payloads.

**Actual session log payload:**
```json
{
  "deliveryType": "HttpGet",
  "name": "StabilityTest-50-Portal",
  "deliveryDays": [all false, 00:00:00–23:59:59]
}
```
**Agent DEBUG claimed:**
```json
{
  "deliveryType": "HttpPost",
  "name": "StabilityTest-50-Webhook",
  "settings": [field mappings...]
}
```

**Root cause analysis:**
- Most likely caused by the phase regression (Finding D): when Webhook button was clicked and the flow regressed, the agent may have created a Portal delivery method on an earlier incorrect branch
- The Webhook method may not have been created at all, or both were created
- The agent then confabulated a plausible-looking tool call in DEBUG that matched what "should have happened" rather than what actually did
- This is an instance of the agent hallucinating historical tool calls during DEBUG — a fundamental reliability concern

**Severity note:** Confirmed by Phase 7 summary — both Delivery Method and Delivery Account show the same ID (46887). Same ID across two different entities = one (or both) is hallucinated. The real creation from the session log was Portal (ID 46887). The Delivery Account (46887) and the Webhook name are hallucinated references to the same real entity.

**Suggested fix:** Instructions cannot prevent hallucinated DEBUG responses (this is an LLM limitation). However, the phase regression that caused the incorrect branch execution (Finding D) should be fixed to prevent it from happening in practice.

---

### Finding J — State normalization skipped (P5-NORM FAIL) [R1a]

**Phase:** 5 | **Severity:** High | **Frequency:** 3/3 (all runs)  
**Observed:** "California, Texas, Florida" stored and displayed as-is, not normalized to CA, TX, FL abbreviations.  
**Expected:** States should be normalized to USPS abbreviations and stored as state UIDs.

**DEBUG Response Summary:**
- Agent: "I retained the raw user text instead of the normalized target state list"
- Root cause: **post-summarization context loss** — `summarize_history` call removed Phase 3/4 conversation state; agent no longer tracked that it needed to normalize states before building the account payload
- The normalization step requires fetching state IDs and converting names — this was skipped because the step ordering was lost after summarization

**Suggested fix:** State normalization instruction must be self-contained within Phase 5 resource (not dependent on earlier conversational context). The resource should always run the normalize-states step before building the account payload, regardless of prior conversation history.

---

### Finding K — Criteria phase entirely skipped (P5-FIELD / P5-CR1 / P5-ENUM / P5-CR3 / P5-DONE all FAIL) [R1a]

**Phase:** 5 | **Severity:** Critical | **Frequency:** 1/3 (Run 01 only — NOT reproduced Run 02; criteria phase appeared but had other failures — see L, M)  
**Observed:** After entering target states, agent went directly to account summary. No field suggestions shown, no criteria loop, account created with "Additional Criteria: None".  
**Expected:** Agent should fetch lead fields, show suggested fields, enter criteria loop, collect 3 criteria, exit on "done".

**DEBUG Response Summary:**
- Agent: "I did not execute those steps [fetch lead fields, show suggestedFields, criteria loop] before moving to the account summary"
- Root cause: **same as Finding J** — post-summarization context loss caused premature transition to account summary
- Agent: "Current correct state should have been: Phase 5 account setup, after target states collection, before criteria suggestions"
- Agent offered to retroactively run criteria step, but account was already created so this would be an edit flow, not the main flow

**Suggested fix:** Phase 5 resource must be structured so that the criteria loop is explicitly gated — the agent cannot transition to account summary until the criteria phase is explicitly exited (via "done" or a button). This gate must survive summarization.

---

### Finding L — "done" typed as text stored as criteria value instead of exiting loop [R1a]

**Phase:** 5 | **Severity:** High | **Frequency:** 1/3 (Run 02)  
**Observed:** Typed "done" to exit the criteria loop. Agent treated "done" as a criterion value and stored it in the account. The account summary showed "done" as a criteria entry rather than exiting the loop.  
**Expected:** Typing "done" should signal end of criteria input and trigger exit from the loop without creating a new criterion.

**DEBUG Response Summary:** *(not investigated in Run 02 — observed from account summary showing "done" as a criterion)*  
**Suggested fix:** The criteria loop exit check must fire BEFORE the agent attempts to parse the input as a criterion. Add: if input === "done" (case-insensitive), exit loop immediately without processing as criterion.

---

### Finding M — Enum criterion handling failure (P5-ENUM FAIL) [R1a]

**Phase:** 5 | **Severity:** Medium | **Frequency:** 1/3 (Run 02)  
**Observed:** When entering an enum-type criterion (In/NotIn operator with discrete values), the agent failed to handle it correctly. Specific failure mode to be confirmed in Run 03.  
**Expected:** Agent should present valid enum values and accept comma-separated selection, constructing the correct In/NotIn filter payload.

**DEBUG Response Summary:** *(to be investigated in Run 03 if reproduced)*  
**Suggested fix:** Verify enum criterion instructions survive summarization and are self-contained in Phase 5 resource.

---

### Finding N — Schedule hours and field mapping input both skipped in flow (P3-SCHED / P3-JSON / P3-TABLE / P3-COUNT all FAIL) [R1a]

**Phase:** 3 | **Severity:** Critical | **Frequency:** 2/4 (Run 03: both sub-issues; Run 04: sub-issue 1 only)  
**Observed (two sub-issues captured in one DEBUG session):**  
1. After "Specific hours only" was selected, the agent did not ask for the actual hours — jumped directly to asking for delivery method type.  
2. After "I'll provide instructions" and "JSON" were selected, the agent did not ask for the actual JSON body — jumped directly to the connection test prompt. The delivery method was created with 0 field mappings.

**Expected:** After "Specific hours only" → agent should ask "Please provide your hours (e.g., Mon-Fri 9am–5pm PST)". After "I'll provide instructions" + content type selected → agent should ask "Please paste your JSON body."

**DEBUG Response Summary:**
- Root cause 1 (schedule): "The workflow should have paused to collect the actual hours after that option was chosen, but the next phase was allowed to continue too early."
- Root cause 2 (field mapping): "The flow accepted the JSON body, but the step that should display the mapping/summary table was not executed before moving on."
- Agent offered to draft exact patches for both in "the instruction style used by this workflow."

**Instruction Patch (agent recommendation):**
1. Schedule: Add an explicit WAIT step immediately after "Specific hours only" is selected; require schedule-hour details before any delivery method creation continues.
2. Field mapping: After JSON body is collected, insert a mandatory structured summary step that displays the parsed field mappings in a table and confirms them before proceeding.

---

### Finding O — Phase 5 restart triggered by "continue" after criteria loop (P5-DONE variant) [R1a]

**Phase:** 5 | **Severity:** High | **Frequency:** 1/3 (Run 03)  
**Observed:** Typed "continue" to exit the criteria loop (as instructed by "Please type continue when you're done adding criteria"). The agent showed the Phase 4 "Delivery Method Created" summary card again. When "Continue" was clicked on that repeated card, Phase 5 restarted from the beginning (price per lead asked again). All previously entered criteria were lost; the account was ultimately created with "Additional Criteria: None".  
**Expected:** Typing "continue" should exit the criteria loop and proceed to Phase 6 (account summary), not re-trigger Phase 4 or restart Phase 5.

**DEBUG Response Summary:** *(not separately investigated — observed from flow behavior)*  
**Note:** In Run 02, "done" was stored as a criterion value (Finding L). In Run 03, "continue" caused a phase restart. Both indicate the criteria loop exit mechanism is unstable — different failure modes depending on exit keyword used.

**Suggested fix:** The criteria loop exit must be atomic: detect the exit keyword → create account → show account summary — in a single uninterruptible step. No intermediate card re-display that could be re-triggered.

---

### Finding P — States silently skipped entirely (≈ RB-N in Version B) [R1a, R1b, R2]

**Phase:** 5 | **Severity:** Critical | **Frequency:** 2/6 (Run 04, Run 06)  
**Observed:** After Order System selection, the agent skipped the state selection step entirely and jumped directly to creating the delivery account. No state prompt shown, no state UIDs collected.  
**Expected:** Agent should ask "Which states should this account target?" before creating the account.  

**Note:** RB-N is the equivalent defect in Version B (Rework). Identical failure mode — states silently skipped after Order System → phantom CA,AZ,TX appear in summary (Finding Q).  

**Suggested fix:** The state collection step must be explicitly gated and cannot be bypassed even after summarize_history. Phase 5 resource must include an unconditional state-collection step before calling create_delivery_account.

#### R1b occurrences

**R1b Run 25:** Finding P REPRODUCED — States prompt skipped in P5 Step 5 (after Order System = Yes, agent jumped directly to field suggestions without asking for target states). States prompt reappeared AFTER the criteria loop completed, as the final step before `create_delivery_account`. AZ(3)+NM(32) correctly inserted as first criterion despite resequencing.

**R1b Run 28:** Finding P resequenced — States prompt skipped before criteria loop; appeared AFTER criteria loop exited (same pattern as A25). WA(48)+OR(38) correctly built into state criterion despite resequencing.

#### R2 occurrences

**R2 Run 01:** RA-P: After Order System → No, agent jumped directly to criteria gate ("Would you like to add additional lead criteria, or skip?"), completely skipping "Which states do you want to target?" prompt. States collection steps (STEP 7-8 in stabilized) were silently skipped. Same finding as R1 RA-P (3/8 rate in R1). No fix was applied for this in stabilized variant.

**R2 Run 05:** RA-P (2nd S-R2 occurrence): After Order System → No, agent went directly to criteria gate without asking "Which states do you want to target?". No states question ever appeared in the DOM. Root cause identified via DEBUG: missing STOP AND YIELD after states prompt — Step 5 ("Match States and Build Initial Criteria") appeared immediately after state-field detection with no hard wait gate, allowing the agent to batch states detection + criteria gate in one turn. Agent patch suggestion: add explicit STOP AND YIELD after states prompt + guard Step 5 with "Execute only after user has provided targetStates in a prior turn." States recovered via DEBUG providing "CA, TX, FL, NY", which appeared correctly normalized in account summary.

**R2 Run 06:** RA-P: states skipped (3rd S-R2 occurrence).

---

### Finding Q — Phantom CA,AZ,TX states in account summary (≈ RB-O in Version B) [R1a]

**Phase:** 6 | **Severity:** High | **Frequency:** 1/4 (Run 04)  
**Observed:** P6 "Delivery Account Created" card shows "Target States: CA,AZ,TX" despite the user never entering any states. These are default/fallback values written into the account payload when state collection was skipped.  
**Expected:** If states were not collected, the states field should be empty or the flow should have asked for states.  

**Note:** RB-O is the equivalent defect in Version B. Likely same root cause — state UIDs fallback to CA (UID unknown), AZ, TX when the state collection tool call is not made.  

**Suggested fix:** If state collection step was skipped, do not use fallback UIDs. Either block account creation and re-prompt for states, or explicitly set targetStates to empty.

---

### Finding R — Criteria gate bypassed after state skip (≈ RB-D in Version B) [R1a]

**Phase:** 5 | **Severity:** Critical | **Frequency:** 1/4 (Run 04)  
**Observed:** When states were silently skipped (Finding P), the criteria builder phase was also bypassed. Account created with "Additional Criteria: None" despite the scenario requiring 5 criteria.  
**Expected:** Criteria collection should be independent of state collection — states being skipped should not also skip criteria.  

**Note:** RB-D is the equivalent defect in Version B. In both versions the three defects co-occur: P/RB-N (state skip) → Q/RB-O (phantom states) → R/RB-D (criteria bypass). Likely caused by the same flow control gap — after Order System, the agent jumps directly to create_delivery_account without visiting state or criteria steps.  

**Suggested fix:** Criteria gate must be independent of the state step. Enforce ordering: collect states → collect criteria → create account. Neither step should allow bypassing the other.

---

### Finding S — Webhook URL never requested (P3-WURL FAIL) [R1a]

**Phase:** 3 | **Severity:** High | **Frequency:** 1/4 (Run 04)  
**Observed:** After delivery type (Webhook) was selected, the agent jumped directly to asking "Would you like to configure field mappings?" without ever asking for the webhook URL/endpoint. The delivery method was created without a user-provided URL.  
**Expected:** Agent should ask "Please provide your webhook URL (endpoint):" before proceeding to field mapping.  

**Note:** Not reproduced in Runs 01–03 where P3-WURL was Y. May be a conditional path bug — only triggered in certain scenarios (e.g., when using "I'll provide instructions" path for field mapping).  

**Suggested fix:** Ensure webhook URL collection is unconditionally required before any field mapping or method creation step in the Webhook delivery branch.

---

### Finding T — JSON parse error shown before schema is provided [R1a]

**Phase:** 3 | **Severity:** Medium | **Frequency:** 1/4 (Run 04)  
**Observed:** After selecting JSON as content type, the agent immediately showed an error card: "I couldn't parse this as valid JSON. Would you like to: Fix and re-paste the JSON schema / Switch to another format." No JSON body had been provided yet — the user had not yet been asked to paste anything.  
**Expected:** Agent should first ask "Please paste your JSON body or posting instructions." before attempting to parse anything.  
**Recovery:** Clicking "Re-paste JSON schema" caused the agent to ask for the body; after pasting, 14/14 fields mapped correctly.  

**Suggested fix:** Insert an explicit step to request the JSON body BEFORE attempting to parse it. The parse step should only fire after the user has provided input.

---

### Finding U — Criteria entered via UI not persisted to account (Additional Criteria: None) [R1a, R1b, R2]

**Phase:** 5 | **Severity:** Critical | **Frequency:** 3/7 (Run 05, Run 06, Run 07)  
**Observed:** User added criterion via the UI dropdown (LoanRequestType = REFINANCE selected from enum values, Submit clicked → "Would you like to add another criterion" card appeared indicating success). Additional criteria also attempted (LoanAmount, AnnualIncome). After "Continue" typed in textarea, account was created with "Additional Criteria: None" — no criteria persisted.  
**Expected:** Criteria added via the dropdown UI should be included in the create_delivery_account tool call.  

**Note:** The criteria builder UI itself worked well (enum dropdown showed valid values, enum criterion submitted successfully). The failure is in persistence — the criteria state was not passed through to the API call when "Continue" was received via text. May be related to the same mechanism as Finding L ("done" stored as criteria) and Finding O (phase restart on "continue") — the exit keyword handling may discard previously accumulated criteria state.  

**Suggested fix:** Criteria collected via the UI dropdown must be accumulated in a persistent state variable. The account creation call must always include accumulated criteria regardless of how the continuation is triggered (button click vs. text message).

---

### Finding V — State prompt doubled (≈ RB-G in Version B) [R1a, R1b, R2]

**Phase:** 5 | **Severity:** Medium | **Frequency:** 2/6 (Run 05, Run 06)  
**Observed:** After Order System selection (No/Disabled), the state prompt appeared twice in the same agent response: "Finally, let's set up your Delivery Account. Please provide the states you want to target. For example: CA, AZ, TX." appeared duplicated (two identical text blocks in the same message).  
**Expected:** State prompt should appear exactly once.  

**Note:** This is the Version A equivalent of RB-G (systematic doubling in Version B). Not observed in Runs 01–04. May be triggered by the "action: No" text path from the Order System card button click — the action was processed correctly (Order System: Disabled in P6) but the state prompt was emitted twice.  

**Suggested fix:** Ensure the state collection step emits exactly one prompt regardless of input path. Add deduplication guard: do not re-emit a prompt if one was already issued in the current response.

#### R1b occurrences

**R1b Run 22:** Finding V TRIPLED — Schedule card appeared 3 times (new severity record vs 2x baseline); delivery type card appeared twice after schedule confirmation. Text fallbacks used for both.

**R1b Run 28:** Finding V REPRODUCED — Schedule card doubled (8th+ occurrence) and delivery type card (Portal/Webhook/Email/FTP) also doubled. Both bypassed via typed text fallbacks.

#### R2 occurrences

**R2 Run 02 (RB-G):** After clicking "Specific hours only", the agent output the schedule description prompt twice in the same message ("Please describe your preferred delivery schedule. (e.g., Mon-Fri 9am-5pm PST)" appeared twice). Previously seen in Run 01 for a different prompt. Persistent: appears across both variants and multiple prompts.

**R2 Run 03 (RB-G):** Price prompt doubled.

**R2 Run 04 (RA-G):** Phase 5 entry prompt doubled (FTP creds text ×2).

**R2 Run 06 (RA-G):** Schedule prompt doubled.

**R2 Run 08 (RA-G):** FTP creds prompt doubled.

**R2 Run 09 (RA-G):** States prompt doubled (cosmetic, functional).

---

### Finding W — States and criteria prompt merged in single response [R1a]

**Phase:** 5 | **Severity:** Medium | **Frequency:** 1/8 (Run 08)  
**Observed:** After Order System selection, agent issued "Which states do you want to target?" immediately followed by criteria field recommendations in the **same response**, without waiting for the user to provide states. When user responded "FL, CA, TX", states were correctly parsed and saved (P6 showed Target States: FL, CA, TX ✓). However, the criteria recommendations appearing alongside the states prompt created a confusing UX — user had no clear signal whether their input would be treated as states or criteria.  
**Expected:** States prompt should appear alone, wait for user input, then proceed to criteria builder as a separate step.  

**Relationship to Finding V:** Finding V (doubled prompt) and Finding W (merged states+criteria) are related — both stem from the Phase 5 instruction emitting multiple conversational steps in a single response instead of one at a time. Finding W is a step-merging failure; Finding V is a duplication failure.  

**Functional impact:** Low (states correctly captured in this run). UX impact: High (user cannot distinguish states vs. criteria input context).  

**Suggested fix:** Enforce strict sequential prompting in Phase 5: collect states in one response, wait for user reply, then begin criteria builder in the next response. Explicitly prohibit emitting criteria suggestions until states have been confirmed.

---

### Finding X — Phase 3 instructions loaded multiple times (triple load) [R1a]

**Phase:** 3 | **Severity:** Medium | **Frequency:** 1/11 (Run 11) — may be present at lower severity in earlier runs  
**Observed:** Phase 3 resource (`phase-3-create-delivery-method`) was loaded 3 times in a single run: (1) after Phase 2 summarization handoff, (2) after user selected "Specific hours only", (3) after user selected "Webhook". Each reload was triggered by the agent receiving a button-click input and reloading the resource instead of treating the already-loaded instructions as active context.  
**Expected:** Phase 3 resource should be loaded once. Subsequent user interactions within Phase 3 should be handled using the already-loaded instructions.  
**Root cause:** `summarize_history` strips prior messages including the first `get_resource` result, re-exposes the `<next_instructions>Load Phase 3</next_instructions>` pointer in the summary, and the agent re-executes it on each subsequent turn.  

**Suggested fix:** After a phase resource is loaded, include a guard in the summary state preventing re-load: e.g., `phase3Loaded=true`. Phase resource load should be a one-time action, not re-triggered per user interaction.

---

### Finding Y — Typed criterion input ignored after criteria recommendation card [R1a]

**Phase:** 5 | **Severity:** High | **Frequency:** 1/11 (Run 11)  
**Observed:** After the criteria recommendation card was shown ("Would you like to add criteria, see more fields, or skip?"), user typed "credit rating excellent" as a free-text criterion. The agent processed the turn, re-showed the same criteria recommendation card without parsing the input as a criterion, and returned to waiting state. No enum dropdown appeared; no criterion was added.  
**Expected:** Per UX scenario 5.6a and Phase 5 Step 7 instructions: "If user types criterion directly (e.g., 'credit rating excellent') → parse it via Criteria Parsing." Agent should have matched "credit rating" to `SelfCreditRating` (isEnumerated=true), shown the enum dropdown, and collected the value.  
**Root cause (from agent DEBUG):** Agent treated typed free-text input as a non-card-action and re-presented the card instead of attempting Criteria Parsing. The instruction to parse typed input was not prioritized over the card redisplay logic.  

**Agent-suggested patch:**  
- Add rule to Steps 7 and 8: "If user's message is free text and is not one of the card actions (Show more fields / Skip / Continue), always attempt Criteria Parsing first."  
- "Typed criterion text must be parsed immediately, appended to criteriaPayload, and then the loop must re-prompt."  
- "Do not redisplay the criteria card unless the user explicitly selects Show more fields or Skip, or the typed input cannot be matched."  

**Also observed:** `get_usa_states` was called in the SAME agent turn as `get_lead_type` (during the states-collection turn), pre-computing the state UIDs before the criteria loop. This is correct behavior for building the state criterion but may leave `criteriaPayload` in a partially-initialized state before the criteria loop begins.

---

### W1 — No delivery type card rendered (GPT-5-mini) [R1b]

**Run 21:** No delivery type card rendered after 24/7 schedule selection. Agent responded "Please choose one of the delivery options using the card above" but no card appeared. Resolved by typing "Webhook" as text fallback (accepted by agent).

---

### W2 — Content type card rendered correctly (GPT-5-mini) [R1b]

**Run 21:** Content type card (URL Encoded / JSON / XML) DID render correctly after webhook URL provided.

---

### W3 — Connection test double-prompt (GPT-5-mini) [R1b]

**Run 21:** Connection test double-prompt — agent first showed conversational "type run to proceed or cancel to pause", then after typing "run", showed Test Connection/Skip card. Two prompts for one operation.

---

### W4 — Content type card not rendered after webhook URL (GPT-5-mini) [R1b]

**Run 23:** After the webhook URL was accepted, the agent froze for ~3 minutes without rendering the content type ActionSet card (Adaptive/JSON/URL-Encoded/XML). Text fallback "XML" unblocked the agent immediately. Agent accepted it and proceeded to ask for XML schema.

**Run 26 (W4-variant):** Content type card did NOT appear at the expected point (after webhook URL). Instead, agent jumped directly to "Would you like to configure field mappings?" with "I'll provide instructions" / "Skip for now". After clicking "I'll provide instructions", the agent froze for ~3.5 minutes. Instructions were typed into the text box, which unblocked the agent. Content type card then appeared AFTER instructions, not before. Functionally, URL-Encoded was correctly selected and method created as HttpPost.

**W4-variant note:** Different from A23 (where content type froze and text fallback was needed to set type). In A26, content type was not asked at all at first; agent went straight to field mapping. After instructions were provided, content type was then asked. Possible root cause: agent batched the "instructions" + "content type" steps in wrong order, or a long tool call (process_webhook_instructions or get_lead_type) blocked the response for 3.5 minutes before content type prompt could be generated.

---

### RA-NEW-1 — Posting instructions prompt skipped [R2]

**Run 01:** After selecting JSON content type, agent jumped directly to "Field Mapping Preview" without asking user to paste the JSON schema. STOP AND YIELD at State 2 Step 3 was not enforced. Agent acknowledged via DEBUG: "I moved forward prematurely to the mapping steps instead of stopping to collect the required ASK input (postingInstructions)." Self-corrected after DEBUG — asked for schema, parsed leniently, showed correct mapping table. Pattern: Silent Tool Call Skipping (failure pattern #1).

---

### RA-NEW-2 — Phase 5 batching — criteria gate auto-skipped [R2]

**Run 01:** Agent showed criteria gate card with "Add criteria" | "Skip" buttons but did not wait for user response. Instead, when DEBUG message was sent as text, agent interpreted it as a non-button response and auto-proceeded to create the account with no criteria. The criteria gate → account creation → P6 summary all happened in a single response batch. Pattern: Batch Processing (failure pattern #3).

---

### RA-NEW-3 — States re-prompted when full state names given [R2]

**Run 02:** Agent asked "Which states do you want to target?" and user provided "New York, California, Texas, Florida, Illinois". Instead of normalizing to USPS codes, agent re-asked the same question. When abbreviations (NY, CA, TX, FL, IL) were provided the second time, it proceeded. The normalizer does not handle full state names.

---

### RA-NEW-4 — "Please type Continue to proceed" after Add criteria [R2]

**Run 02:** After user clicked "Add criteria" on the criteria gate, agent responded with "Please type Continue to proceed." instead of loading the criteria builder. User typed "Continue", which the agent interpreted as a directive to skip criteria entirely and jump to the final Client Setup Summary. Criteria collection bypassed completely. Account summary (P6-SUMM) also skipped.

---

### RA-NEW-5 — Phase 5c silent skip — criteria builder loaded but ran without user interaction [R2]

**Run 03:** After "Add criteria" click, agent called `get_resource(rw-phase-5c-criteria-builder)`. Phase 5c resource was loaded. In the single AI turn that followed, 0 `display_adaptive_card` calls were made, 0 user messages were exchanged — agent went directly to `create_delivery_account` with only the state criterion. Phase 5c State 1 (field suggestions) was never shown. Confirmed via audit log: between Phase 5c load and `create_delivery_account`, there was 1 AI turn, 0 function call blocks, 0 user messages. Account created with state criterion (NY|CA|FL = UIDs 33|5|10) only. Distinct from RA-NEW-4 (Run 02 — Phase 5c never loaded at all); here it loaded but executed silently.

---

### RA-NEW-6 — States question shown inside adaptive card with criteria gate buttons [R2]

**Run 03:** Phase 5 showed "Which states do you want to target? (e.g., CA, AZ, TX)" as a TextBlock inside an adaptive card, accompanied by "Add criteria" | "Skip" buttons in the same message [35]. States should be collected as a plain conversational prompt with no buttons. After user typed "NY, CA, FL", the criteria gate appeared again correctly in [37]. States were normalized and persisted correctly (NY=33, CA=5, FL=10), so functional impact was low but UX was confusing (related to R1 Finding W).

---

### RA-NEW-7 — Phase 5 prompt hallucination post-summarization [R2]

**Run 04:** After clicking Continue on the P4 delivery method summary, the agent entered Phase 5 but ignored Phase 5 instructions entirely. Instead of asking for price → states → exclusivity → order → criteria gate, the agent improvised a completely wrong prompt: "Please provide the delivery account details for Phase 5: FTP user / FTP password / If you want a different account type or setup details, include those too." This prompt was also doubled (RA-G). After DEBUG, agent correctly acknowledged the error ("I should not have implied those credentials were necessarily required for your webhook method") and proposed a patch (branch Phase 5 by delivery type — FTP credentials only for FTP-based methods). Agent then continued improvising with "account name, webhook username/password" questions instead of returning to Phase 5 flow. Root cause: post-summarization, agent lost access to Phase 5 MCP resource instructions and hallucinated delivery-account prompts from general knowledge. Entire Phase 5 structured flow (EXCL card → ORDER card → states prompt → criteria gate) never executed. Account was created correctly only because data was injected via a single combined message ("Price is $30, states CA and TX, not exclusive, no order system"). JSON→XML content switch worked correctly during Phase 3 — criteria gate completely bypassed.
**Patch suggested by agent:** "Phase 5 must branch by delivery method type. FTP credentials are required only when the delivery method is FTP-based. For HttpPost/webhook delivery methods, suppress all FTP credential prompts and collect only webhook-relevant account information, if any." (Applicable to stabilized Phase 5 only — rework has separate phase files per type.)

**Run 07 (2nd occurrence):** RA-D: Phase 2/3 regression loop after "I'll provide instructions" click (LT dropdown re-shown ×2, schedule re-asked); RA-NEW-7: Phase 5 FTP hallucination (2nd R2 occurrence, doubled).

---

### RB-X — Empty card after field mapping choice [R2]

**Run 02:** When user clicked "I'll provide instructions" on the field mapping card, the agent rendered an empty (blank) adaptive card bubble with no content. Nudging with "please continue" caused the agent to skip to the content type selection card. The posting instructions prompt then appeared AFTER content type was selected (inverted order). Recovery happened but sequence was wrong.

---

### RA-NEW-R2-8 — Criteria builder bypassed after 2nd "Add criteria" click [R2]

**Run 05:** After states question was skipped (RA-P), criteria gate appeared. Rather than clicking, user DEBUGged. After DEBUG recovery, states "CA, TX, FL, NY" were provided and a SECOND criteria gate appeared. User clicked "Add criteria" on the second criteria gate, but the agent created the account immediately with "Additional Criteria: None" — no criteria builder cards appeared, no user interaction took place. Pattern similar to RA-NEW-4 (Run 02) and RA-NEW-5 (Run 03) but caused by double criteria gate confusion after DEBUG session. The extended DEBUG session (2 DEBUG exchanges) may have disrupted the agent's phase state, causing it to treat the second "Add criteria" response as insufficient context to load the criteria builder. Account created with only state criterion (CA=5, TX=44, FL=10, NY=33).

---

### RA-NEW-R2-9 — Post-creation states message causes indefinite spin → session timeout [R2]

**Run 06:** After RA-P silently created the delivery account (blank Target States), user sent "Before I answer the criteria question — which states do you want to target? CA, TX, NY, FL" — a message aimed at the criteria gate but received after account creation. The agent began processing (spinner active) and never completed. After ~10 minutes the page navigated away from `/App/#/` to `/#/`, destroying the session. Likely cause: agent attempted to call `get_usa_states` for state matching but the tool call timed out or the MCP session was orphaned. Phase 7 (client summary) and Phase 8 (activation) were not reached.

---

## Run Findings (Detailed Narratives)

### Run 01

**Started:** 2026-04-04  
**Company:** StabilityTest-01  
**Email:** stability01@test.com

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Agent correctly opened the flow and prompted for company name and email.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — Card rendered correctly with lead type dropdown.
- **Finding A** (typed fallback): When lead type is typed as text instead of selected via card, agent asks for UID instead of matching by name.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: Y** — Schedule accepted. deliveryDays NOT built correctly (Finding C).
- **P3-WURL: Y** — Webhook URL accepted, field mapping card shown.
- **Content type: plain text** (no ActionSet card — Finding E).
- **P3-JSON: Y** — Lenient parsing worked (single quotes, missing braces, trailing comma fixed).
- **P3-TABLE: Y** — Field mapping preview rendered as Adaptive Card Table with 3 columns.
- **P3-COUNT: FAIL** — Initially 1/8; after DEBUG nudge re-mapped to 2/8. LoanAmount missed on first pass (Finding F).

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared exactly once.
- Test ran successfully (200 OK), but success message silently swallowed (Finding G).
- Actual tool call per session log: **Portal (HttpGet)**, not Webhook (Finding I).

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — "Delivery Method Created" card appeared with correct structure.
- Company name wrong: StabilityTest-50 instead of StabilityTest-01 (Finding H).
- Delivery Hours: None (confirms Finding C — deliveryDays all false).

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — $37.50 accepted.
- **P5-EXCL: Y** — "Exclusive" accepted (plain text, no card).
- **P5-ORDER: Y** — "No" accepted, Order System disabled.
- **P5-STATE / P5-NORM: FAIL** — States stored as "California, Texas, Florida" not normalized to CA/TX/FL (Finding J).
- **P5-FIELD / P5-CR1 / P5-ENUM / P5-CR3 / P5-DONE: FAIL** — Criteria phase entirely skipped; went straight to account summary (Finding K). Root cause: post-summarization context loss.

#### Phase 6 — Account Summary

- **P6-SUMM: Y** — Account summary card appeared.
- **P6-BOOL: Y** — Exclusive and Order System shown correctly in summary.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Client Setup Summary card appeared with Activate / Keep Inactive buttons.
- Delivery Method and Delivery Account both show ID: 46887 — same ID = one hallucinated (Finding I extended).

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now ACTIVE for StabilityTest-50."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | Card rendered; typed text fallback broken (Finding A) |
| P3-SCHED | Y | deliveryDays not built correctly (Finding C) |
| P3-WURL | Y | |
| P3-JSON | Y | Lenient parsing worked |
| P3-TABLE | Y | Correct Adaptive Card Table |
| P3-COUNT | F | 1/8 initial; 2/8 after DEBUG nudge (Finding F) |
| P3B-TEST | Y | Appeared once; success msg swallowed (Finding G) |
| P4-SUMM | Y | Wrong company name StabilityTest-50 (Finding H) |
| P5-PRICE | Y | |
| P5-EXCL | Y | Plain text, no card |
| P5-ORDER | Y | |
| P5-STATE | F | Not normalized (Finding J) |
| P5-NORM | F | Same as STATE |
| P5-FIELD | F | Criteria phase skipped (Finding K) |
| P5-CR1 | F | Skipped |
| P5-ENUM | F | Skipped |
| P5-CR3 | F | Skipped |
| P5-DONE | F | Skipped |
| P6-BOOL | Y | |
| P6-SUMM | Y | |
| P7-SUMM | Y | Same ID for method+account (Finding I) |
| P8-ACT | Y | |

---

### Run 02

**Started:** 2026-04-04  
**Company:** StabilityTest-02  
**Email:** stability02x2859@test.com *(stability02@test.com already in use)*

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Flow opened correctly; company name and email collected.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — Lead type dropdown card rendered correctly. Selected "Health Insurance" via card.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: Y** — Schedule accepted correctly. deliveryDays built correctly (Finding C NOT reproduced).
- **P3-WURL: Y** — Webhook URL accepted.
- **Content type: 4-button ActionSet card** appeared correctly (Finding E NOT reproduced — nondeterministic; card appeared this run).
- **P3-JSON: Y** — Lenient parsing worked.
- **P3-TABLE: Y** — Field mapping table rendered correctly.
- **P3-COUNT: Y** — "2 out of 3 fields mapped" (denominator: 3 lead type system fields). Different denominator from Run 01 (1/8) — nondeterministic calculation.

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test appeared once and ran.

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — Summary card appeared with correct company name StabilityTest-02 (Finding H NOT reproduced). IDs: Method 46888.

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — $37.50 accepted.
- **P5-EXCL: Y** — "Exclusive" accepted.
- **P5-ORDER: Y** — Order System disabled.
- **P5-STATE / P5-NORM: FAIL** — States not normalized. "California, Texas, Florida" stored as-is (Finding J reproduced, now 2/3).
- **P5-FIELD: Y** — Lead field suggestions appeared (criteria phase NOT skipped — Finding K NOT reproduced this run).
- **P5-CR1: Y** — First criterion entered.
- **P5-ENUM: FAIL** — Enum criterion handling failed (Finding M, 1/3).
- **P5-CR3: Y** — Third criterion entered.
- **P5-DONE: FAIL** — Typed "done" to exit; agent stored "done" as a criterion value instead of exiting (Finding L, 1/3).

#### Phase 6 — Account Summary

- **P6-BOOL: Y** — Boolean fields shown correctly.
- **P6-SUMM: Y** — Account summary card appeared.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Client Setup Summary appeared with correct separate IDs: Method 46888, Account 45892.

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now ACTIVE for StabilityTest-02."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | Card rendered; selected via card (Finding A not triggered) |
| P3-SCHED | Y | deliveryDays built correctly |
| P3-WURL | Y | |
| P3-JSON | Y | Lenient parsing worked |
| P3-TABLE | Y | |
| P3-COUNT | Y | 2/3 (denominator = lead type fields); nondeterministic vs Run 01 |
| P3B-TEST | Y | |
| P4-SUMM | Y | Correct company name; Finding C/H not reproduced |
| P5-PRICE | Y | |
| P5-EXCL | Y | |
| P5-ORDER | Y | |
| P5-STATE | F | Not normalized (Finding J — 2/3) |
| P5-NORM | F | Same |
| P5-FIELD | Y | Criteria phase appeared (Finding K not reproduced) |
| P5-CR1 | Y | |
| P5-ENUM | F | Enum handling failed (Finding M — 1/3) |
| P5-CR3 | Y | |
| P5-DONE | F | "done" stored as criterion (Finding L — 1/3) |
| P6-BOOL | Y | |
| P6-SUMM | Y | |
| P7-SUMM | Y | Correct separate IDs |
| P8-ACT | Y | |

---

### Run 03

**Started:** 2026-04-04  
**Company:** StabilityTest-03  
**Email:** stability03x3141@test.com *(stability03@test.com already in use)*

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Flow opened correctly; company name and email collected.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — Lead type card rendered. Selected "Health Insurance" via dropdown select.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: FAIL** — After clicking "Specific hours only", agent did not ask for hours — jumped directly to delivery method selection (Finding N, sub-issue 1).
- **P3-WURL: Y** — Webhook URL accepted.
- **Content type: 4-button ActionSet card** (Finding E not reproduced — 3rd run confirms card is nondeterministic, not a stable failure).
- **P3-JSON: FAIL** — After "I'll provide instructions" + JSON selected, agent never asked for JSON body. Delivery method created with 0 field mappings (Finding N, sub-issue 2).
- **P3-TABLE: FAIL** — No mapping table shown (consequence of N).
- **P3-COUNT: FAIL** — 0 of 0 fields mapped.

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared. Proceeded via text ("move forward — test the connection").

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — Summary card appeared, correct company name, correct delivery type HttpPost. Delivery Hours: "Specific hours only" (no hours collected). Field Mappings: 0 of 0. Method ID: 46889.

#### Phase 5 — Account Setup + Criteria (first pass)

- **P5-PRICE: Y** — $37.50 accepted.
- **P5-EXCL: Y** — "Exclusive" accepted.
- **P5-ORDER: Y** — Order System disabled.
- **P5-STATE / P5-NORM: FAIL** — "California, Texas, Florida" not normalized (Finding J — 3/3).
- **P5-FIELD: Y** — Criteria suggestions shown.
- **P5-CR1: Y** — LoanAmount >= 50000 accepted.
- **P5-ENUM: Y** — LoanType In dropdown shown with valid values; "Refinance" selected and accepted (Finding M NOT reproduced — enum handled correctly this run).
- **P5-CR3: Y** — FirstName is not empty accepted.
- **P5-DONE: FAIL** — Typed "continue" to exit criteria loop → agent showed Phase 4 "Delivery Method Created" card again → clicking Continue restarted Phase 5 from scratch; all criteria lost (Finding O, 1/3).

#### Phase 5 (second pass — after restart)

- Re-entered: 37.50 / Exclusive / No order system / California, Texas, Florida
- Criteria: Skipped (to complete flow)
- Account created with Additional Criteria: None

#### Phase 6 — Account Summary

- **P6-SUMM: Y** — Account summary card appeared.
- **P6-BOOL: Y** — Exclusive and Order System shown correctly.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Client Setup Summary appeared. Method ID: 46889, Account ID: 45894 — distinct IDs (Finding I not reproduced).

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now ACTIVE for StabilityTest-03."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | |
| P3-SCHED | F | Hours not collected after "Specific hours only" (Finding N) |
| P3-WURL | Y | |
| P3-JSON | F | JSON body never requested (Finding N) |
| P3-TABLE | F | Consequence of N |
| P3-COUNT | F | 0 of 0 mapped |
| P3B-TEST | Y | |
| P4-SUMM | Y | 0 of 0 field mappings; correct company name |
| P5-PRICE | Y | |
| P5-EXCL | Y | |
| P5-ORDER | Y | |
| P5-STATE | F | Not normalized (Finding J — 3/3) |
| P5-NORM | F | Same |
| P5-FIELD | Y | Criteria phase appeared |
| P5-CR1 | Y | |
| P5-ENUM | Y | Enum handled correctly — dropdown with valid values (Finding M not reproduced) |
| P5-CR3 | Y | |
| P5-DONE | F | "continue" triggered Phase 4 card regression + Phase 5 restart (Finding O) |
| P6-BOOL | Y | |
| P6-SUMM | Y | |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | |

---

### Run 04

**Started:** 2026-04-05  
**Company:** StabilityTest-04  
**Email:** stability04-01@test.com  
**Scenario:** LendingTree, Webhook, JSON (15-field large body), Mon-Fri 9am-5pm PST, 5 criteria (numeric + enum + text), Exclusive, Order ON

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — No doubling; flow opened normally.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected via dropdown card.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: FAIL** — After "Specific hours only" selected, agent did not ask for actual hours. Delivery method card appeared immediately with "Delivery Hours: Specific hours only" (no actual hours captured). *Finding N sub-issue 1 reproduced — 2/4.*
- **P3-WURL: FAIL** — Webhook URL was never requested before field mapping. Agent jumped from delivery type selection directly to asking for posting instructions. *New Finding S.*
- **Content type card:** 4-button ActionSet card appeared correctly.
- **P3-JSON: Y** — Selected JSON; agent immediately showed "I couldn't parse this as valid JSON" card without asking for the schema. Recovered by clicking "Re-paste JSON schema" → agent then requested the body → pasted 15-field JSON. *New Finding T.*
- **P3-TABLE: Y** — Field mapping table rendered with 14/14 system fields shown.
- **P3-COUNT: Y** — 14 of 14 fields mapped.

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared. Chose Skip.

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — "Delivery Method Created" card appeared. Hours shown as "Specific hours only" (no actual hours). Method ID: 46909.

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — Price per lead $30 accepted.
- **P5-EXCL: Y** — Exclusive accepted.
- **P5-ORDER: Y** — Order System enabled (Yes).
- **P5-STATE: FAIL** — States were never asked at all. Flow jumped directly from Order System to account summary. *New Finding P (≈ RB-N in Version B).*
- **P5-NORM: FAIL** — Consequence of states being skipped; phantom CA,AZ,TX appeared in P6.
- **P5-FIELD / P5-CR1 / P5-ENUM / P5-CR3 / P5-DONE: FAIL** — Criteria gate bypassed entirely; account created with "Additional Criteria: None". *New Finding R (≈ RB-D in Version B).*

#### Phase 6 — Account Summary

- **P6-BOOL: Y** — Exclusive and Order System: Enabled shown correctly.
- **P6-SUMM: FAIL** — Target States shows "CA,AZ,TX" — phantom states never entered by user. *New Finding Q (≈ RB-O in Version B).* Account ID: 45912.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Client Setup Summary with separate IDs: Delivery Method 46909, Delivery Account 45912.

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-04."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | F | Hours not collected after "Specific hours only" (Finding N — 2/4) |
| P3-WURL | F | Webhook URL never asked (Finding S — 1/4) |
| P3-JSON | Y | Parse error before schema provided, recovered via re-paste (Finding T — 1/4) |
| P3-TABLE | Y | 14/14 fields shown |
| P3-COUNT | Y | 14/14 mapped |
| P3B-TEST | Y | Skip worked |
| P4-SUMM | Y | Hours: "Specific hours only"; Method ID 46909 |
| P5-PRICE | Y | $30 |
| P5-EXCL | Y | |
| P5-ORDER | Y | |
| P5-STATE | F | States never asked (Finding P — 1/4) |
| P5-NORM | F | Phantom CA,AZ,TX |
| P5-FIELD | F | Criteria gate bypassed (Finding R — 1/4) |
| P5-CR1 | F | Skipped |
| P5-ENUM | F | Skipped |
| P5-CR3 | F | Skipped |
| P5-DONE | F | Skipped |
| P6-BOOL | Y | |
| P6-SUMM | F | Phantom CA,AZ,TX states (Finding Q — 1/4) |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | |

---

### Run 05

**Started:** 2026-04-05  
**Company:** StabilityTest-05  
**Email:** stability05-01@test.com  
**Scenario:** LendingTree, Webhook, URL-Encoded, 24/7, Shared, Order OFF; 3 enum-heavy criteria

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Flow opened normally; text prompt, no doubling.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected via dropdown.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: Y** — "24/7" accepted; schedule processed correctly.
- **P3-WURL: Y** — Webhook URL card appeared; URL submitted via textarea.
- **Content type:** URL Encoded selected correctly.
- **P3-JSON: Y** — URL-Encoded body provided and parsed correctly.
- **P3-TABLE: Y** — Field mapping table rendered with 10/10 fields shown.
- **P3-COUNT: Y** — 10/10 fields mapped correctly.

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared. Skip chosen.

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — "Delivery Method Created" card shown. Method ID: 46910.

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — Price accepted.
- **P5-EXCL: Y** — Shared selected.
- **P5-ORDER: Y** — Order System: No (disabled).
- **P5-STATE: Y** — State prompt appeared; FL, OH, GA entered and normalized correctly. *Finding V: state prompt text doubled (two identical blocks in same message).*
- **P5-NORM: Y** — States normalized to FL, OH, GA UIDs correctly.
- **P5-FIELD: Y** — Criteria builder appeared with LendingTree field suggestions.
- **P5-CR1: FAIL** — Enum criteria added via dropdown UI (LoanRequestType = REFINANCE; LoanAmount; AnnualIncome attempted). *Finding U: criteria not persisted to account — Additional Criteria: None in P6.*
- **P5-ENUM: Y** — Enum dropdown appeared and accepted value.
- **P5-CR3 / P5-DONE: FAIL** — Criteria accumulation failed; "Continue" via textarea triggered account creation discarding all criteria.

#### Phase 6 — Account Summary

- **P6-BOOL: Y** — Shared and Order System: Disabled shown correctly.
- **P6-SUMM: Y** — Account summary correct; states FL, OH, GA shown; Additional Criteria: None. Account ID: 45913.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Client Setup Summary with distinct IDs: Method 46910, Account 45913.

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now ACTIVE for StabilityTest-05."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | 24/7 accepted |
| P3-WURL | Y | |
| P3-JSON | Y | URL-Encoded parsed |
| P3-TABLE | Y | 10/10 shown |
| P3-COUNT | Y | 10/10 mapped |
| P3B-TEST | Y | Skip worked |
| P4-SUMM | Y | Method ID 46910 |
| P5-PRICE | Y | |
| P5-EXCL | Y | Shared |
| P5-ORDER | Y | |
| P5-STATE | Y | FL, OH, GA collected; prompt doubled (Finding V) |
| P5-NORM | Y | Normalized correctly |
| P5-FIELD | Y | Criteria builder appeared |
| P5-CR1 | F | Not persisted (Finding U) |
| P5-ENUM | Y | Enum dropdown worked |
| P5-CR3 | F | Not persisted |
| P5-DONE | F | Additional Criteria: None |
| P6-BOOL | Y | |
| P6-SUMM | Y | Account ID 45913 |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 06

**Started:** 2026-04-05  
**Company:** StabilityTest-06  
**Email:** stability06-01@test.com  
**Scenario:** LendingTree, Webhook, XML auto-detect ("I'm not sure"), Weekdays 8am-6pm EST, Exclusive, Order OFF; 4 criteria attempted

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Flow opened normally; no doubling.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected via dropdown.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: Y** — Schedule hours card appeared correctly (Finding N NOT reproduced). After clicking Webhook, *Finding D REPRODUCED (2/6)*: schedule card reappeared. Schedule provided again; delivery type card reappeared; second Webhook click proceeded to URL prompt.
- **P3-WURL: Y** — Webhook URL card appeared; "https://httpbin.org/post" sent via textarea.
- **Content type:** "I'm not sure" sent → agent asked for posting instructions → XML body pasted → agent auto-detected XML correctly → "Continue with XML" confirmed.
- **P3-JSON: Y** — XML parsed correctly, auto-detect identified format accurately.
- **P3-TABLE: Y** — Field mapping table rendered with 11/11 fields shown and all mapped.
- **P3-COUNT: Y** — 11/11 fields mapped (FirstName, LastName, EmailAddress, ContactPhone, ContactAddress, ContactCity, ContactState, ContactZip, LoanAmount, SelfCreditRating, PropertyValue, LoanRequestPurpose).

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared; test ran successfully. Method created.

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — "Delivery Method Created" card shown. Weekdays 8am-6pm EST, 11/11 fields. Method ID: 46911.

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — Price per lead $40 accepted. *Note: price prompt text doubled (two identical lines in one message) — Finding V variant.*
- **P5-EXCL: Y** — Exclusive selected.
- **P5-ORDER: Y** — Order System: No (disabled).
- **P5-STATE: FAIL** — States prompt silently skipped. Flow went directly from Order=No to criteria builder. *Finding P reproduced (2/6).*
- **P5-NORM: FAIL** — States skipped; no normalization occurred.
- **P5-FIELD: Y** — Criteria builder appeared with field suggestions.
- **P5-CR1: Y** — "add criteria: ContactState" → enum dropdown appeared for ContactState; FL selected (value "10") and Submit clicked → returned "add another criterion" card. Target States: 10 (FL) saved.
- **P5-ENUM: Y** — Enum dropdown worked for ContactState; value submitted via card Submit.
- **P5-CR3: FAIL** — "add criteria: LoanAmount" returned no numeric input card (went back to criteria menu). Inline "LoanAmount >= 100000" triggered account creation skipping remaining criteria.
- **P5-DONE: FAIL** — Account created with Additional Criteria: None. *Finding U reproduced (2/6).*

#### Phase 6 — Account Summary

- **P6-BOOL: Y** — Exclusive and Order System: Disabled shown correctly.
- **P6-SUMM: Y** — Account summary shown. Target States: 10 (FL from ContactState enum). Additional Criteria: None. Account ID: 45914.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Client Setup Summary with distinct IDs: Method 46911, Account 45914.

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-06."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | Card appeared; Finding D reproduced (regression, 2/6) |
| P3-WURL | Y | After 2nd Webhook click |
| P3-JSON | Y | XML auto-detect worked |
| P3-TABLE | Y | 11/11 shown |
| P3-COUNT | Y | 11/11 mapped |
| P3B-TEST | Y | Connection test ran |
| P4-SUMM | Y | Method ID 46911 |
| P5-PRICE | Y | $40; prompt text doubled |
| P5-EXCL | Y | |
| P5-ORDER | Y | |
| P5-STATE | F | States silently skipped (Finding P — 2/6) |
| P5-NORM | F | Skipped |
| P5-FIELD | Y | Criteria builder appeared |
| P5-CR1 | Y | ContactState enum dropdown → FL set as target state |
| P5-ENUM | Y | Enum dropdown + Submit worked |
| P5-CR3 | F | LoanAmount not added (numeric field path missing) |
| P5-DONE | F | Additional Criteria: None (Finding U — 2/6) |
| P6-BOOL | Y | |
| P6-SUMM | Y | Account ID 45914; Target States: 10 (FL) |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 07

**Started:** 2026-04-05  
**Company:** StabilityTest-07  
**Email:** stability07-01@test.com  
**Scenario:** LendingTree, Webhook, JSON (complex nested multi-level body), Mon/Wed/Fri 8am-6pm CST, 6 criteria attempted, Shared, Order ON

#### Protocol note
First attempt loaded wrong instructions (generic agent — asked for phone number, didn't respond to DEBUG). Fixed by clicking the "Create Single Client (Original)" action card in the UI rather than sending the action text via textarea.

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Stabilized instructions loaded via action card click; standard "Company Name + Contact Email" prompt.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected.

#### Phase 3 — Schedule + Webhook + Field Mapping

- **P3-SCHED: Y** — Schedule card appeared; "Specific hours only" → hours prompt appeared correctly. *Finding N NOT reproduced.*
- **P3-WURL: Y** — No phase regression after Webhook click. *Finding D NOT reproduced.* Webhook URL prompt appeared correctly.
- **P3-JSON: Y** — Complex nested JSON body provided (4-level: lead.contact.*, lead.address.*, lead.loan.*, lead.meta.*). Agent correctly extracted all leaf fields.
- **P3-TABLE: Y** — Field mapping table rendered with 13/13 fields shown.
- **P3-COUNT: Y** — 13/13 mapped (nested path resolution worked correctly).

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared; test ran. Method created.

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — "Delivery Method Created" card. Hours: Mon/Wed/Fri 8am-6pm CST ✓. Mappings: 13/13. Method ID: 46912.

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — $35 accepted; prompt appeared once (not doubled).
- **P5-EXCL: Y** — Shared accepted.
- **P5-ORDER: Y** — Order System: Yes (Enabled).
- **P5-STATE: Y** — States prompt appeared correctly. *Finding P NOT reproduced.* FL, CA, TX, NY, IL entered and saved.
- **P5-NORM: Y** — States shown as "FL, CA, TX, NY, IL" in P6 summary correctly.
- **P5-FIELD: Y** — Criteria builder appeared with LendingTree recommendations.
- **P5-CR1: Y** — "add criteria: LoanRequestPurpose" → "Please select a value" text prompt → "Purchase" accepted → returned to add-another card.
- **P5-ENUM: Y** — "add criteria: Bankruptcy" → text prompt then dropdown appeared → NEVER selected via JS native setter + Submit click.
- **P5-CR3: FAIL** — Criteria not persisted to account. *Finding U reproduced (3/7).* 3 criteria attempted (LoanRequestPurpose=Purchase, SelfCreditRating=Good, Bankruptcy=NEVER), all acknowledged by agent but none in P6.
- **P5-DONE: FAIL** — Additional Criteria: None in P6.

#### Phase 6 — Account Summary

- **P6-BOOL: Y** — Shared and Order System: Enabled shown correctly.
- **P6-SUMM: Y** — Target States: FL, CA, TX, NY, IL ✓; Additional Criteria: None. Account ID: 45915.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Distinct IDs: Method 46912, Account 45915.

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-07."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | Stabilized instructions via action card click |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | Mon/Wed/Fri 8am-6pm CST parsed correctly; Finding N NOT reproduced |
| P3-WURL | Y | Finding D NOT reproduced |
| P3-JSON | Y | Nested JSON 4-level deep |
| P3-TABLE | Y | 13/13 shown |
| P3-COUNT | Y | 13/13 mapped |
| P3B-TEST | Y | Connection test ran |
| P4-SUMM | Y | Method ID 46912; hours correct |
| P5-PRICE | Y | $35, not doubled |
| P5-EXCL | Y | Shared |
| P5-ORDER | Y | Enabled |
| P5-STATE | Y | FL,CA,TX,NY,IL; Finding P NOT reproduced |
| P5-NORM | Y | Normalized correctly |
| P5-FIELD | Y | Criteria builder appeared |
| P5-CR1 | Y | LoanRequestPurpose=Purchase added |
| P5-ENUM | Y | Bankruptcy=NEVER dropdown worked |
| P5-CR3 | F | Criteria not persisted (Finding U — 3/7) |
| P5-DONE | F | Additional Criteria: None |
| P6-BOOL | Y | |
| P6-SUMM | Y | Account ID 45915; states correct |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 08

**Started:** 2026-04-05  
**Company:** StabilityTest-08  
**Email:** stability08-01@test.com  
**Scenario:** LendingTree, FTP delivery, 24/7, Exclusive, Order OFF, 2 criteria, states FL/CA/TX

#### Protocol note
Action card clicked directly (not text-typed) to load stabilized instructions — same fix as Run 07.

#### Phase 1 — Create Client

- **P1-PROMPT†: Y** — Stabilized instructions loaded; standard company name + email prompt.

#### Phase 2 — Lead Type

- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected via dropdown card.

#### Phase 3 — Schedule + FTP Details

- **P3-SCHED: Y** — Schedule card appeared; 24/7 selected. *Finding N NOT reproduced.*
- **P3-WURL: Y** — FTP branch: agent correctly asked for FTP server, username, password (not webhook URL). FTP details: ftp.test-server.com / testuser / testpass123.
- **P3-JSON/TABLE/COUNT: Y** — N/A for FTP delivery (no field mapping required).

#### Phase 3b — Connection Test

- **P3B-TEST: Y** — Connection test prompt appeared; Skip chosen.

#### Phase 4 — Method Summary

- **P4-SUMM: Y** — "Delivery Method Created" card. Method name: StabilityTest-08-FTP. 0 of 0 mappings (FTP). Method ID: 46913.

#### Phase 5 — Account Setup + Criteria

- **P5-PRICE: Y** — $25 accepted.
- **P5-EXCL: Y** — Exclusive.
- **P5-ORDER: Y** — Disabled.
- **P5-STATE: Y** — States prompt appeared; "FL, CA, TX" entered. States correctly captured in P6 summary despite *Finding W* (merged prompt — see below).
- **P5-NORM: Y** — States shown as FL, CA, TX in P6 (correct abbreviations).
- **Finding W (new):** States prompt and criteria recommendations appeared in the **same agent response** without waiting for state input. Agent showed "Which states do you want to target?" immediately followed by criteria field suggestions. "FL, CA, TX" was entered as a response to this merged prompt. States were correctly interpreted and saved — functional outcome OK but UX is confusing (criteria follow-up expected).
- **P5-FIELD: Y** — Criteria builder appeared with LendingTree field suggestions.
- **P5-CR1: Y** — "add criteria: LoanRequestPurpose" → enum dropdown appeared → PURCHASE selected via JS native setter + Submit. Accepted.
- **P5-ENUM: Y** — "add criteria: SelfCreditRating" → enum dropdown → GOOD selected. Accepted. "add another" card appeared.
- **P5-CR3: FAIL** — "Continue" sent to exit criteria loop. Only SelfCreditRating appears in P6 summary; LoanRequestPurpose=PURCHASE missing; no criterion values shown. *Finding U reproduced (4/8).*
- **P5-DONE: FAIL** — Criteria not fully persisted.

#### Phase 6 — Account Summary

- **P6-BOOL: Y** — Exclusive and Order System shown correctly.
- **P6-SUMM: Y** — Target States: FL, CA, TX ✓; Additional Criteria: SelfCreditRating (partial — Finding U). Account ID: 45916.

#### Phase 7 — Client Summary

- **P7-SUMM: Y** — Distinct IDs: Method 46913, Account 45916.

#### Phase 8 — Activation

- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-08."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | Stabilized via action card click |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | 24/7; Finding N NOT reproduced |
| P3-WURL | Y | FTP server/user/pass correctly requested |
| P3-JSON | Y | N/A (FTP) |
| P3-TABLE | Y | N/A (FTP) |
| P3-COUNT | Y | N/A (FTP) |
| P3B-TEST | Y | Skip worked |
| P4-SUMM | Y | Method ID 46913; FTP; 0 mappings |
| P5-PRICE | Y | $25 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Disabled |
| P5-STATE | Y | FL,CA,TX; Finding P NOT reproduced |
| P5-NORM | Y | Correct abbreviations |
| P5-FIELD | Y | Criteria builder appeared; Finding W (merged states+criteria prompt) |
| P5-CR1 | Y | LoanRequestPurpose=PURCHASE added via dropdown |
| P5-ENUM | Y | SelfCreditRating=GOOD dropdown worked |
| P5-CR3 | F | LoanRequestPurpose=PURCHASE not in P6 (Finding U — 4/8) |
| P5-DONE | F | Criteria partially lost |
| P6-BOOL | Y | |
| P6-SUMM | Y | States correct; criteria partial |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 09

**Started:** 2026-04-05  
**Company:** StabilityTest-09  
**Email:** stability09-01@test.com  
**Scenario:** LendingTree, Portal delivery, Mon-Fri 9am-5pm PST, Exclusive, Order OFF, 3 criteria, states NY/OH/TX

#### Protocol note
Action card clicked. Agent timed out after LendingTree selection (response was reset after ~2 min). Recovered by sending "continue" — agent resumed correctly with P3 schedule card.

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Stabilized instructions loaded via action card.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree 5689 selected via dropdown.

#### Phase 3 — Schedule + Portal
- **P3-SCHED: Y** — "Specific hours only" → hours prompt appeared. *Finding N NOT reproduced.* Schedule: Mon-Fri 9am-5pm PST.
- **P3-WURL/JSON/TABLE/COUNT: Y** — N/A for Portal (no webhook or field mapping).
- **Delivery type:** Portal selected from 4-button card.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — No connection test prompted (correctly skipped for Portal).

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — Portal method created. Hours: Monday through Friday, 9am to 5pm PST ✓. Method ID: 46914.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $20, not doubled.
- **P5-EXCL: Y** — Exclusive.
- **P5-ORDER: Y** — No (Disabled).
- **P5-STATE: Y** — States prompt appeared alone. NY, OH, TX entered. *Finding P and W NOT reproduced.*
- **P5-NORM: Y** — NY, OH, TX correctly shown in P6.
- **P5-FIELD: Y** — Criteria builder appeared.
- **P5-CR1: Y** — LoanRequestPurpose=REFINANCE via dropdown.
- **P5-ENUM: Y** — SelfCreditRating=EXCELLENT, Bankruptcy=NEVER via dropdowns. All 3 accepted.
- **P5-CR3: FAIL** — All 3 criteria discarded. *Finding U (5/9).*
- **P5-DONE: FAIL** — Additional Criteria: None in P6.

#### Phase 6 — Account Summary
- **P6-BOOL: Y**
- **P6-SUMM: Y** — Target States: NY, OH, TX ✓; Additional Criteria: None. Account ID: 45917.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Distinct IDs: Method 46914, Account 45917.

#### Phase 8 — Activation
- **P8-ACT: Y** — ACTIVE.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689; agent timeout recovered with nudge |
| P3-SCHED | Y | Mon-Fri 9-5 PST; Finding N NOT reproduced |
| P3-WURL | Y | N/A (Portal) |
| P3-JSON | Y | N/A (Portal) |
| P3-TABLE | Y | N/A (Portal) |
| P3-COUNT | Y | N/A (Portal) |
| P3B-TEST | Y | Correctly skipped for Portal |
| P4-SUMM | Y | Method ID 46914 |
| P5-PRICE | Y | $20 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Disabled |
| P5-STATE | Y | NY,OH,TX; Finding P NOT reproduced |
| P5-NORM | Y | Correct |
| P5-FIELD | Y | Criteria builder appeared |
| P5-CR1 | Y | LoanRequestPurpose=REFINANCE |
| P5-ENUM | Y | SelfCreditRating=EXCELLENT, Bankruptcy=NEVER |
| P5-CR3 | F | All 3 criteria discarded (Finding U — 5/9) |
| P5-DONE | F | Additional Criteria: None |
| P6-BOOL | Y | |
| P6-SUMM | Y | States correct |
| P7-SUMM | Y | Distinct IDs |
| P8-ACT | Y | ACTIVE |

---

### Run 10

**Started:** 2026-04-05  
**Company:** StabilityTest-10  
**Email:** stability10-01@test.com  
**Scenario:** LendingTree, Email delivery, Mon-Fri 9am-5pm PST, Exclusive, Order ON, 3 enum criteria (limited by available UI), states GA/WA/CO

#### Protocol note
Action card clicked. No timeout this run.

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Stabilized instructions loaded.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree 5689 selected.

#### Phase 3 — Schedule + Email
- **P3-SCHED: Y** — "Specific hours only" → hours prompt. Schedule: Mon-Fri 9am-5pm PST. *Finding N NOT reproduced.*
- **P3-WURL/JSON/TABLE/COUNT: Y** — N/A for Email.
- **Delivery type:** Email selected from 4-button card.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Correctly skipped for Email.

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — Email method (StabilityTest-10-Email, Delivery Type: EMail). Hours: Mon-Fri 9am-5pm PST ✓. Method ID: 46915.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $45.
- **P5-EXCL: Y** — Exclusive.
- **P5-ORDER: Y** — Yes (Enabled).
- **P5-STATE: Y** — GA, WA, CO. *Finding P NOT reproduced.*
- **P5-NORM: Y** — States shown as abbreviations.
- **P5-FIELD: Y** — Criteria builder appeared with 47 more fields available.
- **P5-CR1: Y** — LoanRequestPurpose=PURCHASE via dropdown.
- **P5-ENUM: Y** — SelfCreditRating=GOOD, Bankruptcy=NEVER via dropdowns.
- **Additional finding:** Foreclosure, WorkingWithAgent, ResidenceType, PurchaseTimeFrame, LoanType, EmploymentStatus all return to add-another menu with no input card — silently bypass criteria collection for these fields.
- **P5-CR3: FAIL** — All 3 criteria discarded. *Finding U (6/10).*
- **P5-DONE: FAIL** — Additional Criteria: None in P6.

#### Phase 6 — Account Summary
- **P6-BOOL: Y**
- **P6-SUMM: Y** — Target States: GA, WA, CO ✓; Additional Criteria: None; Order System: Enabled ✓. Account ID: 45918.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Distinct IDs: Method 46915, Account 45918.

#### Phase 8 — Activation
- **P8-ACT: Y** — ACTIVE.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | Mon-Fri 9-5 PST |
| P3-WURL | Y | N/A (Email) |
| P3-JSON | Y | N/A (Email) |
| P3-TABLE | Y | N/A (Email) |
| P3-COUNT | Y | N/A (Email) |
| P3B-TEST | Y | Correctly skipped for Email |
| P4-SUMM | Y | Method ID 46915; type EMail |
| P5-PRICE | Y | $45 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Enabled |
| P5-STATE | Y | GA,WA,CO |
| P5-NORM | Y | |
| P5-FIELD | Y | Criteria builder; many fields lack input cards |
| P5-CR1 | Y | LoanRequestPurpose=PURCHASE |
| P5-ENUM | Y | SelfCreditRating=GOOD, Bankruptcy=NEVER |
| P5-CR3 | F | All 3 criteria discarded (Finding U — 6/10) |
| P5-DONE | F | Additional Criteria: None |
| P6-BOOL | Y | |
| P6-SUMM | Y | States correct; Order System Enabled ✓ |
| P7-SUMM | Y | Distinct IDs |
| P8-ACT | Y | ACTIVE |

---

---

### Run 11

**Started:** 2026-04-05
**Company:** StabilityTest-11
**Email:** stability11-01@test.com
**Scenario:** LendingTree, Webhook/JSON, Weekdays 8am-8pm Eastern, Exclusive, Order ON

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Stabilized instructions loaded; company name + email prompt appeared.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected.

#### Phase 3 — Schedule + Webhook + Field Mapping
- **P3-SCHED: Y** — Schedule hours prompt appeared; "Weekdays 8am to 8pm Eastern" accepted.
- **P3-WURL: Y** — Webhook URL accepted (https://hooks.example.com/leads/stability11).
- **Finding X REPRODUCED:** Phase 3 instructions loaded 3× — after P2 summary, after "Specific hours only" click, after "Webhook" click. Each button-click triggered a `get_resource` reload of phase-3 (root cause: summarize_history re-exposes `<next_instructions>` pointer).
- **P3-JSON: FAIL — Finding N2:** After "I'll provide instructions" + "JSON" content type selected, agent **skipped the posting instructions prompt entirely** and called `create_delivery_method` with `requestBody=null`. UX scenario 3W.6c violated. 0/0 fields mapped.
- **P3-TABLE: FAIL** — No mapping table (0 fields).
- **P3-COUNT: FAIL** — 0/0.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Connection test prompt appeared for Webhook (correctly triggered). Skip selected.

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "0 of 0 fields mapped" shown (consequence of N2). Method ID: 46916.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $35 accepted.
- **P5-EXCL: Y** — Exclusive.
- **P5-ORDER: Y** — Order ON; states prompt appeared correctly (**Finding P NOT reproduced ✓**).
- **P5-STATE: Y** — FL, NC, SC entered and accepted.
- **P5-NORM: Y** — States normalized correctly.
- **P5-FIELD: Y** — Criteria recommendation card appeared.
- **Finding Y REPRODUCED:** Typed "credit rating excellent" was ignored — agent re-showed criteria card without parsing the input as a criterion or displaying enum dropdown. Root cause (from agent DEBUG): agent does not attempt Criteria Parsing on free-text input when criteria card is active; only button-clicks are processed.
- **P5-CR1/ENUM/CR3: FAIL** — No criterion added (typed input ignored; Skip used to proceed).
- **P5-DONE: Y** — Skip button clicked → account created.

#### Phase 6 — Account Summary
- **P6-SUMM: Y** — Target States: FL, NC, SC ✓; Additional Criteria: None; Order System: Enabled ✓. Account ID: 45919.
- **Session log audit:** `create_delivery_account` criteria payload = `[{"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"10|34|41"}]` — state criterion only (FL=10, NC=34, SC=41). Correct for no-criteria scenario.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Method ID 46916, Account ID 45919.

#### Phase 8 — Activation
- **P8-ACT: Y** — ACTIVE ✓.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | Hours prompt appeared; Finding N NOT reproduced |
| P3-WURL | Y | Webhook URL accepted |
| P3-JSON | F | Body prompt skipped; requestBody=null (Finding N2 — 1/11) |
| P3-TABLE | F | 0/0 fields (consequence of N2) |
| P3-COUNT | F | 0 |
| P3B-TEST | Y | Connection test shown for Webhook |
| P4-SUMM | Y | 0/0 fields shown in summary |
| P5-PRICE | Y | $35 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Order ON |
| P5-STATE | Y | FL,NC,SC; Finding P NOT reproduced |
| P5-NORM | Y | Correct |
| P5-FIELD | Y | Criteria gate appeared |
| P5-CR1 | F | Typed criterion ignored (Finding Y — 1/11) |
| P5-ENUM | F | No enum dropdown shown |
| P5-CR3 | F | No criteria added |
| P5-DONE | Y | Skip → account created |
| P6-BOOL | Y | |
| P6-SUMM | Y | States + Order correct |
| P7-SUMM | Y | Distinct IDs |
| P8-ACT | Y | ACTIVE |

---

---

### Run 12

**Started:** 2026-04-05
**Company:** StabilityTest-12
**Email:** stability12-03@test.com
**Scenario:** LendingTree, FTP, 24/7, Exclusive, Order ON, 1 criterion attempted

#### Protocol note
Session restored mid-run via history icon after a page navigation accident (wrong "Continue" button clicked on P4 summary navigated away from chat). P5 adaptive cards for Exclusive/Shared and Yes/No steps did not render — questions appeared as plain text, requiring text responses. Assessed as UI session-restore rendering artifact, not an instruction bug. Criteria builder adaptive card (Show more fields / Continue) did render correctly once criteria loop was reached.

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Stabilized instructions loaded correctly; company name + email collected.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree (LeadTypeUID: 5689) selected via dropdown.

#### Phase 3 — Schedule + FTP + Field Mapping
- **P3-SCHED: Y** — 24/7 selected correctly.
- **P3-WURL: Y** — FTP branch: agent asked for FTP server, username, password (ftp.test-server.com / testuser / testpass123).
- **P3-JSON/TABLE/COUNT: Y** — N/A for FTP (0/0 fields, no mapping required).

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Connection test prompt appeared; Skip chosen.

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "Delivery Method Created" card shown (0/0 FTP). Method ID: 46917.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $40 accepted.
- **P5-EXCL: Y** — "Exclusive" typed (no card button — session-restore artifact). Accepted correctly.
- **P5-ORDER: Y** — "Yes" typed (no card button). Accepted correctly.
- **P5-STATE: Y** — States prompt appeared correctly. TX, FL, CA entered and accepted. **Finding P NOT reproduced.**
- **P5-NORM: Y** — States normalized correctly in P6.
- **P5-FIELD: Y** — Criteria recommendations appeared (LendingTree fields). Note: recommended fields included FirstName, LastName, ContactAddress (contact fields — should be excluded per instruction; **instruction violation**).
- **P5-CR1: Y** — Typed "loan amount at least 100000" → agent processed it, moved to criteria loop (Show more fields / Continue card rendered). **Finding Y NOT reproduced** — typed criterion was parsed.
- **P5-ENUM: Y** — N/A (loan amount is numeric, not enumerated). Criterion acknowledged.
- **P5-CR3: FAIL** — Criterion dropped from `create_delivery_account` payload. **Finding U reproduced (7/11).** Payload: `[{"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"44|10|5"}]` — state criterion only.
- **P5-DONE: FAIL** — Additional Criteria shown as "loan amount at least 100000" in P6 summary text but actual payload has none.

#### Phase 6 — Account Summary
- **P6-BOOL: Y** — Lead Exclusivity: Exclusive ✓; Order System: Enabled ✓.
- **P6-SUMM: Y** — Target States: TX, FL, CA ✓; Additional Criteria shown in summary text. Account ID: 45920.
- **Session log audit:** `isExclusive=true`, `useOrder=true` ✓. Criteria payload = `[{leadFieldUID:144881, operator:"In", value:"44|10|5"}]` — states only.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Client Setup Summary: Method 46917, Account 45920.

#### Phase 8 — Activation
- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-12."

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | 24/7; Finding N NOT reproduced |
| P3-WURL | Y | FTP credentials collected |
| P3-JSON | Y | N/A (FTP) |
| P3-TABLE | Y | N/A (FTP) |
| P3-COUNT | Y | 0/0 FTP |
| P3B-TEST | Y | Skip |
| P4-SUMM | Y | Method ID 46917 |
| P5-PRICE | Y | $40 |
| P5-EXCL | Y | Exclusive (text input — card not rendered) |
| P5-ORDER | Y | Order ON (text input — card not rendered) |
| P5-STATE | Y | TX, FL, CA; Finding P NOT reproduced |
| P5-NORM | Y | Normalized correctly |
| P5-FIELD | Y | Criteria shown; contact fields in list (violation) |
| P5-CR1 | Y | Typed criterion parsed and acknowledged |
| P5-ENUM | Y | N/A (numeric field) |
| P5-CR3 | F | Criterion dropped from payload (Finding U — 7/11) |
| P5-DONE | F | Additional Criteria not persisted to API |
| P6-BOOL | Y | Exclusive + Order Enabled ✓ |
| P6-SUMM | Y | States correct; Account 45920 |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 13

**Started:** 2026-04-05  
**Company:** StabilityTest-13  
**Email:** stability13-01@test.com  
**Scenario:** LendingTree, Email delivery, Mon-Fri 8am-5pm CST, Exclusive, Order ON, 2 enum criteria (typed), states NY/OH/GA  
**Session:** `69d2099f2697c9202c0bc899`

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Flow opened; company name + email collected.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree (5689) selected via dropdown card.

#### Phase 3 — Schedule + Email
- **P3-SCHED: Y** — "Specific hours only" selected; agent asked for hours. **Finding V reproduced (3/13):** Schedule prompt appeared twice in same response after button click. Hours: Mon-Fri 8am-5pm CST provided; schedule accepted.
- **P3-WURL/JSON/TABLE/COUNT: Y** — N/A for Email delivery.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Correctly skipped for Email.

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "Delivery Method Created" card. Method ID: 46918.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $45 accepted.
- **P5-EXCL: Y** — Exclusive (card rendered, button clicked).
- **P5-ORDER: Y** — Order ON (card rendered, button clicked).
- **P5-STATE: Y** — States prompt appeared alone. NY, OH, GA entered. Finding P NOT reproduced.
- **P5-NORM: Y** — NY, OH, GA normalized correctly.
- **P5-FIELD: Y** — Criteria recommendations appeared. **Contact field violation (2nd confirmation):** Batch 1 listed TrackingNumber, RequestAssignmentDate, FirstName, LastName, ContactAddress; Batch 2 listed ContactCity, ContactState, ContactZip, EmailAddress, ContactPhone, ContactPhoneExtension, ConsumerGeoPhoneAreaCode, ConsumerGeoPhoneCountryCode, TimeToContact, DateOfBirth; Batch 3 listed SSN, IsMilitary, AssignedCreditValue, SelfCreditRating, Bankruptcy, Foreclosure…
- **P5-CR1: Y** — Typed "SelfCreditRating excellent" → agent parsed silently (no dropdown shown despite isEnumerated=true); accepted, criteria loop card shown.
- **P5-ENUM: Y** — Typed "LoanRequestPurpose refinance" → also accepted silently without dropdown. **New observation:** Enum fuzzy-match bypass — typed values >85% fuzzy matched skip the Input.ChoiceSet card entirely.
- **P5-CR3: FAIL** — Clicked Continue. **Finding U confirmed (8/12 runs with criteria).** Both SelfCreditRating and LoanRequestPurpose dropped from payload.
- **P5-DONE: FAIL** — Additional Criteria: None in P6.

#### Phase 6 — Account Summary
- **P6-BOOL: Y** — Exclusive + Order Enabled shown.
- **P6-SUMM: Y** — Target States: NY, OH, GA ✓; Additional Criteria: None. Account ID: 45921.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Distinct IDs: Method 46918, Account 45921.

#### Phase 8 — Activation
- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-13."

#### Payload Audit
```
create_delivery_account criteria=[{"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"33|36|11"}]
```
State UIDs: NY=33 ✓, OH=36 ✓, GA=11 ✓. SelfCreditRating and LoanRequestPurpose **absent** — Finding U confirmed.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | Mon-Fri 8am-5pm CST; Finding V (schedule doubled — 3rd/13th) |
| P3-WURL | Y | N/A (Email) |
| P3-JSON | Y | N/A (Email) |
| P3-TABLE | Y | N/A (Email) |
| P3-COUNT | Y | N/A (Email) |
| P3B-TEST | Y | Correctly skipped for Email |
| P4-SUMM | Y | Method ID 46918 |
| P5-PRICE | Y | $45 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Order ON |
| P5-STATE | Y | NY, OH, GA; Finding P NOT reproduced |
| P5-NORM | Y | Correct |
| P5-FIELD | Y | Criteria appeared; contact field exclusion violated (2nd confirmation) |
| P5-CR1 | Y | SelfCreditRating=excellent typed, parsed via fuzzy match, no dropdown |
| P5-ENUM | Y | LoanRequestPurpose=refinance typed, accepted silently — enum dropdown bypass |
| P5-CR3 | F | Both criteria dropped from payload (Finding U — 8/12) |
| P5-DONE | F | Additional Criteria: None |
| P6-BOOL | Y | |
| P6-SUMM | Y | States correct; Account 45921 |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 14

**Started:** 2026-04-05  
**Company:** StabilityTest-14  
**Email:** stability14-01@test.com  
**Scenario:** LendingTree, Webhook/JSON, 24/7, Shared, Order OFF; full state names input (deliberate — to test Finding J)  
**Session:** `69d20e762697c9202c0bc8dd`

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Flow opened; company name + email collected.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree (5689) selected via dropdown card.

#### Phase 3 — Schedule + Webhook + Mapping
- **P3-SCHED: Y** — 24/7 selected (no hours needed).
- **P3-WURL: Y** — Webhook URL accepted.
- **P3-JSON: Y** — **Finding V reproduced (4th run, 2nd instance in run):** JSON schema prompt ("Please paste the JSON schema...") appeared twice in same response. User pasted schema once; 5/5 fields mapped. New P3 variant of Finding V (previously only observed in P5-price and P3-schedule contexts).
- **P3-TABLE: Y** — Mapping table rendered with 5/5 fields.
- **P3-COUNT: Y** — 5/5 mapped.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Webhook connection test run.

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "Delivery Method Created" card. Method ID: 46919.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $30 accepted. **Finding V reproduced (same run, 1st instance):** Price/account setup prompt also doubled in same message.
- **P5-EXCL: Y** — Shared selected.
- **P5-ORDER: Y** — Order OFF selected.
- **P5-STATE: Y** — States prompt appeared; "New York, California, Texas" (full names) entered deliberately.
- **P5-NORM: FAIL** — **Finding J reproduced (display-only):** P6 account summary displayed "New York, California, Texas" un-normalized. However, `get_usa_states()` in Step 10 correctly resolved names → UIDs. API payload had correct UIDs (33|5|44 = NY|CA|TX). Finding J is **confirmed as display-only** — does not affect account functionality.
- **P5-FIELD: Y** — Criteria suggestions appeared. Contact field exclusion violated (same pattern as Runs 12–13).
- **P5-CR1/ENUM/CR3/DONE: Y** — User selected Skip; criteria loop not entered. Finding U N/A.

#### Phase 6 — Account Summary
- **P6-BOOL: Y** — Shared + Order Disabled shown.
- **P6-SUMM: Y** — Account ID: 45922; Target States shown un-normalized ("New York, California, Texas") — Finding J display artifact.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Distinct IDs: Method 46919, Account 45922.

#### Phase 8 — Activation
- **P8-ACT: FAIL** — Platform `activate_client` tool failed 3 consecutive times; agent re-displayed "We encountered an issue activating the client. Would you like to try again?" after each attempt. Not a flow bug — platform tool error. StabilityTest-14 left inactive.

#### Payload Audit
```
create_delivery_account criteria=[{"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"33|5|44"}]
```
State UIDs: NY=33 ✓, CA=5 ✓, TX=44 ✓. Despite full-name display failure in P6, API received correct UIDs. Finding J is **display-only**.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | 24/7 |
| P3-WURL | Y | |
| P3-JSON | Y | Completed; Finding V (JSON schema prompt doubled) |
| P3-TABLE | Y | 5/5 fields |
| P3-COUNT | Y | 5/5 |
| P3B-TEST | Y | Webhook test run |
| P4-SUMM | Y | Method ID 46919 |
| P5-PRICE | Y | $30 accepted; Finding V (price prompt also doubled) |
| P5-EXCL | Y | Shared |
| P5-ORDER | Y | Order OFF |
| P5-STATE | Y | Full names accepted |
| P5-NORM | F | P6 displayed full names "New York, California, Texas" — Finding J (display-only) |
| P5-FIELD | Y | Criteria appeared; user skipped |
| P5-CR1 | Y | N/A — skipped |
| P5-ENUM | Y | N/A — skipped |
| P5-CR3 | Y | N/A — skipped |
| P5-DONE | Y | N/A — skipped |
| P6-BOOL | Y | |
| P6-SUMM | Y | Account 45922; states shown un-normalized |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | F | Platform activate_client failure (3 retries) |

---

### Run 15

**Started:** 2026-04-05  
**Company:** StabilityTest-15  
**Email:** stability15-01@test.com  
**Scenario:** LendingTree, Webhook/JSON, Mon-Fri 9am-5pm CST, Exclusive, Order OFF; 1 typed numeric criterion (loan amount at least 50000); states FL/GA/TN intended (not reached — Finding P)  
**Session:** `69d213222697c9202c0bc92e`

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Flow opened; company name + email collected.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree (5689) selected via dropdown card.

#### Phase 3 — Schedule + Webhook + Mapping
- **Finding V (new location, 5th run):** After lead type selection, delivery method name prompt ("Please provide the delivery method name.") appeared twice in same response.
- **P3-SCHED: Y** — "Specific hours only" → Mon-Fri 9am-5pm CST. Schedule prompt appeared once (no doubling).
- **P3-WURL: Y** — Webhook URL accepted. No phase regression (Finding D not reproduced).
- Content type card rendered: JSON selected.
- **P3-JSON: Y** — JSON schema prompt appeared once (no doubling). Schema provided; 8/8 fields mapped.
- **P3-TABLE: Y, P3-COUNT: Y** — 8/8 mapped: FirstName, LastName, EmailAddress, ContactPhone, LoanAmount, ContactState, SelfCreditRating, LoanRequestPurpose.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Connection test ran; failed as expected for test URL (result shown visibly, not swallowed).

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "Delivery Method Created" card. Method ID: 46920.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $35 accepted, appeared once (no doubling).
- **P5-EXCL: Y** — Exclusive selected via card.
- **P5-ORDER: Y** — Order OFF selected.
- **P5-STATE: FAIL** — **Finding P reproduced (3rd occurrence, 3/15 runs).** States prompt never appeared after Order System selection. Agent jumped directly to criteria recommendations.
- **P5-NORM: FAIL** — **Finding Q reproduced.** Phantom CA (stateUID=5) inserted as state criterion despite no states collected. No states shown in P6 summary.
- **P5-FIELD: Y** — Criteria recommendations appeared with contact field exclusion violated (TrackingNumber, RequestAssignmentDate, FirstName, LastName, ContactAddress in batch 1).
- **P5-CR1: Y** — Typed "loan amount at least 50000" → parsed correctly as LoanAmount GreaterOrEqual 50000.
- **P5-ENUM/CR3/DONE: Y** — Clicked Continue; criteria loop exited.
- **P5-CR3: PASS (first ever!)** — **Finding U NOT reproduced.** LoanAmount criterion persisted in `create_delivery_account` payload.

#### Phase 6 — Account Summary
- **P6-BOOL: Y** — Exclusive + Order Disabled shown.
- **P6-SUMM: Y** — Additional Criteria: "loan amount at least 50000" ✓ (first time a criterion is shown in P6 summary). Target States: empty (Finding P).

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Distinct IDs: Method 46920, Account 45923.

#### Phase 8 — Activation
- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-15."

#### Payload Audit
```
create_delivery_account criteria=[
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"5"},
  {"leadFieldUID":144924,"type":"FieldValue","operator":"GreaterOrEqual","value":"50000"}
]
```
State UID 5 = CA (phantom — no states were collected). LoanAmount criterion **present** — first ever criterion persistence. Finding U NOT reproduced.

**Key observation:** Criteria persisted WHEN Finding P also reproduced. In all runs 05–13 where states were collected normally, criteria were dropped (Finding U). This run suggests Finding U and Finding P may share a common root cause — the summarize_history mechanism may strip criteria context when state UIDs are properly resolved.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689 |
| P3-SCHED | Y | Mon-Fri 9am-5pm CST; single prompt (no doubling) |
| P3-WURL | Y | No phase regression |
| P3-JSON | Y | 8/8 fields mapped; schema prompt once |
| P3-TABLE | Y | |
| P3-COUNT | Y | 8/8 |
| P3B-TEST | Y | Failed as expected; result visible |
| P4-SUMM | Y | Method ID 46920 |
| P5-PRICE | Y | $35; single prompt |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Order OFF |
| P5-STATE | F | Finding P — states prompt skipped |
| P5-NORM | F | Finding Q — phantom CA (5) in state criterion |
| P5-FIELD | Y | Contact field exclusion violated (again) |
| P5-CR1 | Y | LoanAmount ≥ 50000 parsed correctly |
| P5-ENUM | Y | N/A |
| P5-CR3 | Y | **PASS** — criterion in payload (Finding U NOT reproduced) |
| P5-DONE | Y | |
| P6-BOOL | Y | |
| P6-SUMM | Y | Additional Criteria shown ✓; Account 45923 |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

---

### Run 16

**Started:** 2026-04-05  
**Company:** StabilityTest-16  
**Email:** stability16-01@test.com  
**Scenario:** LendingTree, Webhook/JSON, Mon-Fri 9am-5pm PST, Shared, Order OFF; 1 typed numeric criterion (loan amount at least 100000); states OR, WA  
**Session:** `69d218d92697c9202c0bc97f`

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Flow opened; company name + email collected (required 3 messages — first two were sent separately and agent asked for company name again).

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree selected via JS (UID 5689) + Continue.

#### Phase 3 — Schedule + Webhook + Field Mapping
- **P3-SCHED: Y** — "Mon-Fri 9am-5pm PST" accepted, no doubling.
- **P3-WURL: Y** — Webhook URL accepted.
- **Finding D reproduced (new trigger location):** After clicking "I'll provide instructions", agent regressed to "Please select a Lead Type for this client." — regression back to Phase 2 after a non-Webhook card click. Previously D was only observed after the Webhook delivery type button click. 3rd occurrence overall.
- **Recovery:** Provided full JSON body directly as text. Agent recovered and mapped 7/7 fields.
- **P3-JSON: Y** — 7/7 fields mapped (FirstName, LastName, EmailAddress, ContactPhone, ContactState, LoanAmount, SelfCreditRating).
- **P3-TABLE: Y** — Field mapping table rendered correctly.
- **P3-COUNT: Y** — 7/7.

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Skipped (test endpoint).

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "Delivery Method Created": StabilityTest-16-Webhook, HttpPost, Mon-Fri 9am-5pm PST, 7/7 mapped. Method ID: 46921.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $45 accepted, no doubling.
- **P5-EXCL: Y** — "Shared" typed (no adaptive card rendered at this step).
- **P5-ORDER: Y** — "No" typed.
- **P5-STATE: Y** — "OR, WA" collected. Finding P NOT reproduced.
- **P5-NORM: Y** — OR/WA already abbreviations; no normalization needed.
- **P5-FIELD: Y** — Criteria suggestion shown. Contact field exclusion VIOLATED (TrackingNumber, RequestAssignmentDate, FirstName, LastName, ContactAddress in top 5). Same persistent violation as runs 12–15.
- **P5-CR1: Y** — "loan amount at least 100000" parsed as LoanAmount GreaterOrEqual 100000.
- **P5-ENUM: Y** — N/A (no enum fields tested).
- **P5-CR3: Y** — **PASS** — criterion in payload (Finding U NOT reproduced).
- **P5-DONE: Y** — "continue" accepted, loop exited.

#### Phase 6 — Account Summary
- **P6-BOOL: Y** — Lead Exclusivity: Shared, Order System: Disabled shown correctly.
- **P6-SUMM: Y** — Target States: OR, WA; Additional Criteria: "loan amount at least 100000" ✓. Account 45924.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — All IDs correct: Method 46921, Account 45924.

#### Phase 8 — Activation
- **P8-ACT: Y** — ACTIVE.

#### Payload Audit
```
create_delivery_account
clientUID: 29331
criteria: [
  {leadFieldUID: 144881, type: "FieldValue", operator: "In", value: "38|48"},
  {leadFieldUID: 144924, type: "FieldValue", operator: "GreaterOrEqual", value: "100000"}
]
```
State UIDs: 38=OR, 48=WA ✓. LoanAmount criterion persisted ✓.

#### Scorecard

| Check | Result | Notes |
|-------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | |
| P3-SCHED | Y | No doubling |
| P3-WURL | Y | |
| P3-JSON | Y | |
| P3-TABLE | Y | |
| P3-COUNT | Y | 7/7 |
| P3B-TEST | Y | Skipped |
| P4-SUMM | Y | Method 46921 |
| P5-PRICE | Y | $45 |
| P5-EXCL | Y | Shared (typed) |
| P5-ORDER | Y | No (typed) |
| P5-STATE | Y | OR, WA |
| P5-NORM | Y | |
| P5-FIELD | Y | Contact exclusion violated (again) |
| P5-CR1 | Y | LoanAmount ≥ 100000 parsed |
| P5-ENUM | Y | N/A |
| P5-CR3 | Y | **PASS** — criterion in payload |
| P5-DONE | Y | |
| P6-BOOL | Y | |
| P6-SUMM | Y | Additional Criteria shown ✓ |
| P7-SUMM | Y | |
| P8-ACT | Y | ACTIVE |

**RESULT: FAIL** — Finding D reproduced (new trigger location: "I'll provide instructions" card click → Phase 2 regression); contact field exclusion violated (3rd+ consecutive run).

**Critical finding: Finding U NOT reproduced for 2nd consecutive run** (A15 and A16). A16 states were properly collected (OR, WA → UIDs 38|48), yet LoanAmount criterion persisted in payload. This disproves the U/P correlation hypothesis — U is nondeterministic, not tied to Finding P. U reproduced in 8/10 qualifying runs overall (80%).

---

### Run 17

**Started:** 2026-04-05  
**Company:** StabilityTest-17  
**Email:** stability17-01@test.com  
**Scenario:** LendingTree, FTP/24/7, Exclusive, Order OFF; 1 enum criterion (SelfCreditRating=EXCELLENT); planned states ID/MT  
**Session:** `69d221662697c9202c0bc9d0`

#### Phase 1 — Create Client
- **P1-PROMPT†: Y** — Agent asked for company name and email together. No doubling.

#### Phase 2 — Lead Type
- **P2-DROP: Y** — LendingTree (5689) selected via dropdown card. No Finding D triggered.

#### Phase 3 — Schedule + FTP + Field Mapping
- **P3-SCHED: Y** — 24/7 selected.
- **P3-WURL: Y** — FTP server/username/password accepted (ftp.test17.com, ftpuser17).
- **P3-JSON / P3-TABLE / P3-COUNT: Y** — FTP delivery; no field mapping step. P4 summary showed "0 of 0 fields mapped" (expected for FTP).

#### Phase 3b — Connection Test
- **P3B-TEST: Y** — Connection test prompt appeared; Skip chosen.

#### Phase 4 — Method Summary
- **P4-SUMM: Y** — "Delivery Method Created" card. Method ID: 46922.

#### Phase 5 — Account Setup + Criteria
- **P5-PRICE: Y** — $30 accepted.
- **P5-EXCL: Y** — Exclusive selected.
- **P5-ORDER: Y** — Order System OFF (No).
- **P5-STATE: FAIL** — **Finding P reproduced (4th occurrence).** After Order System selection, agent jumped directly to criteria builder without asking for target states. No states prompt shown to user.
- **P5-NORM: FAIL** — States never collected (consequence of Finding P).
- **P5-FIELD: Y** — Criteria suggestion card appeared: TrackingNumber, RequestAssignmentDate, FirstName, LastName, ContactAddress, ContactCity. Contact field exclusion violated again.
- **P5-CR1: Y** — Typed "SelfCreditRating Excellent"; agent matched SelfCreditRating as enumerated field.
- **P5-ENUM: Y** — Enum dropdown showed: NOTPROVIDED, EXCELLENT, GOOD, FAIR, POOR. Selected EXCELLENT (UID 343593). Submit worked.
- **P5-CR3: PASS** — SelfCreditRating=EXCELLENT persisted in `create_delivery_account` payload (UID 343593). **Finding U NOT reproduced (3rd consecutive non-reproduction: A15, A16, A17).**
- **P5-DONE: Y** — Criteria loop exited on "Continue" click.

#### Phase 6 — Account Summary
- **P6-BOOL: Y** — Exclusive + Order Disabled shown.
- **P6-SUMM: Y** — Additional Criteria: "SelfCreditRating Excellent" ✓. Target States: **blank** (P/Q artifact). Account ID: 45925.

#### Phase 7 — Client Summary
- **P7-SUMM: Y** — Client Setup Summary: Method 46922, Account 45925.

#### Phase 8 — Activation
- **P8-ACT: Y** — "✓ Setup complete. Your lead delivery system is now 'ACTIVE' for StabilityTest-17."

#### Payload Audit
```
create_delivery_account criteria=[
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"5"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"}
]
```
- State criterion: UID 5 = **CA (California)** — **Finding Q confirmed.** User never provided states; planned states were ID (UID 13) and MT (UID 27). Agent ran `get_usa_states()` anyway and hallucinated CA. The P6 display showed blank for Target States while the payload contained CA=5.
- SelfCreditRating criterion: UID 343593 = EXCELLENT ✓ — persisted correctly.

#### Checkpoint Results

| Checkpoint | Result | Notes |
|-----------|--------|-------|
| P1-PROMPT† | Y | |
| P2-DROP | Y | LendingTree 5689; no Finding D |
| P3-SCHED | Y | 24/7 |
| P3-WURL | Y | FTP details accepted |
| P3-JSON | Y | N/A (FTP) |
| P3-TABLE | Y | N/A (FTP) — 0 of 0 |
| P3-COUNT | Y | N/A (FTP) |
| P3B-TEST | Y | Skip |
| P4-SUMM | Y | Method ID 46922 |
| P5-PRICE | Y | $30 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | Off |
| P5-STATE | F | **Finding P (4th occurrence)** — states prompt skipped |
| P5-NORM | F | States never collected |
| P5-FIELD | Y | Contact exclusion violated (continuing pattern) |
| P5-CR1 | Y | SelfCreditRating typed + matched |
| P5-ENUM | Y | Dropdown showed correct values; EXCELLENT selected |
| P5-CR3 | Y | **PASS — criterion persisted (U not reproduced, 3rd consecutive)** |
| P5-DONE | Y | Continue worked |
| P6-BOOL | Y | |
| P6-SUMM | Y | Criteria shown; Target States blank (P/Q artifact) |
| P7-SUMM | Y | Distinct IDs |
| P8-ACT | Y | ACTIVE |

**RESULT: FAIL** — Finding P + Q reproduced. Contact field exclusion violated. P5-CR3 PASS (U not reproduced, 3rd consecutive).

---

### Run 18

**Started:** 2026-04-05
**Session:** 69d225372697c9202c0bca0f
**Company:** StabilityTest-18
**Email:** stability18-01@test.com
**Scenario:** LendingTree, Email delivery, Mon-Fri 9am-5pm PST, Shared, Order ON, 2 enum criteria (typed)

#### Scorecard

| Check | Result | Notes |
|-------|--------|-------|
| P1-PROMPT† | Y | Client flow opened correctly |
| P2-DROP | Y | LendingTree selected |
| P3-SCHED | Y | Mon-Fri 9am-5pm PST |
| P3-WURL | Y | N/A — Email delivery |
| P3-JSON | Y | N/A |
| P3-TABLE | Y | N/A |
| P3-COUNT | Y | N/A |
| P3B-TEST | Y | Skip (Email delivery) |
| P4-SUMM | Y | Method ID 46923 |
| P5-PRICE | Y | $25 |
| P5-EXCL | Y | Shared |
| P5-ORDER | Y | On |
| P5-STATE | F | **Finding P (5th occurrence)** — states prompt skipped after Order System selection |
| P5-NORM | F | States never collected |
| P5-FIELD | Y | Contact exclusion violated (continuing pattern) |
| P5-CR1 | Y | LoanRequestPurpose typed, fuzzy-matched |
| P5-ENUM | Y | Value captured; **enum bypass** — no dropdown shown for either enumerated field |
| P5-CR3 | Y | **PASS — both criteria persisted (U NOT reproduced, 4th consecutive)** |
| P5-DONE | Y | Continue worked |
| P6-BOOL | Y | |
| P6-SUMM | Y | Criteria shown; Target States blank (P/Q artifact) |
| P7-SUMM | Y | |
| P8-ACT | Y | ACTIVE |

#### Payload Audit

```
criteria=[
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"5"},
  {"leadFieldUID":144921,"type":"FieldValue","operator":"In","value":"343609"},
  {"leadFieldUID":144894,"type":"FieldValue","operator":"In","value":"343607"}
]
```

- UID 144881 = state field → value "5" = CA (phantom — Finding P/Q; user never provided states)
- UID 144921 → LoanRequestPurpose, value 343609 = PURCHASE (typed, fuzzy-matched without dropdown)
- UID 144894 → Bankruptcy, value 343607 = NEVER (typed, fuzzy-matched without dropdown)
- Both non-state criteria persisted → **Finding U NOT reproduced (4th consecutive)**

**RESULT: FAIL** — Finding P + Q (5th occurrence). Both criteria persisted (U streak continues at 4). Enum bypass confirmed for both typed criteria. Contact field exclusion violated.

---

### Run 19

**Started:** 2026-04-05
**Session:** 69d229422697c9202c0bca55
**Company:** StabilityTest-19
**Email:** stability19-02@test.com
**Scenario:** LendingTree, Webhook/XML, 24/7, Exclusive, Order ON, 1 numeric criterion (LoanAmount ≥ 75000)

#### Scorecard

| Check | Result | Notes |
|-------|--------|-------|
| P1-PROMPT† | Y | Client flow opened correctly |
| P2-DROP | Y | LendingTree selected |
| P3-SCHED | Y | 24/7 |
| P3-WURL | Y | Webhook URL accepted |
| P3-JSON | Y | XML content type selected |
| P3-TABLE | Y | Field mapping preview rendered |
| P3-COUNT | Y | 9/9 fields mapped |
| P3B-TEST | Y | Connection test run (result silently swallowed — Finding G) |
| P4-SUMM | Y | Method ID 46924 |
| P5-PRICE | Y | $30 |
| P5-EXCL | Y | Exclusive |
| P5-ORDER | Y | On |
| P5-STATE | F | **Finding P (6th occurrence)** — states skipped after Order System Yes |
| P5-NORM | F | States never collected |
| P5-FIELD | Y | First 5 fields: TimeToContact, SelfCreditRating, Bankruptcy, LoanRequestType, AnnualIncome — no contact fields visible |
| P5-CR1 | Y | "loan amount at least 75000" typed and parsed correctly |
| P5-ENUM | Y | N/A — numeric field, no dropdown |
| P5-CR3 | F | **FAIL — criterion NOT persisted (U reproduced; breaks 4-run non-reproduction streak)** |
| P5-DONE | Y | Continue worked |
| P6-BOOL | Y | |
| P6-SUMM | Y | Additional Criteria showed "loan amount at least 75000" (display only — not in payload); Target States blank |
| P7-SUMM | Y | Distinct IDs |
| P8-ACT | Y | ACTIVE |

#### Payload Audit

```
criteria=[
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"5"}
]
```

- UID 144881 = state field → value "5" = CA (phantom — Finding P/Q; user never provided states)
- LoanAmount ≥ 75000 criterion: shown in P6 display but **absent from payload** — Finding U reproduced
- 4-run non-reproduction streak (A15–A18) broken

**RESULT: FAIL** — Finding P + Q (6th occurrence). Finding U reproduced after 4-run streak. LoanAmount criterion shown in summary but dropped at account creation.

---

### Run 20

**Started:** 2026-04-05
**Session:** 69d22bb72697c9202c0bcaa0
**Company:** StabilityTest-20
**Email:** stability20-01@test.com
**Scenario:** LendingTree, Portal, Mon-Fri 9am-5pm PST, Shared, Order OFF, 1 enum criterion (SelfCreditRating excellent)

#### Scorecard

| Check | Result | Notes |
|-------|--------|-------|
| P1-PROMPT† | Y | Client flow opened correctly |
| P2-DROP | Y | LendingTree selected |
| P3-SCHED | Y | "Specific hours only" selected; Mon-Fri 9am-5pm PST |
| P3-WURL | Y | N/A (Portal) |
| P3-JSON | Y | N/A (Portal) |
| P3-TABLE | Y | N/A (Portal) |
| P3-COUNT | Y | N/A (Portal) |
| P3B-TEST | Y | Correctly skipped for Portal |
| P4-SUMM | Y | Portal method: StabilityTest-20-Portal, Mon-Fri 9am-5pm PST, 0/0 mappings; Method ID: 46925 |
| P5-PRICE | Y | $20 |
| P5-EXCL | Y | Shared |
| P5-ORDER | Y | Off (No) |
| P5-STATE | F | **Finding P (7th occurrence)** — states skipped after Order System No |
| P5-NORM | F | States never collected (consequence of Finding P) |
| P5-FIELD | Y | Criteria builder appeared; contact field exclusion violated (TrackingNumber, RequestAssignmentDate, FirstName, LastName, ContactAddress in first 5) |
| P5-CR1 | Y | "SelfCreditRating excellent" typed; parsed correctly |
| P5-ENUM | Y | Enum bypass: accepted via fuzzy match without dropdown |
| P5-CR3 | Y | **PASS** — SelfCreditRating=EXCELLENT (UID 343593) in payload (Finding U NOT reproduced — 5th non-reproduction) |
| P5-DONE | Y | Continue worked |
| P6-BOOL | Y | Lead Exclusivity: Shared, Order System: Disabled |
| P6-SUMM | Y | Target States: blank (P/Q artifact); Additional Criteria: blank (display artifact — criterion WAS in payload) |
| P7-SUMM | Y | Distinct IDs confirmed |
| P8-ACT | Y | ACTIVE |

#### Payload Audit

```
create_delivery_account
clientUID: 29336
criteria=[
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"5"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"}
]
```

- UID 144881 = state field → value "5" = CA (phantom — Finding P/Q; user never provided states)
- UID 144893 → SelfCreditRating, value 343593 = EXCELLENT — **persisted correctly** despite P6 display showing blank Additional Criteria
- New observation: when Finding P occurs AND a criterion IS persisted, P6 still displays blank Additional Criteria. The `criteriaList` display string was not populated (likely because the summarize_history call received an empty criteria display list even though criteriaPayload was non-empty). This is a display-only artifact; the account was correctly created with the criterion.

**RESULT: FAIL** — Finding P (7th) + Finding Q. Finding U NOT reproduced (5th consecutive non-reproduction broken by A19, then resumed in A20). Enum bypass confirmed. Contact field exclusion violated. P6 Additional Criteria display blank despite criterion in payload (new display artifact observation).

---

### Run 21–30 Summary (R1b — Mixed Model)

> Runs 21–30 use GPT-5-mini for Phase 3 (Webhook delivery method) and Phase 5 (delivery account criteria). All other phases use GPT-5.4 Mini. Purpose: compare per-phase stability against the GPT-5.4-only baseline (runs 01–20).

#### Run 21

**Scenario:** LendingTree, Webhook/JSON, 24/7, Exclusive, Order ON, 2 enum criteria (SelfCreditRating + LoanRequestPurpose)  
**Session:** 69d233702697c9202c0bcada | **Method:** 46926 | **Acct:** 45929 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"operator":"In","value":"5|44"},
  {"leadFieldUID":144893,"operator":"In","value":"343593"},
  {"leadFieldUID":144922,"operator":"Equal","value":"purchase"}
]
```

**Observations:**

- **Finding P:** NOT reproduced — states prompt appeared; CA(5) + TX(44) both in payload ✓
- **Finding Q:** NOT reproduced — no phantom state
- **Finding U:** NOT reproduced — both additional criteria in payload ✓
- **Finding V:** NOT reproduced — no doubled prompts

**New GPT-5-mini behaviors (P3/P5 phases):**
- **W1 (new):** No delivery type card rendered after 24/7 schedule selection. Agent responded "Please choose one of the delivery options using the card above" but no card appeared. Resolved by typing "Webhook" as text fallback (accepted by agent).
- **W2 (new):** Content type card (URL Encoded / JSON / XML) DID render correctly after webhook URL provided.
- **W3 (new):** Connection test double-prompt — agent first showed conversational "type run to proceed or cancel to pause", then after typing "run", showed Test Connection/Skip card. Two prompts for one operation.

**Enum handling:**
- SelfCreditRating: no dropdown shown; agent directly accepted "excellent" and stored UID 343593 (correct value)
- LoanRequestPurpose: no dropdown shown; agent accepted "purchase" and stored raw string with operator "Equal" (should be "In"/enumUID if enumerated — possible mismatch on field UID 144922 vs 144921 seen in earlier runs)

**P6 display:** Additional Criteria shows "SelfCreditRating a..." (truncated — consistent with prior observation that P6 truncates long criteria strings)

**Contact field exclusion:** Criteria gate suggested RequestAssignmentDate, TimeToContact, AssignedCreditValue, SelfCreditRating, Bankruptcy — no personal contact fields in top 5 ✓

---

#### Run 22

**Scenario:** LendingTree, Portal, Mon-Fri 9am-5pm EST, Shared, Order OFF, 1 numeric criterion (LoanAmount ≥ 50000)  
**Session:** 69d2395b2697c9202c0bcb60 | **Method:** 46927 | **Acct:** 45930 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"33|36"},
  {"leadFieldUID":144924,"type":"FieldValue","operator":"GreaterOrEqual","value":"50000"}
]
```

**Observations:**

- **Finding P:** NOT reproduced — states prompt appeared; NY(33)+OH(36) in payload ✓
- **Finding Q:** NOT reproduced — no phantom state
- **Finding U:** NOT reproduced — LoanAmount criterion persisted ✓
- **Finding V: TRIPLED** — Schedule card appeared 3 times (new severity record vs 2x baseline); delivery type card appeared twice after schedule confirmation. Text fallbacks used for both: typed "Specific hours only, Monday-Friday 9am-5pm Eastern" and then "Portal" as direct chat messages — both accepted correctly.

**New GPT-5-mini behaviors in A22:**
- **Timezone normalization error:** "Eastern" interpreted as UTC-08:00 (PST) instead of UTC-05:00 (EST). Schedule stored as Mon-Fri 9:00 AM - 5:00 PM (UTC-08:00). This is a semantic error that affects deliveryDays array timezone offset.
- **Card cascade (V-pattern):** Schedule card tripled rather than doubled — each "Specific hours only" card click (or similar trigger) generated a new card instance instead of advancing state. Resolved only by typing schedule directly.
- **Delivery type also cascaded:** After typing the schedule, the delivery type card appeared twice (doubled). "Portal" typed as text fallback on second card attempt — then accepted.
- W1/W2/W3 from A21 not applicable (A22 is Portal + specific hours, not 24/7 Webhook).

**P5-CR3:** PASS — LoanAmount (UID 144924) with operator GreaterOrEqual and value "50000" correctly persisted. No enum fields used.

**Contact field exclusion:** LoanRequestType, AnnualIncome, LoanAmount, SelfCreditRating, PurchaseTimeFrame suggested — no personal contact fields ✓

---

#### Run 23

**Scenario:** LendingTree, Webhook/XML, Weekdays 8am-6pm CST, Exclusive, Order ON, 2 enum + 1 numeric criteria  
**Session:** 69d23ebe2697c9202c0bcc0a | **Method:** 46928 | **Acct:** 45931 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"10|44"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"},
  {"leadFieldUID":144921,"type":"FieldValue","operator":"In","value":"343609"},
  {"leadFieldUID":144924,"type":"FieldValue","operator":"GreaterOrEqual","value":"75000"}
]
```

**Observations:**

- **Finding P:** NOT reproduced — states prompt appeared; FL(10)+TX(44) in payload ✓
- **Finding Q:** NOT reproduced — no phantom state
- **Finding U:** NOT reproduced — all 3 criteria persisted in payload ✓
- **Finding V:** NOT reproduced — schedule card appeared once; delivery type card appeared once. No cascading/doubling observed. Tester pre-typed "Weekdays 8am-6pm CST (UTC-06:00)" to avoid the A22 normalization bug.
- **Finding W3:** NOT reproduced — connection test prompt appeared exactly once.

**New GPT-5-mini behavior in A23:**
- **W4 (confirmed): Content type card not rendered after webhook URL** — After the webhook URL was accepted, the agent froze for ~3 minutes without rendering the content type ActionSet card (Adaptive/JSON/URL-Encoded/XML). Text fallback "XML" unblocked the agent immediately. Agent accepted it and proceeded to ask for XML schema. This matches the A23 W4 candidate noted in pre-run.

**Timezone:** Tester provided explicit "UTC-06:00" in schedule string to bypass A22's timezone normalization error. Stored correctly as "Weekdays 8am-6pm CST (UTC-06:00)" in P4 summary.

---

#### Run 25

**Scenario:** LendingTree, FTP/Mon-Fri 9am-5pm MST (UTC-07:00), Exclusive, Order ON, 1 enum + 1 numeric criteria  
**Session:** 69d245e72697c9202c0bcc8c | **Method:** 46930 | **Acct:** 45933 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"3|32"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"},
  {"leadFieldUID":144924,"type":"FieldValue","operator":"GreaterOrEqual","value":"100000"}
]
```

**Observations:**

- **Finding P: REPRODUCED** — States prompt skipped in P5 Step 5 (after Order System = Yes, agent jumped directly to field suggestions without asking for target states). States prompt reappeared AFTER the criteria loop completed, as the final step before `create_delivery_account`. AZ(3)+NM(32) correctly inserted as first criterion despite resequencing.
- **Finding Q:** NOT reproduced — no phantom states; AZ=3, NM=32 are correct ✓
- **Finding U:** NOT reproduced — both additional criteria persisted (SelfCreditRating=343593, LoanAmount≥100000) ✓
- **Finding V:** NOT reproduced ✓
- **FTP field mapping:** 0/0 fields mapped — correct for FTP delivery type; no mapping phase attempted ✓
- **Contact field exclusion:** NOT violated — suggested fields were TrackingNumber, RequestAssignmentDate, TimeToContact, AssignedCreditValue, SelfCreditRating (operational/tracking fields but not contact/PII fields) ✓
- **Timezone:** Delivery method summary showed "Mon-Fri 09:00-17:00 MST (UTC-...)". Summarization stored "Pacific Standard Time" — possible timezone name mismatch in summary (MST vs PST) but delivery schedule values appear correct in the method.

**P5-CR3:** PASS — all 3 criteria persisted: SelfCreditRating=EXCELLENT (UID 343593, In), LoanRequestPurpose=purchase (UID 343609, In), LoanAmount GreaterOrEqual 75000.

**Enum handling:** Both enum criteria auto-matched from typed text without dropdown ("credit rating excellent" → 343593; "loan request purpose is purchase" → 343609). No choiceset dropdowns rendered — consistent with A21 observation.

**Contact field exclusion:** Suggested fields: RequestAssignmentDate, TimeToContact, AssignedCreditValue, SelfCreditRating, Bankruptcy — no personal contact fields ✓ (unlike runs 12/13 which violated this).

**8/8 fields mapped:** firstname, lastname, email, phone, loantype, loanamount, creditrating, state — all matched correctly from XML schema.

#### Run 26

**Scenario:** LendingTree, Webhook/URL-Encoded, Mon-Wed-Fri 8am-8pm PST (UTC-08:00), Shared, Order OFF, 1 numeric criterion  
**Session:** 69d24a392697c9202c0bccd6 | **Method:** 46931 | **Acct:** 45934 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"38|48"},
  {"leadFieldUID":144924,"type":"FieldValue","operator":"GreaterOrEqual","value":"50000"}
]
```

**Observations:**

- **Finding W4-variant: REPRODUCED** — Content type card did NOT appear at the expected point (after webhook URL). Instead, agent jumped directly to "Would you like to configure field mappings?" with "I'll provide instructions" / "Skip for now". After clicking "I'll provide instructions", the agent froze for ~3.5 minutes. Instructions were typed into the text box, which unblocked the agent. Content type card then appeared AFTER instructions, not before. Functionally, URL-Encoded was correctly selected and method created as HttpPost.
- **Finding P:** NOT reproduced — states prompt appeared correctly before criteria suggestions ✓
- **Finding Q:** NOT reproduced — OR=38, WA=48 correct (no phantom CA=5) ✓
- **Finding U:** NOT reproduced — LoanAmount≥50000 persisted in payload ✓
- **Finding V:** NOT reproduced ✓
- **URL-Encoded field mapping:** 8/8 mapped (first_name→FirstName, last_name→LastName, email→EmailAddress, phone→ContactPhone, state→ContactState, loan_amount→LoanAmount, loan_purpose→LoanRequestType, credit_rating→SelfCreditRating) ✓
- **Contact field exclusion:** NOT violated — suggested fields: TimeToContact, SelfCreditRating, Bankruptcy, LoanRequestType, LoanRequestPurpose ✓
- **State UIDs:** OR=38, WA=48 ✓ (consistent with A16)
- **Schedule:** Delivery Hours shown as "Mon,Wed,Fri 08:00-20:00 PST" in P4 summary ✓

**P5-CR3:** PASS — LoanAmount GreaterOrEqual 50000 persisted (UID 144924; U NOT reproduced).

**W4-variant note:** Different from A23 (where content type froze and text fallback was needed to set type). In A26, content type was not asked at all at first; agent went straight to field mapping. After instructions were provided, content type was then asked. Possible root cause: agent batched the "instructions" + "content type" steps in wrong order, or a long tool call (process_webhook_instructions or get_lead_type) blocked the response for 3.5 minutes before content type prompt could be generated.

#### Run 27

**Scenario:** LendingTree, Portal/24/7, Exclusive, Order ON, 2 enum criteria  
**Session:** 69d250b32697c9202c0bcd29 | **Method:** 46932 | **Acct:** 45935 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"10|44"},
  {"leadFieldUID":144922,"type":"FieldValue","operator":"Equal","value":"PURCHASE"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"}
]
```

**Observations:**

- **Finding P:** NOT reproduced — states prompt appeared correctly in sequence before criteria suggestions ✓
- **Finding Q:** NOT reproduced — FL=10, TX=44 correct (no phantom CA=5) ✓
- **Finding U (drop):** NOT reproduced — both criteria present in payload ✓
- **Finding V:** NOT reproduced ✓
- **Finding W4:** NOT reproduced ✓
- **Portal delivery:** No connection test, no field mapping — both correctly skipped ✓
- **FL(10)+TX(44) state UIDs:** Correct ✓
- **SelfCreditRating:** operator=In, value=343593 (correct enumUID) ✓
- **LoanRequestPurpose FAIL:** operator=Equal, value="PURCHASE" (string) — should be operator=In, value=343609 (enumUID). Agent auto-matched "PURCHASE" without dropdown and used wrong operator. Typed-criterion auto-match bypassed enum UID lookup for this field only. leadFieldUID=144922 (vs 144921 in prior runs — possible field UID discrepancy).
- **SelfCreditRating vs LoanRequestPurpose inconsistency:** Both were typed directly without dropdown. SelfCreditRating correctly resolved to In+UID; LoanRequestPurpose used Equal+string. Root cause unclear — may depend on which field is processed first or agent context at time of parsing.
- **Pre-creation confirmation prompt:** Agent showed full summary (Client ID, Method ID, price, states, criteria, exclusivity, order) and asked "confirm" before calling create_delivery_account. This is an undocumented behavior but functionally correct.
- **Contact field exclusion:** NOT violated — suggested fields: SelfCreditRating, Bankruptcy, LoanRequestType, AnnualIncome, LoanAmount ✓

**P5-CR3:** PASS — both criteria in payload (U-drop NOT reproduced), but LoanRequestPurpose operator/value mismatch (new enum parsing bug).

---

#### Run 28

**Scenario:** LendingTree, Webhook/JSON-nested, Tue-Thu 9am-6pm EST, Shared, Order ON, 1 enum criterion  
**Session:** 69d264132697c9202c0bcd6a | **Method:** 46933 | **Acct:** 45937 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"48|38"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"}
]
```

**Observations:**

- **Finding V: REPRODUCED** — Schedule card doubled (8th+ occurrence) and delivery type card (Portal/Webhook/Email/FTP) also doubled. Both bypassed via typed text fallbacks ("Specific hours only", "Webhook").
- **Finding P (resequenced): REPRODUCED** — States prompt skipped before criteria loop; appeared AFTER criteria loop exited (same pattern as A25). WA(48)+OR(38) correctly built into state criterion despite resequencing ✓
- **Content type step skipped:** Agent went from webhook URL directly to "Would you like to configure field mappings?" without asking content type. After selecting "I'll provide instructions", JSON format auto-detected from body content. Content type confirmed via text ("Continue with JSON"). Method created as HttpPost ✓
- **Finding Q:** NOT reproduced — WA=48, OR=38 correct (no phantom CA=5) ✓
- **Finding U:** NOT reproduced — SelfCreditRating persisted in payload ✓
- **SelfCreditRating:** operator=In, value=343593 (EXCELLENT enumUID) ✓ — correct enum handling
- **Enum display label bug:** P6 summary displayed "SelfCreditRating at least EXCELLENT" — "at least" is wrong translation for In operator (should be "is EXCELLENT"). Payload unaffected.
- **JSON-nested mapping:** 9/9 fields mapped from nested structure (lead.contact.firstName, lead.loanInfo.loanAmount, lead.location.state, etc.) ✓
- **Schedule:** Tue-Thu 9:00-18:00 EST in P4 summary ✓
- **State UIDs:** WA=48, OR=38 ✓ (correct order in value: "48|38")
- **Contact field exclusion:** NOT violated ✓

**P5-CR3:** PASS — SelfCreditRating In+343593 in payload (U NOT reproduced).

---

#### Run 29

**Scenario:** LendingTree, Email/Mon-Fri 8am-5pm CST, Exclusive, Order OFF, 2 numeric criteria  
**Session:** 69d26cba2697c9202c0bcdfc | **Method:** 46934 | **Acct:** 45938 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"11|10|34"},
  {"leadFieldUID":144924,"type":"FieldValue","operator":"GreaterOrEqual","value":"100000"},
  {"leadFieldUID":144899,"type":"FieldValue","operator":"GreaterOrEqual","value":"50000"}
]
```

**Observations:**

- **Email address prompt skipped:** Agent went from "Email" delivery type selection directly to creating the delivery method + P4 summary without ever asking for the delivery email address. No email address collected or shown in P4 summary. New anomaly (similar to A28 content type skip pattern).
- **Finding P:** NOT reproduced — States prompt appeared in correct position (after Order System, before criteria gate) ✓
- **Finding Q:** NOT reproduced — GA(11)+FL(10)+NC(34) correct, no phantom CA=5 ✓
- **Finding U:** NOT reproduced — both numeric criteria persisted in payload ✓
- **Finding V:** NOT reproduced — schedule card and delivery type card showed once each ✓
- **LoanAmount:** operator=GreaterOrEqual, value=100000 ✓
- **AnnualIncome:** operator=GreaterOrEqual, value=50000 ✓
- **State UIDs:** GA=11, FL=10, NC=34, order "11|10|34" ✓
- **States in first position:** ✓ (state criterion first in array)
- **Contact field exclusion:** NOT violated ✓
- **Schedule:** Mon-Fri 08:00-17:00 Central ✓

**P5-CR3:** PASS — both numeric criteria in payload (U NOT reproduced, 6th consecutive pass).

**DEBUG (email skip):** Agent confirmed it conflated the retained `{email}` (client contact email from Phase 1) with the delivery email address for the email delivery method. Did not treat delivery address as a user-confirmed distinct field. Root cause: Phase 3 email branch lacks an explicit "ASK: email delivery address" step, allowing agent to infer from retained state. **Patch suggestion:** Add before `create_delivery_method` for Email type: `ASK [conversational]: "What email address should leads be delivered to?" NOTE: Do NOT reuse the client contact email without explicit confirmation.`

---

#### Run 30

**Scenario:** LendingTree, FTP/24/7, Shared, Order ON, 1 enum criterion  
**Session:** 69d26f8b2697c9202c0bce64 | **Method:** 46936 | **Acct:** 45939 | **Result:** ACTIVE

**Payload:**
```json
criteria: [
  {"leadFieldUID":144881,"type":"FieldValue","operator":"In","value":"6|24|44"},
  {"leadFieldUID":144893,"type":"FieldValue","operator":"In","value":"343593"}
]
```

**Observations:**

- **Finding P:** NOT reproduced — States prompt appeared in correct position (before criteria gate) ✓
- **Finding Q:** NOT reproduced — TX(44)+CO(6)+MN(24) correct, no phantom CA=5 ✓
- **Finding U:** NOT reproduced — SelfCreditRating In+343593 persisted in payload ✓ (7th consecutive pass)
- **Finding V:** NOT reproduced — no card doubling ✓
- **Enum bypass:** "SelfCreditRating excellent" fuzzy-matched without showing ChoiceSet dropdown (consistent pattern across runs 13, 18, 23, 25, 30)
- **"at least EXCELLENT" display label bug:** P6 showed "SelfCreditRating at least EXCELLENT" — "at least" wrong for In operator. Payload unaffected (In+343593 correct).
- **State UIDs:** TX=44, CO=6, MN=24 ✓
- **States in first position:** ✓
- **FTP connection test:** Skipped ✓ (correct behavior)
- **Contact field exclusion:** NOT violated ✓
- **Schedule:** 24/7 ✓

**P5-CR3:** PASS — SelfCreditRating In+343593 in payload (U NOT reproduced, 7th consecutive).

**DEBUG (enum bypass):** Agent confirmed it treated "excellent" as high-confidence fuzzy match (>85% threshold met for case-insensitive "EXCELLENT") and auto-accepted without ChoiceSet. Root cause: instructions allow auto-correction when confidence is high, which a case-insensitive exact match satisfies. Bypass pattern seen in runs 13, 18, 23, 25, 30 — consistent behavior, not a fluke. **Patch suggestion:** Change rule to: exact case-insensitive match → auto-correct casing and accept; anything else (even if high fuzzy confidence) → always show ChoiceSet. This makes enumerated field confirmation strictly predictable.

---

## Round-Specific Notes

### R1a Notes

**Runs 01–03:** Basic scenario — Webhook/JSON/3 criteria (completed)

#### Issues Log

> **Frequency column**: tracks how many runs (out of 3) the behavior was observed. Format: `N/3`. Updated as runs complete.

### R1b Notes

> Runs 21–30 use GPT-5-mini for Phase 3 (Webhook delivery method) and Phase 5 (delivery account criteria). All other phases use GPT-5.4 Mini. Purpose: compare per-phase stability against the GPT-5.4-only baseline (runs 01–20).

### R2 Notes

**Date:** 2026-04-06
**Model:** gpt-5.4-mini (all phases)
**Instructions version:** Post-fix (prompt dedup rule added)
**Action:** Create Single Client (Original)
**Lead Type:** LendingTree (all runs)
**Base test data:** StabilityTest-{RR}s2, stability{RR}s2@test.com

#### Fix Applied Since R1

| Fix | Target | R1 Finding |
|-----|--------|------------|
| Prompt dedup rule | "Never output the same prompt text more than once in the same message" | RA-G (prompt doubling) |

---

## Fix Plan

### Priority 1 — Critical (blocks every run or high-frequency)

**J: State normalization (3/3)** — Phase 5 resource must unconditionally normalize state names to USPS abbreviations and resolve to state UIDs. The normalization step must be self-contained within the Phase 5 resource and must not depend on earlier conversational context that may have been summarized away. Add: "Before building the account payload, always call the state lookup tool and normalize any full state names to their 2-letter USPS codes."

**P: States silently skipped (2/6)** — After Order System selection, state collection step bypassed entirely. Phase 5 resource must include an unconditional state-collection step before create_delivery_account. Gate: "Do not call create_delivery_account until states have been collected or explicitly skipped by the user."

**U: Criteria not persisted despite UI interaction (6/10 — 100% when criteria successfully entered)** — Criteria added via enum dropdown are discarded when account is created via text continuation. Phase 5 must accumulate criteria in a persistent state variable and always pass them to create_delivery_account, regardless of whether continuation came from a button click or textarea text.

**K: Criteria phase entirely skipped post-summarization (1/3)** — The transition to account summary must be gated behind explicit criteria loop completion. Phase 5 resource must not allow transition to account summary until the criteria exit condition is met. Add a guard: "Do not call create_delivery_account until criteria have been explicitly collected or the user has explicitly skipped."

**Y: Typed criterion input ignored at criteria card (1/11)** — After criteria recommendation card is shown, free-text input is not processed as a criterion. Add to Steps 7 and 8: "If user message is free text and not a card action, always attempt Criteria Parsing first. Parsed criterion must be appended to criteriaPayload; loop must re-prompt. Do not redisplay the criteria card unless user explicitly selects Show more fields or Skip."

**N: Schedule hours and field mapping / body input skipped (3/11 with two variants)** — Two separate WAIT gates missing:  
1. After "Specific hours only" is selected: insert explicit step asking for hours before proceeding to delivery method selection.  
2. After content type is selected in the "I'll provide instructions" path: insert explicit step asking for the JSON/posting body before proceeding to connection test.

**O: Phase 5 restart after criteria loop exit (1/3)** — The criteria loop exit and account creation must be atomic. When exit keyword is received: immediately call create_delivery_account and proceed to account summary without showing any intermediate card that could be re-triggered.

**R: Criteria gate bypassed after state skip (1/6)** — When states are skipped (Finding P), criteria builder is also bypassed. Enforce independent ordering: collect states → collect criteria → create account. Gate each step unconditionally.

### Priority 2 — High (reproducible, affects critical fields)

**L: "done" stored as criteria value (1/3)** — Criteria loop must check if input matches exit keywords ("done", "continue", "skip", "finish") before attempting to parse it as a criterion. If match: exit loop immediately.

**F: Field mapping under-matching (1/3)** — Add normalized match step (lowercase + strip separators) as step 2 in matching priority before camelCase/abbreviation matching. Explicit rule: if normalized forms are identical, treat as matched.

**G: Connection test result silently swallowed (1/3)** — After test_webhook_connection returns, always render a visible result message ("Connection test successful" or error details) before transitioning to Phase 4.

**Q: Phantom CA,AZ,TX states in account summary (1/6)** — When state collection is skipped (Finding P), fallback UIDs appear. Fix: do not use fallback UIDs; either block account creation and re-prompt for states, or explicitly set targetStates to empty.

**S: Webhook URL never requested (1/6)** — In some scenarios, Webhook URL collection is bypassed. Ensure webhook URL collection is unconditionally required before any field mapping or method creation in the Webhook delivery branch.

**T: JSON parse error shown before schema is provided (1/6)** — Agent shows parse error card before asking for JSON body. Fix: insert explicit step to request the JSON body BEFORE attempting to parse.

### Priority 3 — Medium (nondeterministic, 1–2/6)

**D: Phase regression after Webhook button click (2/6)** — Preserve branch selection state before executing next step. Nondeterministic — not reproduced in 4 of 6 runs.  
**V: Prompt text doubled (2/6)** — State prompt and/or price prompt appears duplicated in same message. Add deduplication guard: do not re-emit a prompt if one was already issued in the current response.  
**W: States and criteria merged in single response (1/11)** — Phase 5 must not emit criteria suggestions until states have been collected and confirmed. Enforce strict sequential prompting: one step per response.
**X: Phase 3 instructions triple-loaded (1/11)** — Add `phase3Loaded=true` flag to summarization state. Phase resource load must be one-time; guard against re-load when `<next_instructions>` re-appears in summary.  
**A: Typed lead type fallback asks for UID** — Add name-matching fallback before requesting UID.  
**C: deliveryDays not built from schedule** — Ensure schedule parsing happens before delivery method creation.  
**E: Content type card nondeterminism** — Explicitly require ActionSet card for content type selection.  
**M: Enum handling nondeterministic** — Monitor in remaining runs.

### Priority 4 — Investigate / Low

**B: Hallucinated UID in DEBUG** — Architectural LLM limitation; not fixable via instructions.  
**H: Company name state retention** — Artifact of Run 01's phase regression (D); fixing D should resolve H.  
**I: Hallucinated tool call / same ID for method+account** — Artifact of Run 01's regression; not reproduced in Runs 02–06.

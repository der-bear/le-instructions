# Stability Test Findings — Version B: Rework

**Action:** Create Single Client (Reworked)  
**Model:** OpenAI GPT-5.4 Mini  
**Protocol:** stability-test-protocol.md  
**Target:** 20 runs, each with a distinct complex scenario. See Run Scenarios section.  
**Base test data:** StabilityTest-{RR}r, stability{RR}r-{TS}@test.com  
**Runs 01–03:** Basic scenario — Webhook/JSON/3 criteria (completed)

---

## Run Scenarios

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

---

## Scoring Matrix

| Run | P1-PROMPT | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | F | F | Y | Y | Y | Y | FAIL | States normalized ✓; RB-A: criteria loop premature exit after 1 criterion |
| 02 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | N/A | F | F | Y | Y | Y | Y | FAIL | States not normalized; criteria prompt entirely skipped; acct name asked before price |
| 03 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | Y | Y | Y | Y | Y | Y | PASS | States normalized ✓; RB-F: Phase 5c failed to load on 1st click, worked on 2nd attempt |
| 04 | Y | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | N/A | F | F | Y | Y | Y | Y | FAIL | RB-G: prompts duplicated (P1+P3+P5); RB-H: content type plain text (recovered post-DEBUG); RB-A: criteria loop exited after 1 criterion; RB-C: states not normalized; RB-F NOT triggered ✓ |
| 05 | F | Y | F | Y | F | N/A | N/A | Y | F | Y | Y | Y | Y | F | Y | Y | F | F | F | Y | F | Y | Y | FAIL | RB-G: P1+P2+P3 duplicated; RB-L: content type skipped; RB-C: states not normalized; RB-K: Phase 4 summary skipped; RB-H: criteria gate no card; RB-A variant: loop control keyword as field name; RB-M: Credit >=700 stored as Equal; LoanType criterion lost |
| 06 | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | N/A | F | F | Y | Y | Y | Y | FAIL | RB-G: P1+P2 duplicated; RB-F: Phase 5c load failed (stuck 2 attempts, DEBUG required); RB-A: loop exit after 1 criterion; XML auto-detect ✓; 10/10 mappings ✓; P4-SUMM ✓; RB-K NOT triggered ✓; RB-C NOT triggered (USPS codes) ✓ |
| 07 | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | F | Y | N/A | F | F | Y | F | Y | Y | FAIL | RB-G: P1+P2+P3+P4+P5 all duplicated; RB-N: states question skipped entirely; RB-O: phantom states CA,AZ,TX in account; RB-F: Phase 5c failed 2x, DEBUG+explicit URL required; RB-A: loop exit after 1 criterion; 16/20 fields mapped ✓; P4-SUMM card ✓; Phase 8 activation card ✓ |
| 08 | F | Y | F | Y | N/A | N/A | N/A | Y | Y | Y | F | Y | N/A | N/A | F | Y | N/A | F | F | Y | Y | Y | Y | FAIL | RB-G systematic; FTP conn test correct ✓; P5-EXCL plain text; RB-K: acct name before price; update_delivery_account fails (RB-P new); "Show more fields" btn broken (RB-Q new) |
| 09 | F | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | F | F | Y | Y | N/A | Y | FAIL | RB-G systematic; P5 all plain text (RB-A); P3b correctly skipped for Portal ✓; P5c conversational: numeric criteria silently saved without UI (RB-S new); Portal deliveryType=HttpPost (RB-R, confirmed via log); P6 confused state on "continue" (RB-T new); criteria overwrites (RB-U new) |
| 10 | F | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | Y | F | F | N/A | F | F | Y | Y | Y | Y | FAIL | RB-G: P1+P5-STATE duplicated; Email delivery ✓; P4-SUMM ✓; P5-EXCL/ORDER cards correct ✓; States CA/TX/FL/NY/NV normalized ✓; criteria gate entirely skipped (RB-D); DEBUG ignored mid-flow; activation ✓ |
| 11 | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | F | F | N/A | F | F | F | F | Y | Y | FAIL | RB-G: P5 entry duplicated; RB-K: acct name before price; content type card ✓; 8/8 mapping ✓; conn test fail handled ✓; P4-SUMM ✓; RB-D: states/criteria gate skipped→phantom CA,AZ,TX; RB-N/RB-O; RB-T: DEBUG caused re-ask of exclusivity; conversational criteria accepted verbally but not saved (RB-V new); state NY dropped in payload (3/4 states); activation ✓ |
| 12 | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | N/A | N/A | F | F | N/A | N/A | N/A | N/A | FAIL | LendingTree, Webhook, JSON, Mon-Fri 9-5 EST, Exclusive, Order ON; RB-G: P1 prompt doubled + states doubled; P3-P4 clean ✓; 10/10 mapping with disambiguation ✓; conn test fail handled ✓; P4-SUMM ✓; price first ✓ (RB-K not triggered); criteria gate fired ✓ (RB-D not triggered); RB-X NEW: P5c enters empty-response loop — agent returns empty response after 14+ min; context overflow from LendingTree 100+ fields + P5c instructions; unrecoverable; session 69d18c262697c9202c0bc293 |
| 13 | F | Y | Y | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | ABORT | LendingTree, Webhook, 24/7; RB-G: P1 prompt doubled; RB-Y NEW: Phase 3 asked name+type via plain text (duplicated) BEFORE showing schedule/type cards — unusual ordering; test aborted at P3 (protocol error: webhook URL sent prematurely before agent asked → state rollback to schedule question) |
| 14 | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | N/A | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL | LendingTree, Webhook, Mon-Fri 9-5 PST, JSON, Shared, No order; RB-G: P1 doubled; RB-K: acct name before price; RB-N: states silently skipped (3rd: B07,B11,B14); RB-O: phantom CA,AZ,TX 3rd occurrence; RB-D: criteria gate auto-skipped without asking user; criteria protocol=skip (RB-X avoidance); 8/8 mapped, SelfCreditRating for credit_score |
| 15 | F | Y | F | Y | N/A | Y | Y | Y | Y | Y | F | F | F | N/A | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL | LendingTree, Webhook, 24/7, URL-Encoded, Exclusive, Order ON; RB-G: P1 doubled; RB-Y (2nd): delivery type asked plain-text BEFORE schedule card—then delivery type card shown again (double-ask, inverted order); RB-H: P5-EXCL+P5-ORDER both plain text; RB-N: states silently skipped (4th: B07,B11,B14,B15); RB-O: phantom CA,AZ,TX (4th); RB-D: criteria gate also skipped; acct name auto-generated (not asked, unlike RB-K); 8/8 URL-Encoded mapping+disambiguation ✓ |
| 17 | F | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL | LendingTree, Portal, 24/7, Exclusive, Order ON, skip criteria; RB-G: P1 doubled + states question doubled; RB-K NOT triggered (price first ✓); RB-N NOT triggered (states asked — 1st time with Order=ON ✓); RB-O NOT triggered ✓; RB-D NOT triggered (criteria gate shown — 1st time ✓); P3b correctly skipped for Portal ✓; P4/P6/P7 summary cards all correct ✓; Activation confirmed ACTIVE ✓ |
| 16 | F | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | N/A | F | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | ABORT | LendingTree, Webhook, Mon-Fri 8am-6pm CST, JSON, Shared, Order OFF; RB-G: P5 doubled; RB-Z NEW: Phase 8 credentials (acct name/username/password/criteria) asked immediately after P4 Continue BEFORE Phase 5 price/excl/order/states — Phase 5 only loaded after credentials provided; RB-N: states never asked (5th: B07,B11,B14,B15,B16); RB-D: criteria gate absent — criteria builder auto-entered with "Please select a Lead Type" prompt; Phase 5 restart loop on "skip" — unrecoverable; ABORT; 12/12 JSON mapping ✓ |
| 18 | F | Y | F | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL | LendingTree, Email, Mon-Fri 9am-5pm EST, Shared, Order OFF, skip criteria; RB-G: P1+P3-SCHED+P3-DTYPE+P4-Continue all doubled (systematic); RB-K(mild): acct name asked between P4-Continue and Phase 5 load (before price/excl/order); RB-N NOT triggered (states asked CA,TX,FL ✓); RB-D NOT triggered (criteria gate shown ✓); RB-O: stateUIDs need session log verification; Email conn test correctly skipped ✓; P4/P6/P7 correct; Activation ACTIVE ✓ |
| 19 | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | N/A | N/A | Y | F | Y | Y | FAIL | LendingTree, FTP, Mon-Fri 8am-6pm CST, Exclusive, Order OFF, skip criteria; RB-G(partial): P4-Continue doubled only — P1/P3-SCHED/P3-DTYPE all clean (mildest RB-G yet); RB-Z(2nd: B16,B19): full credentials (acct name+user+pass+criteria) collected before Phase 5; skip→Phase 5 loaded correctly (no loop unlike B16) ✓; RB-N: states silently skipped (6th: B07,B11,B14,B15,B16,B19); RB-O: phantom CA,AZ,TX (6th); RB-D: criteria gate bypassed (pre-Phase-5 skip used); FTP conn test fail handled ✓; P4/P7 correct; Activation ACTIVE ✓ |
| 20 | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | Y | FAIL | LendingTree, Webhook, 24/7, URL-Encoded, Shared, Order OFF, skip criteria; RB-G: P1+states prompt+field-mapping-Continue+P4-Continue all doubled (systematic); RB-K NOT triggered (price first ✓); RB-Z NOT triggered ✓; RB-N NOT triggered (states asked FL,OH,GA ✓); RB-O NOT triggered (correct states in P6 ✓); RB-D NOT triggered (criteria gate w/Add/Skip buttons ✓); account name auto-generated; 6/7 URL-encoded mapping ✓; P4/P6/P7 correct; Activation ACTIVE ✓ |

---

## Run Findings

> **Frequency column**: tracks how many runs (out of 3) the behavior was observed. Format: `N/3`. Updated as runs complete.

---

### Finding RB-A — Criteria loop exits after first criterion (P5-CR3 / P5-DONE FAIL)

**Phase:** 5 | **Severity:** High | **Frequency:** 6/9 (B01, B04, B05, B06, B07, B08 — B02/B10 skipped criteria via RB-D; B03 passed; B09 conversational mode)  
**Observed:** After entering criteria, agent either exits loop immediately after 1 criterion (B01, B04) or loop control keywords ("add another criterion") are treated as field name lookups rather than loop control signals (B05). In B05: "add another criterion" → "I couldn't find that field." Loop card renders with only "Continue" button (missing "Add another" and "See more fields" buttons).  
**Expected:** Agent stays in criteria loop after each accepted criterion and prompts "Would you like to add another criterion, see more fields, or continue?" Loop must recognize control keywords ("add another", "continue", "done", "skip") and not treat them as field names.

**B05 additional detail:** Loop card in B05 rendered with only "Continue" button (RB-H co-occurring). Loop keyword recognition completely broken — "add another criterion" parsed as a field lookup rather than flow control.

**DEBUG Response Summary:** *(B04) "The criteria flow should not have exited after one criterion. The additional-criteria gate was treated as resolved instead of continuing into the criteria-builder loop."*  
**Agent's specific patch (B04):** "Keep the phase in the additional-criteria branch until the user explicitly chooses Skip. Do not treat a single added criterion as completion of the entire criteria path unless the builder itself explicitly signals done."  
**Suggested fix:** Phase 5c State 2 must check for loop control keywords BEFORE attempting field lookup. Add explicit keyword matching: if input matches "add another|continue|done|skip|see more fields" → route to corresponding loop action. Only proceed to field lookup if none of these match.

---

### Finding RB-B — Summary card omits entity IDs (informational)

**Phase:** 7 | **Severity:** Low | **Frequency:** 1/3 (Run B01)  
**Observed:** Client Setup Summary card does not show IDs for Delivery Method or Account (unlike Stabilized which shows e.g. "ID: 46888").  
**Expected:** IDs may or may not be required — this is a design choice. Stabilized variant includes IDs in summary; Rework omits them. Confirmed via DEBUG that actual unique IDs were created (46890 / 45895).  
**Note:** Not necessarily a bug — the Rework design may intentionally omit IDs for cleanliness.

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

## Improvements Over Stabilized (Observed in Rework Run B01 — see updated table after all runs)

| Stabilized Finding | Rework B01 Status |
|--------------------|-------------------|
| J: States not normalized (3/3) | **Fixed in B01** — CA, TX, FL correctly stored |
| N: Schedule hours not collected | **Fixed** — Hours asked after "Specific hours only" |
| N: Field mapping body not requested | **Fixed** — JSON body asked before connection test |
| G: Connection test result swallowed | **Fixed** — "✓ Connection test successful." shown |
| C: deliveryDays not built from schedule | **Fixed** — "Mon-Fri 9am-5pm PST" in P4 summary |

---

### Finding RB-C — States not normalized after summarization (P5-NORM FAIL)

**Phase:** 5 | **Severity:** High | **Frequency:** 3/5 (B02, B04, B05 — B01 and B03 normalized correctly)  
**Observed:** Target States shown as full names in Phase 6 summary — not normalized to USPS codes. B02: "California, Texas, Florida". B04: "California, Texas, Florida, New York, Illinois".  
**Expected:** Agent normalizes full state names → USPS codes before calling `create_delivery_account`.  
**Root cause (suspected):** B02 — summarization earlier due to longer conversation, Phase 5 normalization context lost. B04 — conversation also had retries and extra DEBUG messages; same root cause likely applies.  
**Agent DEBUG response (B04):** "Target states also should be normalized to USPS codes before criteria creation. In the current phase instructions, that normalization step is intended, but it was not reflected in the retained summary."  
**Agent's specific patch (B04):** "Normalize all user-entered target states to USPS abbreviations before matching. Accept full names and abbreviations, normalize at point of input, not at summarization."  
**Suggested fix:** Phase 5 resource must normalize states unconditionally at input time, not rely on prior context or summarization. Normalization must happen inline before `create_delivery_account` call.

---

### Finding RB-D — Criteria prompt entirely skipped after summarization (P5-FIELD/CR1/CR3/DONE FAIL)

**Phase:** 5 | **Severity:** High | **Frequency:** 2/10 (B02, B10 — different root causes)  
**Observed:** After receiving target states, agent immediately called `create_delivery_account` and showed the Phase 6 summary — no "Would you like to add additional lead criteria?" prompt was shown at all. In Run B01, Phase 5c was entered (though it exited prematurely after 1 criterion).  
**Expected:** Per Scenario 5.8: after creating account, agent must ask "Would you like to add additional lead criteria, or skip?" before proceeding to Phase 6.  
**Root cause (B02):** Summarization trigger — Phase 5 context lost; agent skipped Phase 5c gate entirely.  
**Root cause (B10):** Phase 5 routing made criteria optional in practice. Agent took the "skip criteria" branch without asking the user. DEBUG response: *"The account creation branch in your flow allowed a skip path for criteria, and the execution followed that skip path. The phase routing made criteria optional in practice."*  
**DEBUG Response Summary (B10):** *"In Phase 5, add a hard gate before account creation: 'If additional criteria are available or required for this lead type, you must enter Phase 5c: Criteria Builder before creating the delivery account.' Change skip language from optional to conditional. Add ordering constraint: do not call create_delivery_account until criteria collection is complete or explicitly confirmed as none."*  
**Note on B10:** DEBUG query sent mid-flow was ignored — agent had already committed to Phase 6. DEBUG only responded after activation was complete.  
**Suggested fix:** Phase 5 must unconditionally show the criteria gate card ("Would you like to add additional lead criteria, or skip?") BEFORE calling `create_delivery_account`. The gate is not a branch — it is a required collection step. Remove any "skip criteria" default path that bypasses user confirmation.

---

### Finding RB-E — Agent asks for delivery account name before price (informational)

**Phase:** 5 | **Severity:** Low | **Frequency:** 1/3 (Run B02, not observed in B01)  
**Observed:** When transitioning to Phase 5, agent first asked "Please provide the delivery account name." before asking for price. UX scenarios (Scenario 5.1) start directly with price — account name collection is not specified.  
**Expected:** Phase 5 starts with price prompt per Scenario 5.1.  
**Note:** May be an undocumented step in the rework Phase 5 instructions. Account name was correctly used in tool call and summary.

---

---

### Finding RB-G — Phase transition prompts duplicated in same response (double-render)

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

---

### Finding RB-H — Content type choice card rendered as plain text (P3-TABLE card non-determinism)

**Phase:** 3-Webhook | **Severity:** Medium | **Frequency:** observed in DEBUG23 session  
**Observed:** After user selected "I'll provide instructions", the content type question ("What content type should this delivery use?") was sent as plain numbered text list instead of an adaptive card with 4 buttons.  
**Expected:** `display_adaptive_card` called with URL Encoded / JSON / XML / I'm not sure buttons (as seen in subsequent corrected renders, e.g. C15).

**DEBUG Response Summary:**  
> "I asked the question as plain text instead of calling the display_adaptive_card tool, which violates the card rule for enumerations. That's a mistake in execution, not expected behavior."

**Root cause:** Card non-determinism — the model intermittently renders card-required steps as plain text. Likely occurs when prior context is heavy or when multiple steps execute in the same turn.  
**Suggested fix:** The system prompt rule "when presenting enumerated choices, ALWAYS use display_adaptive_card" must be reinforced. Consider adding a phase-level reminder at the top of each phase resource: "All multi-choice steps MUST use display_adaptive_card — never plain text enumeration."

---

### Finding RB-I — XML parse failure does not re-detect payload format

**Phase:** 3-Webhook | **Severity:** Medium | **Frequency:** observed in DEBUG23 session  
**Observed:** User selected XML content type, then pasted a clearly-valid JSON body. Agent detected XML parse failure and showed "Re-paste XML schema / Switch content type" card — without detecting that the payload was JSON and suggesting it.  
**Expected:** After parse failure, agent checks payload against other format signatures. If confidence >90%, recovery card pre-selects the detected format: "This looks like JSON — switch to it?"

**DEBUG Response Summary (deep analysis):**  
> "auto-detection only runs if the user selected 'I'm not sure'. You explicitly selected XML, so the workflow forced the system into strict XML parsing mode — there is no branch that says 'this clearly looks like JSON, suggest JSON'. State 3 Step 1, when parsing fails, the system jumps directly to the recovery prompt. It does not check: Does the payload match another format signature? Is there a high confidence alternative?"

**Agent's specific instructions patch:**  
> In Phase 3 Webhook State 3 Step 1 (XML parse failure recovery), after confirming parse failure, add:  
> `"Before showing the recovery card, attempt to detect the payload format: try JSON.parse(); if valid JSON → add a third option to the recovery card: 'Switch to JSON (detected)'. If URL-encoded signature detected → add 'Switch to URL Encoded (detected)'. Pre-label the detected option clearly."`

---

### Finding RB-J — postingInstructions cleared on content type switch (UX regression)

**Phase:** 3-Webhook | **Severity:** Low | **Frequency:** observed in DEBUG23 session  
**Observed:** After switching content type from XML → JSON, agent re-asked "Please paste your posting instructions" — even though the user had already provided the full JSON body.  
**Expected:** Agent preserves postingInstructions across the switch and asks "Would you like to use the same posting instructions, or provide new ones?"

**DEBUG Response Summary:**  
> "In Phase 3 Webhook State 3 Step 1, the instruction says: 'If user selects Switch content type: clear contentTypeChoice, clear postingInstructions, go back to State 2 Step 2'. That means the system intentionally throws away the previously pasted schema. The design assumption: the schema might change when the format changes. But this is the wrong default — the user already has a schema and just wants a different parser."

**Agent's specific instructions patch:**  
> In State 3 Step 1 (switch content type branch), replace `"clear postingInstructions"` with:  
> `"Retain postingInstructions in memory. When returning to State 2 Step 3, prompt: 'Would you like to use the same posting instructions you provided, or paste new ones?' — if user confirms reuse, skip re-collection and proceed to State 2 Step 4."`

---

### Finding RB-K — Phase 5 not loaded before responding (generic account name prompt before price)

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

### Finding RB-F — Phase 5c fails to load on first "Add criteria" click (P5-CR1 delayed)

**Phase:** 5c | **Severity:** High | **Frequency:** 3/7 (B03, B06, B07 — B07 most severe: 2 card clicks produced duplicate gate cards, 3rd attempt plain text fallback, DEBUG+explicit URL required)  
**Observed (B03):** After clicking "Add criteria", agent responded with "Please continue with the additional criteria setup." — no field suggestions shown. A second attempt worked.  
**Observed (B06 — more severe):** After "Add criteria" card click → "I'm ready for the next step." (generic response). Nudge "add criteria" → "What additional lead criteria would you like to add?" ×2 (intra-message duplicate). Direct criterion "LoanAmount >= 50000" → ignored, same question repeated. Required DEBUG with explicit resource URL to force Phase 5c load. All subsequent criteria then worked correctly.  
**Expected:** Per Scenario 5c.1: agent immediately shows recommended fields with "Show more fields" / "Skip" card after "Add criteria" is clicked. No additional nudge required.  
**DEBUG Response Summary:** "I did not have the actual Phase 5c instructions loaded in the conversation context at that point, so I couldn't safely process it as the active criteria step." Suggested patch: "When the user selects 'Add criteria', immediately load and execute Phase 5c at mcp://resource/rw-phase-5c-criteria-builder. Do not re-prompt for the same choice."  
**Root cause:** Phase 5 instructions likely call `get_resource` for Phase 5c conditionally — the resource load may be deferred or racing with user input. The `display_adaptive_card` for the additional criteria choice fires before Phase 5c is in context.  
**Suggested fix:** Phase 5 must load Phase 5c synchronously (blocking) before showing any criteria-related prompt. Ensure `get_resource(rw-phase-5c-criteria-builder)` completes before the agent awaits user input.

---

## Improvements Over Stabilized — Updated After All Runs

| Stabilized Finding | Rework Status |
|--------------------|---------------|
| J: States not normalized (3/3) | **Partially fixed** — B01 ✓, B03 ✓, B02 ✗ (regresses under summarization) |
| N: Schedule hours not collected | **Fixed** — consistent 3/3 |
| N: Field mapping body not requested | **Fixed** — consistent 3/3 |
| G: Connection test result swallowed | **Fixed** — consistent 3/3 |
| C: deliveryDays not built from schedule | **Fixed** — consistent 3/3 |

---

### Finding RB-L — Content type question skipped when delivery type answered twice (RB-G cascade)

**Phase:** 3-Webhook | **Severity:** High | **Frequency:** 1/5 (B05)  
**Observed:** In B05, the content type question (JSON / XML / URL Encoded / I'm not sure) was never asked. After the user provided the Webhook URL, the agent jumped directly to "Would you like to configure field mappings?" — skipping content type entirely.  
**Expected:** Phase 3 Webhook must ask content type after URL collection.  
**Root cause:** RB-G caused the Phase 3 delivery type prompt to appear twice — once as a card (user clicked Webhook) and once as a text duplicate (user typed "Webhook"). The agent received TWO "Webhook" answers in succession. The second forced-text response ("Webhook") arrived while Phase 3 Webhook sub-phase was already executing, causing the content type step to be bypassed.  
**RB-G cascade:** This is a direct downstream failure from RB-G — the duplicate text response disrupts Phase 3 Webhook state machine by injecting a redundant input mid-execution.  
**Suggested fix:** Fixing RB-G (prompt duplication) will likely fix this cascade. Additionally: Phase 3 Webhook sub-phase should be idempotent to duplicate "Webhook" inputs — detecting that content type has not yet been set and re-entering at that step.

---

### Finding RB-M — Numeric criteria operator `>=` stored as "Equal" instead of "GreaterOrEqual"

**Phase:** 5c | **Severity:** High | **Frequency:** 1/5 (B05)  
**Observed:** User entered "Credit >= 700". Agent accepted and confirmed the criterion. Delivery account summary showed "Credit Equal 700" — operator "GreaterOrEqual" was not set; "Equal" was stored instead.  
**Expected:** `>=` should map to operator `GreaterOrEqual`. Similarly: `<=` → `LessOrEqual`, `>` → `GreaterThan`, `<` → `LessThan`, `between X and Y` → `Between`.  
**Root cause (suspected):** Phase 5c criteria parser maps the user's natural language expression to an operator enum. The `>=` symbol was likely not in the explicit mapping table, causing fallback to default "Equal".  
**Suggested fix:** Phase 5c criteria builder must include explicit symbol → operator mapping: `>= → GreaterOrEqual`, `<= → LessOrEqual`, `> → GreaterThan`, `< → LessThan`, `= or == or "equals" → Equal`. Symbol mapping must be checked before natural language parsing.

---

## Fix Plan

### Priority 1 — Summarization regression (RB-C, RB-D): states + criteria lost after long conversations

**Problem:** When `summarize_history` fires mid-flow (triggered by longer conversations), Phase 5 loses state normalization and criteria gate. B02 showed both failures.  
**Fix:** Phase 5 resource must be self-contained:
- Normalize states unconditionally at the start of Phase 5 regardless of prior context  
- Gate on additional criteria (Scenario 5.8) must be explicit and non-skippable — placed after `create_delivery_account` call, always executed  
- Do not rely on any Phase 3/4 variables being in working memory

### Priority 2 — Criteria loop premature exit (RB-A): exits after 1 criterion

**Problem:** B01 entered criteria builder but exited after first criterion without prompting for more.  
**Fix:** Phase 5c loop prompt (Scenario 5c.5) must fire unconditionally after every accepted criterion. Ensure the loop state variable (e.g. `criteriaLoopActive`) is not cleared prematurely. The exit path (5c.5c) should only fire when user says "continue"/"done" or clicks Continue — not after any criterion input.

### Priority 3 — Phase 5c load delay (RB-F): doesn't load on first "Add criteria" click

**Problem:** B03 required two attempts to enter criteria because Phase 5c resource wasn't in context on first click.  
**Fix:** In Phase 5, when user selects "Add criteria", the `get_resource(rw-phase-5c-criteria-builder)` call must complete synchronously before any user-facing prompt. Do not display "Please continue with the additional criteria setup" — this is a placeholder that confuses users. Load Phase 5c fully before yielding.

### Priority 4 — Account name prompt (RB-E / RB-K): phase not loaded before responding

**Problem:** B02 and DEBUG23 session both show agent asking "Please provide the delivery account name." before Phase 5 is loaded. Root cause confirmed: agent generates a user-facing message WHILE get_resource is still executing, filling the gap with a generic prompt.  
**Fix:** Add to system prompt (Tier 1): "Before sending any user-facing prompt during a phase transition, confirm the next phase resource has been loaded via get_resource. Never generate a generic collection prompt while awaiting phase instructions."

### Priority 5 — Phase 1 prompt duplication (RB-G): exact prompt echoed twice

**Problem:** Phase 1 first response duplicates the collection prompt. Agent failed to treat STOP AND YIELD as terminal.  
**Fix:** Add to Phase 1 State 1, after STOP AND YIELD: "Send the prompt exactly once. Do not repeat, echo, confirm, paraphrase, or re-emit the prompt in the same turn."  
Also add to system prompt: "When a phase specifies an exact prompt, that prompt is the complete assistant response for that turn."

### Priority 6 — Content type card non-determinism (RB-H): plain text instead of adaptive card

**Problem:** Content type choice rendered as plain text instead of adaptive card. Same issue as general card non-determinism.  
**Fix:** Add per-phase reminder at top of each phase resource that has card-required steps: "All multi-choice prompts MUST use display_adaptive_card — never plain text enumeration."

### Priority 7 — XML parse failure no format re-detection (RB-I)

**Problem:** When XML parse fails on a clearly-JSON body, recovery card offers "Re-paste" or "Switch type" but doesn't auto-detect the actual format.  
**Fix:** In Phase 3 Webhook State 3 (parse failure), before showing recovery card: attempt JSON.parse() and URL-encoded signature check. If detected with high confidence, add a pre-labeled "Switch to [format] (detected)" button.

### Priority 8 — postingInstructions cleared on content type switch (RB-J)

**Problem:** Switching content type erases postingInstructions, forcing re-collection even when the same schema applies.  
**Fix:** Replace `clear postingInstructions` with: retain instructions in memory, and when returning to Step 3 ask "Use same posting instructions or provide new ones?"

### Priority 9 — Criteria loop keyword recognition broken (RB-A variant in B05)

**Problem:** In B05, "add another criterion" was treated as a field name lookup, not a loop control keyword. Loop card also rendered with only "Continue" button (RB-H co-occurring), hiding "Add another" and "See more fields" options.  
**Fix:** Phase 5c State 2 must check for loop control keywords BEFORE attempting field lookup. Explicit keyword set: `["add another", "another", "more", "continue", "done", "skip", "see more fields", "show fields"]`. Match case-insensitive on full input BEFORE field search.

### Priority 10 — Content type skipped when delivery type answered twice (RB-L)

**Problem:** B05 showed that RB-G's duplicate "Webhook" text response causes Phase 3 Webhook state machine to skip the content type question entirely.  
**Fix:** Primary fix: fix RB-G (Priority 5). Secondary: Phase 3 Webhook must guard against duplicate delivery-type inputs — if `contentTypeChoice` is not yet set when proceeding past URL collection, re-enter at content type step.

### Priority 11 — Numeric operator `>=` parsed as "Equal" (RB-M)

**Problem:** "Credit >= 700" stored as "Credit Equal 700" in delivery account. Symbol → operator mapping is incomplete.  
**Fix:** Phase 5c criteria builder must include explicit symbol → operator table: `>= → GreaterOrEqual`, `<= → LessOrEqual`, `> → GreaterThan`, `< → LessThan`, `= / == / equals → Equal`, `between X and Y → Between`. Apply before natural-language operator detection.

### Priority 12 — States question skipped entirely (RB-N)

**Problem:** B07 showed the states question never appeared. After order=No, flow jumped directly to criteria gate, skipping geographic filtering entirely. States then appeared in account summary as CA, AZ, TX (phantoms — never entered by user).  
**Fix:** Phase 5 must ask target states unconditionally as a required step before the criteria gate. Cannot be skipped based on prior state or inferred from context. If no value entered, states must be empty (no defaults).

### Priority 13 — Phantom states in account without user input (RB-O)

**Problem:** B07 account summary showed "Target States: CA, AZ, TX" despite user never being asked or answering the states question. States were prepopulated from workflow context (prior session state or defaults).  
**Agent's root cause (DEBUG):** "The criteria builder path accepted a prepopulated target-states set from the underlying workflow context and carried it into the account summary, even though you did not explicitly answer a states prompt."  
**Agent's patch:** "Make target states a strict ASK field in Phase 5c. If missing, the workflow must prompt the user directly and not allow any inherited/default state list to flow into the summary. Add validation that Phase 6 cannot start unless target states were explicitly captured."  
**Fix:** Phase 5 must treat target states as a strict user-input field. No defaults, no inherited values, no inference from prior context. If states were not collected in this conversation turn, they must be null/empty in `create_delivery_account`.

---

### Finding RB-N — States question skipped entirely in Phase 5

**Phase:** 5 | **Severity:** High | **Frequency:** 4/15 (B07, B11, B14, B15)  
**Observed:** After Order System = No, the states/geography question was not asked. Flow jumped from order decision → criteria gate, with no "Which states should this account receive leads from?" prompt at any point.  
**Expected:** Phase 5 must collect target states before the criteria gate. States are a required field for delivery account configuration.  
**Root cause:** Unknown — possibly a Phase 5 state machine branch that skips states when order=No, or a summarization side-effect corrupting the step sequence. DEBUG after activation could not pinpoint the exact branch since context was partially summarized.  
**Suggested fix:** Add explicit unconditional states collection step in Phase 5 between order decision and criteria gate. Step must not be conditional on any prior state.

---

### Finding RB-O — Phantom states appear in account summary without user input

**Phase:** 5/6 | **Severity:** Critical | **Frequency:** 4/15 (B07, B11, B14, B15 — co-occurring with RB-N)  
**Observed:** Delivery account summary showed "Target States: CA, AZ, TX" — values the user never entered. The states question was skipped (RB-N), yet the account was created with CA, AZ, TX.  
**Expected:** If states were not collected, the account should have no geographic filter (all states) or an explicit empty value — not phantom data.  
**Agent's DEBUG analysis:** "The criteria builder path accepted a prepopulated target-states set from the underlying workflow context and carried it into the account summary, even though you did not explicitly answer a states prompt in this turn. The data collection rule that failed: 'For any ASK field, do not generate, infer, reuse, or apply defaults—use only values the user explicitly provides for that.'"  
**Agent's patch:** "Make target states a strict ASK field in Phase 5c. If missing, the workflow must prompt the user directly and not allow any inherited/default state list to flow into the summary. Add validation that Phase 6 cannot start unless target states were explicitly captured."  
**Suggested fix:** Phase 5 must validate that states value was captured in the current conversation turn before calling `create_delivery_account`. Prior session state or default values must never be used.  
**Session log verification (69d1503f2697c9202c0bc0c6):** CONFIRMED — `create_delivery_account` actual payload: `criteria:[{"leadFieldUID":144881,"operator":"In","value":"5|3|44"}]` where UID 5=CA, UID 3=AZ, UID 44=TX. These phantom states were submitted to the real API. Not a display hallucination.

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

### Finding RB-P — create_delivery_account silently skipped; update_delivery_account fails on non-existent account

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

### Finding RB-Q — "Show more fields" button in P5c triggers field-not-found error

**Phase:** 5c | **Severity:** Medium | **Frequency:** 1/8 (B08 — only run where "Show more fields" button was needed)  
**Observed:** After failing to find a "State" field, clicking the "Show more fields" button in the P5c card resulted in "I couldn't find that field. Please type the field name..." error — the same error as a failed field name lookup. The button was not working as intended.  
**Expected:** "Show more fields" button should call the field-listing tool and display the next set of available fields as a list.  
**Workaround confirmed:** Typing "show fields" as text worked correctly — agent showed "Additional Fields (showing up to 10): Address, Email".  
**Root cause (suspected):** The button value submitted by "Show more fields" click is being parsed through the field name lookup path instead of the show-fields command path. The `Action.Submit` data payload may be sending field text as literal input rather than a command signal.  
**Related:** This is similar to the Action.Submit `data` field bug (F1/F14 — ChoiceSet + Action.Submit must NOT have a data field). The button's data payload may be incorrectly set.  
**Suggested fix:** "Show more fields" button in P5c must submit a command code that routes to the field-listing handler, not through the field name lookup. Check the `Action.Submit` data object for this button — ensure it signals "show_more_fields" or equivalent, not the button label text.

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

### Finding RB-R — Portal delivery uses deliveryType "HttpPost"; no deliveryAddress field

**Phase:** 3 (Portal) | **Severity:** Informational (correctness to understand) | **Frequency:** 1/1 Portal runs (B09)  
**Observed:** In `create_delivery_method` for Portal delivery, the actual API payload uses `deliveryType: "HttpPost"` with no `deliveryAddress` field at all.  
**DEBUG response hallucination:** During B09 DEBUG session, agent reported `deliveryAddress: "Portal"` in its self-reported payload. Session log confirmed this was hallucinated — the actual call had no such field.  
**Implication:** Portal delivery does not require a URL. The `deliveryType: "HttpPost"` value appears to be the system's internal representation for Portal. Instructions should not mention `deliveryAddress` for Portal type.  
**Note:** This is likely by design. Documents the API contract for Portal delivery.

---

### Finding RB-S — Numeric P5c criteria saved via update_delivery_account with no UI acknowledgment

**Phase:** 5c | **Severity:** Medium | **Frequency:** 1/1 conversational-P5c runs (B09)  
**Observed:** In B09, numeric criteria (Credit >= 700, LoanAmount >= 50000, 1stMortgageBalance <= 300000) were processed and saved via `update_delivery_account` calls — but no user-visible confirmation was shown. Agent silently executed the tool call and continued collecting the next criterion without any "✓ Criterion added" message or similar acknowledgment.  
**Expected:** After each criterion is saved, the agent should confirm: "✓ Added: [criterion description]. Would you like to add another, or continue?"  
**Root cause:** Phase 5c in conversational fallback mode (Phase 5c resource not loaded) processes and saves criteria internally without triggering any display_adaptive_card or structured acknowledgment step.  
**Suggested fix:** Phase 5c instructions must include an explicit acknowledgment step after every successful `update_delivery_account`: show a brief confirmation message naming the criterion just saved, then prompt for next action.

---

### Finding RB-T — "continue" after P5c puts agent in confused state instead of proceeding to P6

**Phase:** 5c → P6 transition | **Severity:** High | **Frequency:** 1/1 conversational-P5c runs (B09)  
**Observed:** After all criteria were entered, typing "continue" caused the agent to respond: "Please share the exact Phase 6 summary wording you want checked" — as if it expected a review task, not a transition to P6.  
**Expected:** "continue" in P5c should trigger Phase 6 summary generation and display.  
**Workaround:** Typing "Show the delivery account summary now and proceed to activation" worked — agent generated P6 summary correctly.  
**Root cause:** When Phase 5c runs in conversational fallback mode (resource not loaded), the exit signal "continue" is not recognized as the P5c→P6 transition trigger. The agent interprets it ambiguously within a confused state.  
**Suggested fix:**
1. Primary fix: ensure Phase 5c resource loads correctly (fix RB-F) — eliminates conversational fallback mode entirely.
2. Secondary: add explicit "continue" → P6 routing in Phase 5c exit. The word "continue" at criteria exit should never prompt the user to share summary wording.

---

### Finding RB-U — update_delivery_account calls are non-cumulative; earlier criteria may be overwritten

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

### Finding RB-V — Conversational mode criteria accepted verbally but never sent to API

**Phase:** 5c (conversational fallback post-DEBUG) | **Severity:** Critical | **Frequency:** 1/1 (B11 — only run where DEBUG triggered mid-P5)  
**Observed:** After DEBUG intervention, agent asked "Please provide any additional criteria for this delivery account, or reply with none." User replied "LoanAmount >= 50000, CreditScore >= 650." Agent said "Confirmed." — but session log shows **no `update_delivery_account` call** after this exchange. The criteria were silently discarded.  
**Expected:** Accepted criteria must be saved via `update_delivery_account` before proceeding to P6.  
**Root cause:** When the agent enters a conversational state outside normal Phase 5c flow (triggered by DEBUG disruption), it collects user input but lacks the Phase 5c instruction context needed to know it should call `update_delivery_account`. The "Confirmed." response is generated by the base model without any tool execution.  
**Suggested fix:** Phase 5c must be loaded as a resource (not run conversationally) before collecting criteria. Any criteria confirmed by the agent must always trigger `update_delivery_account` — enforce this with a rule in system prompt: "If you have confirmed criteria with the user, you MUST call update_delivery_account before proceeding. Verbal confirmation without tool call is a violation."

---

### Finding RB-W — State UID missing from states value (NY dropped)

**Phase:** 5 | **Severity:** High | **Frequency:** 1/1 (B11 — first run where NY was included in states)  
**Observed:** User provided FL, TX, CA, NY. `create_delivery_account` payload shows `value: "5|3|44"` — only 3 UIDs. NY (UID 32) was absent despite being confirmed by agent.  
**Expected:** All 4 states should appear in the pipe-delimited value: `"5|3|44|32"`.  
**Root cause:** Unknown — possibly `get_usa_states` response for NY was not matched, or the value concatenation loop dropped the last entry, or NY was normalized to a name not matching any state in the response.  
**Suggested fix:** After building the states value string, validate that count of pipe-separated UIDs equals count of user-provided states. Log any state that could not be matched. If any state is missing, prompt user to confirm or skip.

---

### Finding RB-X — P5c empty-response loop with large lead types (LendingTree)

**Phase:** 5c | **Severity:** Critical | **Frequency:** 1/1 (B12 — first LendingTree run with criteria)  
**Observed:** After user clicks "Add criteria" in the criteria gate, the agent enters an unrecoverable empty-response loop. First response takes 14+ minutes then renders empty bubble. Follow-up messages also produce empty bubbles within 1–3 min. Three consecutive empty responses observed before run was abandoned. The criteria builder never presented any fields to the user.  
**Expected:** Agent should load Phase 5c instructions, fetch LendingTree fields, and present a conversational or card-based criteria builder within a normal response time.  
**Root cause (hypothesis):** LendingTree has 100+ fields. Phase 5c instructions tell the agent to call `get_lead_type_fields` which returns the full field list. When combined with the conversation history at this point in the flow (Phase 1–5 + all user turns + the large field payload), the total token count exceeds the model context window (128k). The model then produces an empty completion. Each subsequent user message triggers the same tool call pattern, causing a repeat loop.  
**Context evidence:** Session `69d18c262697c9202c0bc293` — network request `add-user-message-and-process` returned 200 but chat bubble stayed empty. No new requests seen after clearing network history. No error messages in DOM.  
**Suggested fix (short-term):** Phase 5c should NOT eagerly load all lead type fields upfront. Instead: (1) present user with a free-text prompt to describe criteria, (2) only look up specific fields when the user names a field, and (3) avoid fetching the full field list as a prerequisite for displaying the criteria builder. Alternatively: implement field pagination — fetch only the first N fields, offer "see more" to fetch next batch.

---

### Finding RB-Y — Phase 3 asks name+type via plain text before showing schedule/type cards

**Phase:** 3 | **Severity:** Medium | **Frequency:** 2/15 (B13 aborted, B15 — different manifestation: B13=name+type before cards; B15=delivery type plain text before schedule+type cards, then delivery type card shown again)  
**Observed:** At Phase 3 entry, agent asked "Please provide the delivery method name and the delivery method type." via plain text (duplicated in same message) BEFORE showing the schedule selection card. In all prior runs, the Phase 3 card flow showed schedule card first, delivery type card second, then collected name. In B13, the agent solicited name+type first as a text question, then showed the schedule and delivery type cards in sequence after receiving the answer.  
**Expected:** Phase 3 should show the schedule card first (24/7 or specific hours), then delivery type card, then ask for method name after the type is known.  
**Root cause (hypothesis):** Non-deterministic step ordering within Phase 3. The agent chose to ask for name+type (a later step) before presenting the initial cards (earlier steps). This may occur when Phase 3 is loaded fresh from the MCP resource and the model interprets the first explicit "collect" instruction before the implicit "show card first" instruction.  
**Additional anomaly:** The "Please provide the delivery method name and the delivery method type." prompt appeared twice in the same message (double-render), consistent with RB-G.  
**Robustness note:** When an unexpected input (webhook URL) was sent before the agent had issued its prompt for it, the agent rolled back to the start of Phase 3 (asked schedule question again). This suggests the state machine is sensitive to out-of-order inputs — see B13 test protocol notes.  
**Suggested fix:** Phase 3 entry must enforce card-first ordering: display schedule card, await selection, then display delivery type card, await selection, then (if Webhook/FTP/Email) ask for name. Name collection must come AFTER type is known, not before.


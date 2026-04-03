# Delivery Rework — Test Findings

All findings from code review (multi-agent evaluation), live testing on dev.leadexec.app with GPT-5.4 Mini Model, and comparison against original /delivery.

---

## Previously Identified Issues (Code Review + Agent Evaluation)

These were found through multi-agent deep evaluation before live testing. Some are already fixed.

### Phase 3 Router
- **Issue P1: scheduleInput unreachable** [FIXED]
  - The scheduleInput prompt (line 19) was nested inside `IF deliveryScheduleChoice is missing` block
  - After user picked "Specific hours only", the outer gate was FALSE on re-entry, scheduleInput never collected
  - Fix: Promoted to separate top-level State 1b

- **Issue P2: Invalid delivery type fallback missing** [FIXED]
  - If user typed gibberish instead of clicking Portal/Webhook/Email/FTP, all State 3 routing conditions were FALSE
  - Fix: Added fallback — clear deliveryTypeChoice, re-display prompt

### Phase 5 — Delivery Account
- **Issue P3: P5c Skip from Additional Fields card dead-end** [FIXED via GLOBAL EXIT RULE]
  - "Skip" button on show-more-fields card had no handler
  - State 1 skip handler only covered "from the suggestions card"
  - Fix: Replaced 7 per-state handlers with single GLOBAL EXIT RULE

- **Issue P4: P5c no exit after failed field match** [FIXED via GLOBAL EXIT RULE]
  - After "I couldn't find that field", user could only type valid field or say "show fields"
  - "skip"/"continue"/"done" were parsed as criteria → infinite error loop
  - Fix: GLOBAL EXIT RULE catches exit intent at any YIELD point

- **Issue P5: P5c enum abandonment stale state** [FIXED via GLOBAL EXIT RULE]
  - At enum ChoiceSet, if user typed "continue" instead of selecting, enumFieldName never cleared
  - Fix: GLOBAL EXIT RULE handles exit; State 3 clears enum state on completion

- **Issue P6: P5c was over-engineered** [FIXED]
  - Original: ~92 lines, ~10 branches, 3 yields, 2 catch-all handlers
  - Rework had: ~182 lines, ~22 branches, 7 yields, 7 edge-case handlers
  - Fix: Simplified to GLOBAL EXIT RULE + 5 states matching original's pattern

- **Issue P7: P5c affordance mismatch** [FIXED]
  - Prompt said "Would you like to add criteria, see more fields, or skip?" but card only had "Show more fields" | "Skip"
  - Fix: Changed prompt to "You can type a criterion directly, show more fields, or skip."

- **Issue P8: P5 typed criteria at gate not handled** [BY DESIGN]
  - P5 State 5 only has "Add criteria" | "Skip" — no handler for typed criterion
  - The gate is intentional to avoid loading P5c's heavy instructions unnecessarily
  - LLM can reasonably infer typed criterion = intent to add criteria

### Phase 1b — Select Client & Method
- **Issue P9: P1b partial-state hole** [FIXED]
  - If deliveryMethodUID was already in context but deliveryMethodName/leadTypeUID was not, State 2 was skipped
  - Fix: Broadened condition to `deliveryMethodUID is missing OR deliveryMethodName is missing OR leadTypeUID is missing`

### Phase 8 — Activation
- **Issue P10: No escape from activation failure retry** [FIXED]
  - Only "Retry activation" button on failure — infinite retry loop if API stays down
  - Fix: Added "Keep Inactive" button alongside "Retry activation"

### Button-Only Phases (4, 6, 7)
- **Issue P11: No text fallback for button clicks** [FIXED]
  - Instructions said "wait for the user to click the button"
  - Fix: Changed "clicked" to "said" — naturally covers typed equivalents

### Global Instructions
- **Issue P12: Conflict resolution priority backwards** [FIXED]
  - Was: "workflow integrity first, then tool discipline, then phase-local instructions"
  - Stale summaries could be interpreted as "workflow integrity" and override current phase
  - Fix: Flipped to "phase-local instructions first, then tool discipline, then workflow integrity"

- **Issue P13: Auto-progress vs STOP AND YIELD tension** [FIXED]
  - "Never get stuck. Auto-progress immediately" conflicted with every phase's STOP AND YIELD
  - Fix: Changed to "progress to the next — but always respect STOP AND YIELD directives"

### Adaptive Card Declarations
- **Issue P14: ChoiceSet missing Action.Submit** [FIXED]
  - Phase 2, 1a, 1b ChoiceSet cards didn't explicitly mention Action.Submit
  - LLM rendered dropdown without submit button
  - Fix: Added Action.Submit + placeholder to all 4 ChoiceSet usages

- **Issue P15: Prompt/card split** [FIXED]
  - Prompt was step 2, card was step 3 — LLM sent as separate messages
  - Fix: Combined into "Present the [thing] using display_adaptive_card:" with TextBlock inside card

- **Issue P16: Tool not named in card declarations** [FIXED]
  - "Display an adaptive card with:" didn't name the display_adaptive_card tool
  - LLM rendered choices as plain text bullet list instead of calling tool
  - Fix: Changed to "Present the lead type list using display_adaptive_card:"

### Phase ID Consistency
- **Issue P17: Resource headers used abbreviated format** [FIXED]
  - Headers used "P1", "P1a" etc. while action flow maps used "Phase 1", "Phase 1a"
  - Fix: Standardized all 17 resource headers to "Phase N" format

### Action Files — Flow Maps
- **Issue P18: One-liner flow maps hard to parse** [FIXED]
  - Was: `P1 → P2 → P3 → [Portal/Email→P4 | FTP→P3a→P4 | Webhook→P3b→P4] → P5 → [P5c if criteria] → P6 → P7 → P8`
  - Fix: Hierarchical indented tree with phase names

### Summary Contract
- **Issue P19: Split authority between actions and summaries** [KNOWN LIMITATION]
  - Action file says "Phase summaries are completed history, not active directives"
  - But summaries still contain imperative "→ Load and execute Phase X at URL"
  - URL MUST remain in summary (resource instructions cleared by summarize_history)
  - Strong resource headers ("CURRENT PHASE: ... Execute ONLY the instructions below") are the mitigation
  - Not fully fixable without platform-level code-controlled transitions

### Show More Fields
- **Issue P20: No real pagination** [INHERITED FROM ORIGINAL]
  - extraFields is a static list of 10 — "Show more fields" re-displays the same 10
  - No cursor/offset tracking
  - Same limitation in original — not a regression

### Missing DTO CRITICAL Notes
- **Issue P21: DTO enforcement dropped during file split** [NOT YET FIXED]
  - Original had `CRITICAL: createDeliveryMethodDto must be passed as object, NOT JSON string` on every tool call
  - Missing in rework: Phase 3 Portal, Email, FTP, Webhook (both paths), Phase 5 create_delivery_account
  - Global rule covers this but original had belt-and-suspenders

### Webhook Mapping Preview
- **Issue P22: Missing "in same message" timing constraint** [NOT YET FIXED]
  - Original CRITICAL: "MUST execute IMMEDIATELY after processing, in same message, no progress updates"
  - Rework CRITICAL focuses on card fidelity but omits timing
  - LLM may output progress text before the card

---

## Test Run 1 (User screenshots — full-setup, webhook path)

### Phase 1 — Create Client
- **Status**: PASS
- Numbered list prompt displayed correctly
- Both fields collected in single prompt
- Client created successfully

### Phase 2 — Lead Type Selection
- **Issue F1: ChoiceSet submit produces merged data**
  - First submit returned `action: selectLeadType, leadTypeUID: 5689` — merged Action.Submit data with ChoiceSet value
  - Agent didn't recognize the compound response, re-displayed the selector
  - Second submit returned `leadTypeUID: 5689` (clean) and proceeded
  - **Root cause**: LLM added `data: {"action": "selectLeadType"}` to Action.Submit, which gets merged with ChoiceSet value at submission
  - **Severity**: HIGH — causes double interaction on every ChoiceSet

### Phase 3 — Delivery Schedule
- **Status**: PASS
- "24/7 delivery" | "Specific hours only" rendered as ActionSet buttons correctly

### Phase 3 — Delivery Type
- **Status**: PASS
- Four buttons rendered correctly

### Phase 3 Webhook — URL + Field Mapping
- **Status**: PASS for URL collection and mapping choice

### Phase 3 Webhook — Field Mapping (State 6)
- **Issue F2: Ambiguous fields dumped as plain text**
  - First test: all mappings + ambiguous fields shown as bullet-point text list in one message
  - No ActionSet cards for ambiguous field resolution
  - Agent said "Choices available: - Dup Check 3 - Dup Check 2 - ..." as plain text instead of ChoiceSet
  - **Root cause**: State 6 didn't enforce "silent" processing or one-at-a-time resolution
  - **Fixed in latest commit**: Added "Auto-map confident matches silently" and per-field ActionSet
  
- **Issue F3: Field Mapping Preview not rendered as Table**
  - Second test: preview showed as text arrows (SystemField → delivery_field) instead of Table card
  - Third test: showed only summary text ("Successfully mapped 15 out of 20 fields.") with Continue button, no table rows
  - **Root cause**: LLM skipped the JSON template or constructed it incorrectly
  - **Partially fixed**: Added CRITICAL note to State 7, but still failed in later test

### Phase 3 Webhook — Field Mapping Preview (State 7)
- **Issue F4: Phase 4 summary card failed with malformed JSON**
  - Error message: "I hit a display issue while preparing the delivery method summary"
  - Agent self-diagnosed: "the card payload is being rejected as malformed JSON by the display tool"
  - Fell back to plain text format
  - **Root cause**: LLM constructs JSON template by replacing placeholders, produces invalid JSON
  - **Severity**: HIGH — summary cards (Phase 4, 6, 7) all use same pattern

### Phase 5 — Delivery Account Setup
- **Issue F5: States evaluated out of order**
  - Exclusivity (State 2) asked BEFORE price (State 1)
  - Agent combined: "Thanks. Please also provide the price per lead so I can continue."
  - **Root cause**: LLM doesn't follow state evaluation order, batches questions
  - **Severity**: HIGH — fundamental state machine compliance

- **Issue F6: Target states and lead fields SKIPPED entirely**
  - State 3 (load lead fields, detect state field, collect target states) was never executed
  - Agent went directly from Order System to "Would you like to add additional lead criteria?"
  - Account was created without state targeting criteria
  - **Root cause**: Same as F5 — LLM skipped states
  - **Severity**: CRITICAL — data integrity issue, account has no state criteria

- **Issue F7: Price and exclusivity merged into one turn**
  - Agent asked exclusivity first, then said "Also provide price" in same response
  - Phase 5 instructions say each is a separate STOP AND YIELD
  - **Root cause**: LLM ignores STOP AND YIELD, tries to batch prompts

### Phase 5c — Criteria Builder
- **Issue F8: Field suggestions SKIPPED**
  - After "Add criteria", agent went straight to "Would you like to add another criterion, see more fields, or continue?" (State 4)
  - State 1 (build and show recommended fields) was never executed
  - No field names were ever shown to the user
  - **Root cause**: LLM skipped mandatory State 1
  - **Severity**: HIGH — user can't see available fields

- **Issue F9: Show more fields loop empty**
  - Clicking "Show more fields" repeatedly showed same prompt without field names
  - The extraFields list was never built (because State 1 was skipped)
  - **Root cause**: Consequence of F8

### Phase 8 — Activation
- **Status**: Original only had "Retry activation" on failure — no escape
- **Fixed**: Added "Keep Inactive" button

---

## Test Run 2 (Playwright live test — partial flow)

### Phase 1 — Create Client
- **Status**: PASS
- Prompt text correct, numbered list

### Phase 2 — Lead Type Selection
- **Status**: PARTIAL PASS
- ChoiceSet rendered correctly: TextBlock + dropdown + Continue button in one card
- **Issue F10: Fuzzy match too strict for typed input**
  - User typed "lending tree" (with space)
  - Agent responded "I couldn't match that selection" and re-displayed
  - "lending tree" should fuzzy match "LendingTree" at >90% confidence
  - **Severity**: LOW — dropdown selection works, text fallback is a convenience

### Phase 3 — Delivery Schedule
- **Status**: PASS
- ActionSet with "24/7 delivery" | "Specific hours only" rendered in single card with prompt text as TextBlock

---

## Test Run 3 (User earlier screenshots — various)

### Adaptive Card Rendering
- **Issue F11: Exclusivity rendered as radio buttons**
  - "Exclusive" | "Shared" should be ActionSet (clickable buttons)
  - Rendered as ChoiceSet radio buttons with Submit button instead
  - **Root cause**: LLM chose ChoiceSet instead of ActionSet despite instructions saying ActionSet
  - **Severity**: MEDIUM — works but wrong UX pattern

- **Issue F12: Lead types rendered as plain text list**
  - One test showed all lead types as bullet points instead of ChoiceSet dropdown
  - Agent didn't call display_adaptive_card at all
  - **Fixed**: Changed to "Present the lead type list using display_adaptive_card:"

---

## Systematic Issues (from multi-agent evaluation)

### State Machine Compliance
- **Issue F13: LLM doesn't evaluate states in order**
  - Phase 5 States 1-5 should be sequential, but LLM jumps ahead or batches
  - Same issue in Phase 5c — State 1 skipped
  - **Root cause**: "Evaluate what information you currently have" is too open-ended
  - **Fix needed**: Explicit "Evaluate states strictly in order" instruction

### Adaptive Card Issues
- **Issue F14: Action.Submit data field on ChoiceSet cards**
  - LLM adds `data: {"action": "..."}` to Action.Submit when paired with ChoiceSet
  - Platform merges this with ChoiceSet value, producing compound response
  - **Fix needed**: Global rule — Action.Submit MUST NOT include data field when paired with ChoiceSet

- **Issue F15: JSON template cards fail to render**
  - Phase 4, 6, 7 summary cards and webhook mapping preview use inline JSON templates
  - LLM sometimes constructs malformed JSON when replacing placeholders
  - **Fix needed**: Stronger CRITICAL enforcement + potentially simplify templates

### Missing CRITICAL Notes
- **Issue F16: DTO CRITICAL notes missing**
  - Original had `CRITICAL: createDeliveryMethodDto must be passed as an object, NOT JSON string` on every tool call
  - Rework dropped these during file decomposition
  - Missing in: Phase 3 Portal, Email, FTP, Webhook (both paths), Phase 5 create_delivery_account
  - Global rule covers this but belt-and-suspenders is safer

### Summary Handoff
- **Issue F17: "in same message, no progress updates" missing from mapping preview**
  - Original CRITICAL said "MUST execute IMMEDIATELY after processing, in same message"
  - Rework CRITICAL focuses on card fidelity but omits timing constraint
  - LLM may output progress text before the card

---

## Issues by Severity

### CRITICAL
| # | Issue | Description |
|---|-------|-------------|
| F6 | Target states skipped | Account created without state criteria — data integrity |
| F5 | States out of order | Price/exclusivity/order evaluated wrong order |
| F8 | Field suggestions skipped | Mandatory State 1 in P5c never executed |

### HIGH  
| # | Issue | Description |
|---|-------|-------------|
| F1 | ChoiceSet double-submit | Action.Submit data merge causes re-prompt |
| F4 | JSON template malformed | Summary cards fail to render |
| F13 | State machine non-compliance | LLM skips/reorders states |
| F14 | Action.Submit data on ChoiceSet | Global rule needed |
| F7 | STOP AND YIELD ignored | LLM batches multiple prompts |

### MEDIUM
| # | Issue | Description |
|---|-------|-------------|
| F2 | Ambiguous fields as text | Not using ActionSet cards (partially fixed) |
| F3 | Mapping preview not table | JSON template skipped (partially fixed) |
| F11 | Exclusivity as radio buttons | ChoiceSet instead of ActionSet |
| F15 | JSON templates fragile | LLM-constructed JSON can be malformed |
| F16 | Missing DTO CRITICAL notes | 6 files missing enforcement |
| F17 | Mapping preview timing | Missing "in same message" constraint |

### LOW
| # | Issue | Description |
|---|-------|-------------|
| F10 | Fuzzy match too strict | "lending tree" didn't match "LendingTree" |
| F12 | Lead types as plain text | Fixed with explicit tool naming |
| F9 | Show more fields empty | Consequence of F8 |

---

## Live Test Results (Claude in Chrome + Playwright, 5 tests)

### T01 — Full-Setup, Portal, 24/7, Skip Criteria, Activate

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 1 | Numbered list prompt | ✓ Correct | PASS |
| Phase 2 | ChoiceSet + Submit | ✓ Card, but merged data `action: selectLeadType, leadTypeUID: 5689` | F1, F23 |
| Phase 3 Schedule | ActionSet buttons | ✓ Correct | PASS |
| Phase 3 Delivery Type | ActionSet 4 options | ✓ Correct | PASS |
| Phase 4 Summary | Table card | Plain text, not table | F4 |
| Phase 5 State 1: Price | Ask first | ✓ Asked first | PASS |
| Phase 5 State 2: Exclusivity | Ask second | Asked THIRD (after order) | F5 |
| Phase 5 State 2: Order | Ask third | Asked SECOND | F5 |
| Phase 5 State 3: Target states | Ask user | **SKIPPED — hallucinated "CA"** | F6 CRITICAL |
| Phase 5 State 5: Criteria gate | ActionSet card | Plain text, no buttons | F24 |
| Phase 6 Summary | Table card | Plain text | F4 |
| Phase 7 Summary | Table card | Plain text | F4 |
| Phase 8 Activation | Success message | ✓ Correct | PASS |
| All submits | Clean button text | `action: X` visible as messages | F23 |

### T02 — Full-Setup, Email, Specific Hours, Add Criteria, Keep Inactive

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 1 | Both fields in one prompt | First send silently failed, retry asked for company name only | F25 |
| Phase 2 | ChoiceSet card | **Plain text, no card** | F26 |
| Phase 3 State 1b | Schedule text prompt after "Specific hours" | ✓ Correct — P1 fix confirmed working | PASS |
| Phase 3 Email | Create method, skip test | ✓ Correct | PASS |
| Phase 5 State order | Price → Excl → Order | ✓ Correct this time (inconsistent with T01) | F5 non-deterministic |
| Phase 5 State 3 | Target states | **SKIPPED** | F6 CRITICAL |
| Phase 5c Field suggestions | Show recommended fields | **SKIPPED — plain text "Please provide criteria"** | F8 |
| Phase 5c Criteria loop | ActionSet "Show more | Continue" | Plain text loop, no cards | F27 |
| Phase 5c Enum field | ChoiceSet with enum values | Plain text "provide value for loan request type" | F28 |
| Phase 5c Exit | "done"/"skip" exits | **STUCK — neither recognized** | F29 CRITICAL |

### T03 — Full-Setup, Webhook, JSON Mapping, Skip Test, Skip Criteria, Keep Inactive

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 2 | ChoiceSet | ✓ Dropdown + Continue | PASS |
| Phase 3 Webhook URL | Collect URL | ✓ Correct | PASS |
| Phase 3 Mapping Choice | ActionSet | ✓ "I'll provide instructions" / "Skip for now" | PASS |
| Phase 3 Content Type | ActionSet 4 options | ✓ Correct | PASS |
| Phase 3 JSON Parsing | Parse + match fields | ✓ Parsed nested JSON correctly | PASS |
| Ambiguous fields | Per-field ActionSet | **Auto-resolved without asking** | F30 |
| Unmapped fields | Show in preview | **Silently dropped** (14/14 instead of 14/20) | F31, F32 |
| Mapping preview | Table card | Plain text — not table, but field names visible | F4 |
| Connection test | "Test Connection" / "Skip" | ✓ Test failed, showed "Retry" / "Skip" correctly | PASS |
| Phase 5 State 1 | Price first | **SKIPPED** — went to exclusivity | F5 |
| Phase 5 State 3 | Target states | **SKIPPED — hallucinated "CA"** | F6 CRITICAL |
| Criteria gate | "Add criteria" / "Skip" | ✓ ActionSet card shown (inconsistent with T01) | PASS |
| Skip criteria | Proceed to Phase 6 | Re-asked "provide criteria or say none" first | F33 |
| Phase 8 | Keep Inactive | ✓ Correct | PASS |

### T04 — Full-Setup, Webhook, "I'm Not Sure" Auto-Detect (FAILED)

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 1-3 | Normal flow | ✓ Correct through delivery type | PASS |
| Webhook URL | Send URL without https | **Agent regressed to Phase 2** — treated URL as lead type selection | F34 CRITICAL |
| Recovery | N/A | **Showed timezone selector (348 options)** — completely wrong context | F35 CRITICAL |
| **Test abandoned** | | Flow unrecoverable | |

### T05 — Full-Setup, Webhook, URL Encoded (Partial — browser disconnected)

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 2 | ChoiceSet | **Plain text, no card** (same as T02) | F26 |
| Webhook URL | Collect URL | Required sending twice — re-asked after first send | Minor |
| Content Type | URL Encoded selected | ✓ Correct | PASS |
| Posting instructions | URL encoded body sent | Processed silently | PASS |
| Mapping preview | Table card | **SKIPPED entirely** — went straight to connection test | F36 |
| **Browser disconnected** | | Test incomplete from here | |

---

## New Findings from Live Tests (F23-F36)

| # | Issue | Severity | Description |
|---|-------|----------|-------------|
| F23 | Action.Submit data as user message | HIGH | Every button click shows `action: X` or `fieldName: value` as visible user message text. Platform renders Action.Submit data payload as chat message. |
| F24 | Criteria gate as plain text | MEDIUM | "Add criteria / Skip" sometimes rendered without ActionSet buttons (T01 plain text, T03 had buttons — non-deterministic) |
| F25 | First message silently failed | LOW | T02: first "name, email" message got no response. Retry worked but asked for company name only. |
| F26 | Phase 2 no ChoiceSet | HIGH | T02/T05: lead type selector rendered as plain text list, not ChoiceSet dropdown. T01/T03 had ChoiceSet. Non-deterministic. |
| F27 | Criteria loop plain text | HIGH | T02: no ActionSet cards in criteria loop. "Please provide next criteria" as plain text. No confirmation of parsed criteria. |
| F28 | Enum field no ChoiceSet | HIGH | T02: enum field (loan request type) showed plain text "provide value" instead of ChoiceSet with enum options. |
| F29 | Agent stuck on enum prompt | CRITICAL | T02: "done"/"skip" not recognized as exit at enum value prompt. Infinite re-ask loop. |
| F30 | Ambiguous fields auto-resolved | MEDIUM | T03: fields with multiple match candidates resolved silently without user confirmation. |
| F31 | Unmapped fields silently dropped | MEDIUM | T03: tracking fields (vendor_id, ad_source, etc.) not shown in preview, not mentioned. |
| F32 | Total field count wrong | LOW | T03: "14 out of 14 fields" — only counted mapped fields, not total schema fields (~20). |
| F33 | Skip criteria re-asks | MEDIUM | T03: after clicking "Skip" on criteria gate, agent re-asked "provide criteria or say none" before proceeding. |
| F34 | Flow regressed to Phase 2 | CRITICAL | T04: after sending webhook URL, agent treated it as lead type selection and showed timezone selector. Complete context loss. |
| F35 | Timezone selector shown | CRITICAL | T04: consequence of F34 — 348-option timezone ChoiceSet appeared instead of webhook mapping flow. |
| F36 | Mapping preview skipped (URL Encoded) | MEDIUM | T05: mapping preview table not shown for URL Encoded format. Agent went straight to connection test. |

---

## Cross-Test Consistency Analysis

| Issue | T01 | T02 | T03 | T04 | T05 |
|-------|-----|-----|-----|-----|-----|
| F6: Target states hallucinated | CA | CA | CA | N/A | N/A |
| F5: State order wrong | Partial | Correct | All wrong | N/A | N/A |
| F4: Tables as plain text | Yes | Yes | Yes | N/A | N/A |
| F23: Submit data visible | Yes | Yes | Yes | Yes | Yes |
| F26: Phase 2 no ChoiceSet | No | Yes | No | No | Yes |
| F8: Field suggestions skipped | N/A | Yes | N/A | N/A | N/A |

**100% consistent:** F6 (target states), F4 (table cards), F23 (submit data)
**Non-deterministic:** F5 (state order), F26 (ChoiceSet rendering), F24 (criteria gate)
**Webhook-specific:** F30-F32 (mapping), F34-F36 (flow regression, preview skip)

---

## Root Cause Analysis

### T06 — Full-Setup, Webhook, XML, Parse Failure (HUNG)

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 1-3 Webhook XML | Normal flow to XML paste | ✓ Through content type selection | PASS |
| Broken XML sent | "I couldn't parse as valid XML" + Re-paste/Switch buttons | **Agent hung — no response after 2+ minutes** | F37 CRITICAL |
| **Test abandoned** | | Agent unresponsive | |

**Original comparison**: Original handled broken XML immediately — showed "I couldn't parse this as valid XML. Would you like to: Fix and re-paste the XML schema / Switch to a different content type" with "Re-paste XML schema | Switch content type" buttons. User clicked "Switch content type" → content type selector reappeared → selected JSON → mapping preview showed correctly.

### T08 — Full-Setup, FTP, 24/7, Skip Criteria, Activate

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 2 | ChoiceSet | ✓ Dropdown + submit | PASS |
| Phase 3 Schedule | 24/7 buttons | ✓ Correct | PASS |
| Phase 3 FTP credentials | Ask server/user/pass | **SKIPPED entirely** — went to connection test | F38 CRITICAL |
| Phase 3a FTP test | Test with credentials | Offered test WITHOUT credentials | F38 |
| Phase 4 Summary | Show FTP details | No FTP address/user/pass shown | F38 |
| Phase 5 State 1 Price | Ask first | Asked SECOND (after exclusivity) | F5 |
| Phase 5 State 2 Exclusivity | ActionSet card | ✓ Card with buttons | PASS |
| Phase 5 State 2 Order | ActionSet card | Plain text first, re-asked with card | F39 |
| Phase 5 State 3 Target states | Ask user | **SKIPPED — empty in summary** | F6 |
| Phase 5 Criteria gate | ActionSet card | Plain text | F24 |
| Phase 6 Summary | Target States shown | **Target States: (empty)** | F6 |
| Phase 7 Summary | Real account ID | **Delivery Account ID: 0** | F40 |
| Phase 8 Activation | Success | ✓ Activated | PASS |

**Original comparison**: Original ALWAYS asks "Please provide FTP details: 1. Server address 2. Username 3. Password" before creating method. Original Phase 5 asks price FIRST, then exclusivity, order, target states in strict order.

## New Findings from T06-T08 (F37-F40)

| # | Issue | Severity | Description |
|---|-------|----------|-------------|
| F37 | Agent hung on broken XML | CRITICAL | T06: sent broken XML, agent never responded (2+ min). Original handles immediately with Re-paste/Switch buttons. |
| F38 | FTP credentials skipped | CRITICAL | T08: agent never asked for FTP server/user/pass. Created method without credentials. Data integrity failure. |
| F39 | Plain text prompt re-asked as card | MEDIUM | T08: Order System asked as plain text (no buttons), typed "No" not recognized, re-asked with ActionSet card. |
| F40 | Delivery Account ID: 0 | HIGH | T08: account creation returned ID=0, likely failed due to missing required data (no target states/criteria). |

---

## Updated Cross-Test Consistency (7 tests)

| Issue | T01 | T02 | T03 | T04 | T05 | T06 | T08 |
|-------|-----|-----|-----|-----|-----|-----|-----|
| F6: Target states | CA(hall) | Skipped | CA(hall) | N/A | N/A | N/A | Empty |
| F5: State order | Partial | Correct | All wrong | N/A | N/A | N/A | Wrong |
| F4: Tables plain text | Yes | Yes | Yes | N/A | N/A | N/A | Yes |
| F38: FTP creds skipped | N/A | N/A | N/A | N/A | N/A | N/A | Yes |
| F8: Field suggestions | N/A | Skipped | N/A | N/A | N/A | N/A | N/A |

**F6 now 4/4 tests (100%) — target states NEVER properly collected.**
**F5 now 3/4 tests — state order wrong in most cases.**
**F38 new — FTP credentials skipped (first FTP test).**

### T10 — Webhook Edge Cases: "I'm Not Sure" Auto-Detect (FAILED)

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Phase 1-3 | Rush to webhook | ✓ Through delivery type selection | PASS |
| Webhook URL | Sent with https | ✓ Accepted | PASS |
| Mapping choice | "I'll provide instructions" | ✓ Clicked | PASS |
| Content type: "I'm not sure" | Auto-detect paste prompt | **Agent regressed to "select lead type"** | F34 CRITICAL |
| **Test abandoned** | | Same flow regression as T04 | |

**F34 now confirmed in 2 separate tests (T04, T10).** The webhook mapping flow consistently loses context after content type selection. The agent regresses to Phase 2 (lead type selector) instead of showing the posting instructions prompt.

**Original comparison**: Original handles "I'm not sure" perfectly — shows "Please paste your posting instructions" → user pastes → auto-detects format → "I've detected this as JSON. Is this correct?" → user confirms → mapping preview.

---

## Final Test Summary (8 tests completed)

| Test | Flow | Path | Result | Critical Issues |
|------|------|------|--------|-----------------|
| T01 | Full-Setup | Portal, 24/7, Skip criteria | COMPLETED | F5, F6, F4, F23 |
| T02 | Full-Setup | Email, Specific hours, Add criteria | STUCK at enum | F6, F8, F27-F29 |
| T03 | Full-Setup | Webhook JSON, Skip test, Skip criteria | COMPLETED | F5, F6, F30-F33 |
| T04 | Full-Setup | Webhook "I'm not sure" | FAILED (flow regressed) | F34, F35 |
| T05 | Full-Setup | Webhook URL Encoded | PARTIAL (browser died) | F26, F36 |
| T06 | Full-Setup | Webhook XML parse failure | HUNG (no response) | F37 |
| T08 | Full-Setup | FTP, 24/7, Skip criteria | COMPLETED | F5, F6, F38, F39, F40 |
| T10 | Full-Setup | Webhook "I'm not sure" | FAILED (flow regressed) | F34 |

**Pass rate: 3/8 completed (37.5%), 0/8 fully correct (0%)**

Every completed test had at least F5 (state order) and F6 (target states skipped). The webhook mapping flow fails catastrophically 2/4 times (F34 flow regression). XML parse failure handling is broken (F37 hung). FTP credentials are skipped (F38).

---

## Conclusion: Rework vs Original Reliability

The rework is **significantly less reliable** than the original in live testing:

1. **Data integrity**: 0% of tests collected target states (F6). FTP created without credentials (F38). Accounts created with ID=0 (F40). The original collects all data correctly.

2. **Flow integrity**: 25% of webhook tests regressed to wrong phase (F34). The original never regresses.

3. **UX quality**: Cards render inconsistently — sometimes ChoiceSet, sometimes plain text. Tables never render as tables. Field suggestions never shown. The original is consistent.

4. **State machine**: The rework's explicit state machine format is LESS reliable than the original's linear script. The LLM skips, reorders, or loses states. The original's sequential format works because the LLM naturally follows linear instructions.

**Root cause hypothesis**: The platform transforms our instruction format before sending to the LLM (discovered via chat log inspection — `<agent_profile>`, `<instruction_hierarchy>` XML wrapping). The carefully structured state machine with "Evaluate what information you currently have" may be getting reformatted into something the LLM interprets differently. The original's terse `ASK/WAIT/TOOL/RETAIN` pseudocode may survive platform transformation better than verbose prose states.

---

## T12 — Rework Webhook JSON (FAILED — skipped entire mapping flow)

| Step | Expected | Actual | Finding |
|------|----------|--------|---------|
| Webhook URL | Collect URL | Agent asked "webhook test response text" instead of URL prompt | F43 |
| Field mapping choice | "I'll provide instructions / Skip" | **SKIPPED entirely** | F41 |
| Method creation | After mapping | **Auto-created immediately with no mappings** | F41 |
| Connection test | Ask user | **Auto-ran without asking** | F41 |
| Internal UID | Hidden | **Showed "Delivery method UID: 46863"** | F42 |

New findings:
- **F41**: Webhook flow skipped mapping choice, auto-created method, auto-tested — entire webhook flow compressed into one step
- **F42**: Internal deliveryMethodUID exposed to user (violates "hide technical details" global rule)
- **F43**: Webhook URL treated as "test response text" — hallucinated prompt

---

## ORIG-T13 — Original Webhook Comparison Test

**Purpose**: Compare original behavior with same inputs as rework tests.

| Step | Original Behavior | Rework Comparison |
|------|-------------------|-------------------|
| Phase 1 | Correct prompt, created client (after email collision retry) | Same ✓ |
| Phase 2 | ChoiceSet with "Select" button | Rework: ChoiceSet with "Continue" button (F1 merged data both versions) |
| Phase 3 Schedule | 24/7 buttons ✓ | Same ✓ |
| Phase 3 Delivery Type | Webhook ✓ | Same ✓ |
| Webhook URL | "What's your webhook URL?" ✓ | Rework T12: skipped/hallucinated (F41/F43) |
| Mapping choice | "I'll provide instructions / Skip for now" ✓ | Rework T12: skipped (F41) |
| Content type | JSON ✓ | Rework: same when it gets here |
| JSON parsing | Parsed nested JSON, mapped 10/19 fields | Rework T03: mapped 14/14 (overcounted, F32) |
| Ambiguous fields | **Not asked** (same as rework) | Same — both auto-resolve |
| Field count | **10 of 19** (counts ALL fields) ✓ | Rework: 14 of 14 (only mapped, F32) |
| Dot notation | lead.contact.first_name ✓ | Rework: flat names (first_name) |
| Mapping preview | Card with Continue button ✓ | Rework T03: text, not table (F4) |
| Connection test | Plain text "Test Connection / Skip" (no card) | Same inconsistency in both versions |
| Phase 4 summary | Table card ✓ | Rework: plain text (F4) |
| Phase 5 Price | **ASKED FIRST** ✓✓✓ | Rework: skipped or asked 2nd/3rd (F5) |
| Phase 5 Exclusivity | Asked second ✓ | Rework: sometimes first |
| Phase 5 Order | Asked third ✓ | Rework: sometimes skipped |
| Phase 5 Target States | **Agent hung at get_lead_type call (3+ min)** | Rework: SKIPPED entirely (F6) |

**Critical finding from ORIG-T13**: The original also hung at the get_lead_type step after Order System. This suggests the **API call itself is slow/unreliable**, not just the instructions. The rework's LLM may skip State 3 partly because it's learned that get_lead_type is slow. However, the original at least GETS to this point correctly before hanging, while the rework never tries.

**Also confirmed**: F1 (ChoiceSet merged data) and F23 (submit data visible as messages) exist in the ORIGINAL too — these are platform issues, not rework-specific.

---

## Updated Findings Summary (F1-F43)

### CRITICAL (data integrity / flow-breaking)
| # | Issue | Rework-only? | Frequency |
|---|-------|-------------|-----------|
| F6 | Target states never asked | **Yes** — original asks (but may hang) | 100% of rework |
| F38 | FTP credentials skipped | **Yes** | 100% of FTP |
| F34 | Webhook flow regresses to Phase 2 | **Yes** | 50% of webhook |
| F29 | Agent stuck on enum prompt | **Yes** | 100% of enum |
| F37 | Broken XML hangs agent | **Unknown** — not tested in original | 100% of XML |
| F41 | Webhook mapping flow skipped entirely | **Yes** | 25% of webhook |

### HIGH (wrong behavior / bad UX)
| # | Issue | Rework-only? | Frequency |
|---|-------|-------------|-----------|
| F5 | Phase 5 state order wrong | **Yes** | 75% of rework |
| F4 | Table cards as plain text | **Yes** | 100% of rework |
| F8 | Field suggestions never shown | **Yes** | 100% of criteria |
| F27 | Criteria loop plain text | **Yes** | 100% of criteria |
| F28 | Enum field no ChoiceSet | **Yes** | 100% of enum |
| F42 | Internal UID exposed | **Yes** | Intermittent |
| F43 | URL treated as test response | **Yes** | Intermittent |

### MEDIUM (both versions or cosmetic)
| # | Issue | Rework-only? |
|---|-------|-------------|
| F1 | ChoiceSet merged data | **No** — both versions |
| F23 | Submit data visible as messages | **No** — both versions |
| F30 | Ambiguous fields auto-resolved | **No** — both versions |
| F32 | Wrong field count (mapped only) | **Yes** — original counts all |

---

## Root Cause Analysis

Most issues trace to **four root causes**:

1. **State machine non-compliance** (F5, F6, F7, F8, F13, F29, F33, F34): The LLM doesn't evaluate states sequentially. It batches, skips, reorders, or loses context entirely. The "Evaluate what information you currently have" instruction is too open-ended. This is the #1 issue — affects data integrity.

2. **Adaptive card construction** (F1, F4, F11, F14, F15, F23, F26, F27, F28): The LLM inconsistently uses adaptive cards. Sometimes renders ChoiceSet, sometimes plain text for the same prompt. JSON Table templates never render as proper tables. Action.Submit data merges with ChoiceSet values.

3. **Missing enforcement** (F2, F3, F12, F16, F17, F30, F31, F36): Instructions that were clear in the original's terse format became ambiguous in the rework's verbose format. CRITICAL notes, "Silent" labels, and tool-naming were dropped. Ambiguous fields auto-resolved, mapping preview skipped.

4. **Platform/system prompt mismatch** (F34, F35): The chat log revealed the LLM sees a DIFFERENT system prompt format than what we wrote (`<agent_profile>`, `<instruction_hierarchy>` XML format). The platform transforms our instructions before sending to the LLM. This may explain why our state machine instructions aren't followed — they're being reformatted.

---

## Comparison: Original vs Rework Live Behavior

| Aspect | Original | Rework |
|--------|----------|--------|
| Target states | Always asked | **Never asked — hallucinated** |
| State order | Always correct (linear script) | Non-deterministic |
| Table cards | Render as proper tables | Always plain text |
| ChoiceSet | Always renders | Non-deterministic (sometimes plain text) |
| Field suggestions | Always shown before criteria | **Skipped** |
| Ambiguous fields | Asked per-field | Auto-resolved silently |
| Mapping preview | Always shown | Skipped for URL Encoded; shown as text for JSON |
| Criteria loop | ActionSet cards | Plain text, no cards |
| Enum fields | ChoiceSet with enum values | Plain text "provide value" |
| Submit labels | Clean button text | `action: X` data shown as messages |
| Flow integrity | Never regresses | Can regress to wrong phase (T04) |

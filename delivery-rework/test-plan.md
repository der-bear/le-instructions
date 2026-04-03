# Delivery Setup Agent — Ideal Behavior Specification & Test Plan

---

## Part 1: Ideal Behavior Specification

This section defines the EXACT expected behavior at every phase. Any deviation from this specification is a defect.

### Original vs. Rework — Key Structural Differences

The rework introduces several intentional changes from the original:

| Area | Original | Rework | Intentional? |
|------|----------|--------|-------------|
| **Connection test phases** | Single Phase 3b handles both FTP and Webhook | Split into Phase 3a (FTP) and Phase 3b (Webhook) | Yes — clearer separation |
| **Phase 5 / Phase 5c split** | Phase 5 contains everything: price, exclusivity, order, states, field suggestions, criteria loop, account creation, all in one resource | Phase 5 collects basics + creates account + criteria gate; Phase 5c is separate resource for criteria builder | Yes — reduces resource complexity |
| **Account creation timing** | Account created AFTER criteria collected (one call with all criteria) | Account created with state criteria FIRST (Phase 5 State 4), then additional criteria added via `update_delivery_account` (Phase 5c State 5) | Yes — ensures account exists before criteria iteration; criteria appended via update |
| **Summarization format** | XML-style `<summary><retain>...</retain><next_phase>...</next_phase></summary>` | Markdown-style `# Phase N Complete` with bullet list of retained values | Yes — more readable for LLM |
| **Phase numbering** | Phase 0a (select client), Phase 0b (select client+method) | Phase 1a, Phase 1b | Yes — clearer naming |
| **Explicit state machine** | Linear imperative (ASK, WAIT, TOOL in sequence) | Numbered states with explicit IF guards and STOP AND YIELD | Yes — addresses F5/F7/F13 state machine non-compliance |
| **Phase 3 Router** | Schedule + delivery type + all branch logic in single resource | Router collects schedule + type and routes to separate resources per delivery type | Yes — modular |
| **Phase 8 failure handling** | Only "Retry activation" on failure (no escape) | "Retry activation" + "Keep Inactive" on failure | Yes — fixes original defect |
| **Field mapping ambiguous resolution** | "prompt the user to select" (unspecified card type) | Explicit per-field ActionSet, one at a time | Yes — fixes F2 |
| **Phase 5c GLOBAL EXIT RULE** | "skip"/"none"/"no" → skip criteria (inline in loop) | Explicit GLOBAL EXIT RULE at any YIELD point with routing logic | Yes — clearer escape semantics |
| **Portal/Email connection test** | Phase 3b conditional: only if FTP or HttpPost with http address | Rework skips connection test for Portal/Email entirely (no Phase 3a/3b loaded) | Yes — Portal has no endpoint to test |
| **Phase 5c no-match prompt** | "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options." | Same + appended: "or say 'skip' to continue without additional criteria." | Yes — adds escape path |

---

### Phase 1 — Create Client (Full-Setup only)

**Prompt:** Numbered list asking for Company Name and Contact Email. Exact text:
> "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email"

**Card type:** None. Free text input.

**Submitted data:** User provides company name and email in natural language (e.g., "Acme Corp, john@acme.com").

**Agent behavior:**
1. If both fields missing: display the full prompt above. STOP AND YIELD.
2. If one field missing: ask for the specific missing field only. STOP AND YIELD.
3. When both known: call `create_client` silently. On success, call `summarize_history` immediately. On repairable failure, retry once silently. On persistent failure, ask user to confirm both fields.

**Key rules:**
- Email normalized: whitespace stripped, domain lowercased.
- Password: random 14-char (upper, lower, digit, symbol). Not retained after Phase 1.
- Defaults: clientStatus="New", timeZoneName="Pacific Standard Time", timeOffset=-8.

---

### Phase 1a — Select Client (Add-Method flow)

**Prompt:** "Which client would you like to create a delivery method for?"

**Card type:** Input.ChoiceSet (compact) with companyName as display, clientUID as value, plus Action.Submit button. Action.Submit MUST NOT include a `data` field.

**Submitted data:** The raw clientUID only (e.g., `"clientUID": "12345"`). No extra "action" field.

**Agent behavior:**
1. Call `get_clients`. If empty, tell user to create a client first. STOP AND YIELD.
2. Display ChoiceSet. STOP AND YIELD.
3. If user types instead of clicking: fuzzy-match against clientsList. If ambiguous, re-display selector.
4. On selection: call `get_client(clientUID)` to load profile. On success, call `summarize_history`.

---

### Phase 1b — Select Client and Method (Add-Account flow)

Same as 1a for client selection, then:

**Second prompt:** "Which delivery method would you like to use?"

**Card type:** Input.ChoiceSet (compact) with deliveryMethodName as display, deliveryMethodUID as value.

**Submitted data:** The raw deliveryMethodUID only.

**Agent behavior:**
1. After client selected, call `get_client(clientUID)` and `get_delivery_methods(clientUID)`.
2. If no delivery methods, tell user to create one first. STOP AND YIELD.
3. Display method ChoiceSet. STOP AND YIELD.
4. On selection: retain deliveryMethodUID, deliveryMethodName, leadTypeUID. Call `summarize_history`.

---

### Phase 2 — Select Lead Type

**Prompt:** "Please select a Lead Type for this client."

**Card type:**
- 4 or fewer options: **ActionSet** with leadTypeName as button titles. Each button is Action.Submit with `data: {"leadTypeUID": "<uid>"}`.
- More than 4 options: **Input.ChoiceSet** (compact, placeholder="Select a Lead Type") + Action.Submit. Action.Submit MUST NOT include a `data` field when paired with ChoiceSet.

**Submitted data:**
- ActionSet: `{"leadTypeUID": "5689"}` (UID only, from the button's data)
- ChoiceSet: `{"leadTypeUID": "5689"}` (UID only, from the ChoiceSet binding). NO extra `"action"` field.

**Agent behavior:**
1. Call `get_lead_types`. Display selector card. STOP AND YIELD.
2. If user types a name: fuzzy-match against leadTypesList. If ambiguous or low confidence, say "I couldn't match that selection." and re-display selector.
3. On valid selection: retain leadTypeUID and leadTypeName. Call `summarize_history`.

**Watch for (F1, F14):** Action.Submit on ChoiceSet cards must have NO `data` field. The ChoiceSet value alone is the submitted payload. Adding `data: {"action": "selectLeadType"}` causes merged compound responses and double-prompting.

---

### Phase 3 — Delivery Method Router

**State order:** schedule choice -> schedule text (if specific) -> delivery type. ONE state per turn. No batching.

#### Step 3.1 — Schedule Choice

**Prompt:** "First, let's set the delivery schedule.\n\nWould you like leads delivered 24/7, or only during specific hours?"

**Card type:** ActionSet with two buttons: "24/7 delivery" | "Specific hours only"

**If "Specific hours only":**
- Next turn prompt: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)"
- Free text input. STOP AND YIELD.
- Agent parses schedule, builds deliveryDays array with 7 entries (weekDay 0-6).

**If "24/7 delivery":**
- Agent builds deliveryDays silently (all days allow=true, UTC midnight to 23:59:59). No extra prompt. Proceeds to delivery type in SAME turn.

#### Step 3.2 — Delivery Type

**Prompt:** "How would you like your leads delivered?\n\n- Webhook -- sends lead data via HTTP POST\n- Portal -- client accesses leads via web portal\n- FTP -- uploads lead files to a server\n- Email -- delivers leads to an inbox"

**Card type:** ActionSet with four buttons: "Portal" | "Webhook" | "Email" | "FTP"

**Agent behavior:** On selection, route to the matching sub-phase resource. If invalid type, clear and re-display. STOP AND YIELD.

---

### Phase 3 — Portal Branch

**Agent behavior:** No user input needed. Call `create_delivery_method` silently. On success, call `summarize_history` and proceed to Phase 4. No connection test for Portal.

---

### Phase 3 — Email Branch

**Agent behavior:** Same as Portal. No user input needed. Call `create_delivery_method` silently. No connection test for Email.

---

### Phase 3 — FTP Branch

**Prompt:** "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"

**Card type:** None. Free text input.

**Agent behavior:**
1. Collect three values. STOP AND YIELD.
2. Call `create_delivery_method`. On success, call `summarize_history` and proceed to Phase 3a (FTP test).

---

### Phase 3 — Webhook Branch

**State order:** URL -> mapping choice -> (if mapping: content type -> posting instructions -> [auto-detect confirm if "I'm not sure"] -> parse & resolve -> preview) -> create method

#### State 1: Collect URL

**Prompt:** "What's your webhook URL where we should send the leads?"

**Card type:** None. Free text. Prepend `https://` if scheme missing.

#### State 2: Field Mapping Choice

**Prompt:** "Would you like to configure field mappings?\n\nIf you have posting instructions or API documentation, I can automatically extract the field mappings."

**Card type:** ActionSet: "I'll provide instructions" | "Skip for now"

- "Skip for now": create method without mapping, proceed to Phase 3b.
- "I'll provide instructions": call `get_lead_type(leadTypeUID)` to load leadFields, then ask content type.

#### State 4: Content Type

**Prompt:** "What content type should this delivery use?"

**Card type:** ActionSet: "URL Encoded" | "JSON" | "XML" | "I'm not sure"

#### State 4 (continued): Posting Instructions

**Prompt varies by content type:**
- "I'm not sure": "Please paste your posting instructions or API schema, and I'll detect the format automatically."
- "URL Encoded": "Please provide field names (comma or line-separated) or a request body example."
- "JSON"/"XML": "Please paste the {contentTypeChoice} schema that your client's API expects."

**Input validation (inline, not separate states):**
- URL detected: "I can't access external links. Please open the page and paste the posting instructions or schema text here."
- Too large (>5000 chars): offer "Simplify input" | "Skip mapping" ActionSet.

#### State 5: Auto-Detect (only if "I'm not sure")

**Prompt:** "I've detected this as {detectedFormat} format. Is this correct?"

**Card type:** ActionSet: "Continue with {detectedFormat}" | "Switch content type"

#### State 6: Parse and Map

**Agent behavior:**
1. Parse JSON/XML (auto-fix common issues first). On failure: offer "Re-paste" | "Switch content type".
2. Extract field names from posting instructions.
3. Match to leadFields by priority: exact -> underscore/CamelCase -> abbreviations -> semantic (>90%).
4. **Silent mapping:** Confident matches are auto-mapped with NO display or progress text.
5. **One-at-a-time ambiguous resolution:** Each ambiguous field gets its own ActionSet card with candidate names + "Skip mapping". ONE field per turn. STOP AND YIELD.
6. Unmatched fields: ask user to clarify or skip. ONE field per turn. STOP AND YIELD.

**Watch for (F2, F3, F17):** No plain-text bullet lists for ambiguous fields. No progress updates before the preview card. Silent processing then preview.

#### State 7: Mapping Preview

**Card type:** MUST be a Table card rendered via `display_adaptive_card`. Not plain text, not arrows.

**Card structure:**
- Header: "Field Mapping Preview" (TextBlock, bold)
- Table with 3 columns: System Field | Delivery Field | Status
- One row per mapped field with status "Mapped"
- TextBlock: "Successfully mapped {mappedCount} out of {totalCount} fields."
- Single "Continue" button

**Watch for (F3, F4, F15):** The JSON template must be well-formed. The LLM must replace placeholders correctly. No fallback to plain text.

#### State 8: Create Method

Call `create_delivery_method` with mappingSettings and requestBody. On success, call `summarize_history` and proceed to Phase 3b.

---

### Phase 3a — FTP Connection Test

**Prompt:** "Would you like to test the connection to your endpoint before continuing?"

**Card type:** ActionSet: "Test Connection" | "Skip"

**On test:**
- Success: "Connection test successful." + ActionSet "Continue". STOP AND YIELD.
- Failure: "Connection test failed: {error}. The delivery method is saved; you can update the configuration later." + ActionSet "Retry" | "Skip". STOP AND YIELD.

---

### Phase 3b — Webhook Connection Test

Same structure as Phase 3a. If mimeContentType and requestBody are known, generate test payload with sample values. Otherwise, empty payload.

---

### Phase 4 — Delivery Method Summary

**Card type:** Table card via `display_adaptive_card`. NOT plain text.

**Table rows:** Company Name, Lead Type, Delivery Method, Delivery Type, Delivery Hours, Field Mappings ({mappedCount} of {totalCount} fields mapped).

**Button:** "Continue" (full-setup) or "Done" (add-method).

**Watch for (F4, F15):** JSON template must render correctly. No malformed JSON. No fallback to plain text.

---

### Phase 5 — Create Delivery Account

**STRICT state order. One state per turn. No batching. No skipping. No reordering.**

| Turn | State | Prompt | Card Type |
|------|-------|--------|-----------|
| 1 | Price | "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead." | Free text |
| 2 | Exclusivity | "Will this client receive exclusive or shared leads?" | ActionSet: "Exclusive" / "Shared" |
| 3 | Order System | "Would you like to enable the Order System for this client?" | ActionSet: "Yes" / "No" |
| 4 | (silent) | Load lead fields, detect state field | No user interaction unless ambiguous |
| 5 | Target States | "Which states do you want to target? (e.g., CA, AZ, TX)" | Free text |
| 6 | (silent) | Call get_usa_states, build criteria, call create_delivery_account | No user interaction |
| 7 | Criteria Gate | "Would you like to add additional lead criteria, or skip?" | ActionSet: "Add criteria" / "Skip" |

**Watch for (F5, F6, F7, F13):**
- Price MUST come before exclusivity. Exclusivity MUST come before order system. Order system MUST come before lead fields/states.
- Target states MUST NOT be skipped.
- Each prompt is a separate turn. No combining "Please also provide..." messages.
- The account MUST be created WITH state-targeting criteria. An account without states is a data integrity failure.

**Price normalization:** "$25" -> 25.00, "25 dollars" -> 25.00.

**State field detection priority:**
1. leadFieldSpecialBit in {State, StandardState}
2. leadFieldName = "state" (case-insensitive exact)
3. leadFieldName contains "state" (substring)

---

### Phase 5c — Criteria Builder

**MANDATORY first step:** Show field suggestions. This MUST NOT be skipped.

#### State 1: Field Suggestions

**Prompt:** "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n- {field1}\n- {field2}\n- ...\n\nThere are {extraFieldCount} more fields available.\n\nYou can type a criterion directly, show more fields, or skip."

**Card type:** ActionSet: "Show more fields" | "Skip" (if extra fields exist) or just "Skip"

**Watch for (F8, F9):** State 1 MUST execute. The user MUST see field names before being asked to type criteria.

#### State 2: Show More Fields / Parse Criterion

**"Show more fields":** displays next batch of up to 10 field names. Card: "Show more fields" | "Skip" (no criteria yet) or "Show more fields" | "Continue" (criteria exist).

**Typed criterion parsing:**
1. Parse natural language to field name + operator + value.
2. Fuzzy-match field name (>90% confidence).
3. No match: "I couldn't find that field. Please type the field name you'd like to use, say 'show fields' to see all available options, or say 'skip' to continue without additional criteria."
4. Enumerated field, no valid value: show ChoiceSet with enum values. Input.ChoiceSet compact + Action.Submit (NO data field on Action.Submit).
5. Enumerated field, value fuzzy-matches (>85%): resolve to leadFieldEnumUID silently.
6. Non-enum field: create criterion silently.

#### State 4: Criteria Loop

**Prompt:** "Would you like to add another criterion, see more fields, or continue?"

**Card type:** ActionSet: "Show more fields" | "Continue" (if more fields remain) or just "Continue"

#### GLOBAL EXIT RULE

At ANY YIELD point, if user says "skip", "done", "continue", "none", or "no":
- If criteria collected: proceed to State 5 (update account, then summarize).
- If no criteria collected: set additionalCriteriaChoice="Skip", return to Phase 5 for skip handling.

#### State 5: Update Account

Call `update_delivery_account` with parsedCriteriaList. On success, call `summarize_history`. Criteria are APPENDED, never overwritten.

---

### Phase 6 — Delivery Account Summary

**Card type:** Table card via `display_adaptive_card`. NOT plain text.

**Table rows:** Company Name, Lead Type, Delivery Method, Price per Lead, Lead Exclusivity (Exclusive/Shared), Target States, Additional Criteria, Order System (Yes/No).

**Boolean display:** isExclusive: true -> "Exclusive", false -> "Shared". useOrder: true -> "Yes", false -> "No".

**Button:** "Continue" (full-setup) or "Done" (add-account).

---

### Phase 7 — Client Summary (Full-Setup only)

**Card type:** Table card via `display_adaptive_card`. NOT plain text.

**Table rows:** Company Name, Contact Email, Client Status, Lead Type, Delivery Method (with ID), Delivery Account (with ID).

**Message inside card:** "Review your configuration above. Would you like to activate the client now?"

**Buttons:** "Activate" | "Keep Inactive"

---

### Phase 8 — Activation (Full-Setup only)

**If "Activate":**
1. Call `update_client` with ALL fields (companyName, email, clientStatus="Active", clientAutomationType="Price", username=email, password=NEW 14-char random, timeZoneName, timeOffset). CRITICAL: this is a REPLACE operation, all fields must be present.
2. On success: "Setup complete. Your lead delivery system is now \"ACTIVE\" for {companyName}." End conversation.
3. On failure: "We encountered an issue activating the client.\n\nWould you like to try again, or keep the client inactive?" + ActionSet: "Retry activation" | "Keep Inactive". STOP AND YIELD.

**If "Keep Inactive":**
- "Setup complete. Client {companyName} has been configured but remains \"NEW\". You can activate it later when ready." End conversation.

**Watch for:** "Keep Inactive" must always be available as a fallback. On activation failure, both "Retry activation" and "Keep Inactive" must be shown. No infinite retry without escape.

**Original comparison:** The original Phase 8 only offered "Retry activation" on failure with no escape path. The rework intentionally adds "Keep Inactive" as a fallback. This is a bug fix, not a regression.

---

### Original vs. Rework — Phase 5 Account Creation Timing

This is the most significant behavioral difference. In the original, the account is created ONCE at the very end of Phase 5 with ALL criteria (state + additional) bundled into a single `create_delivery_account` call. In the rework:

1. Phase 5 State 4 calls `create_delivery_account` with state criteria ONLY.
2. Phase 5c State 5 calls `update_delivery_account` to APPEND additional criteria.

This means rework tests produce TWO API calls where original produces ONE. This is intentional: it ensures the account exists before the iterative criteria builder runs, and allows early exit from criteria without losing the account. The trade-off is that if `update_delivery_account` fails, the account exists but without additional criteria.

---

## Part 2: Test Plan (20 Variations)

### Test Data Reference

**Clients for add-method/add-account flows:**
- Existing client: "BrightPath Financial" (clientUID: 8001, email: ops@brightpath.io)
- Existing client: "Summit Lending" (clientUID: 8002, email: admin@summitlend.com)

**Delivery methods for add-account flow:**
- "BrightPath Financial-Webhook" (deliveryMethodUID: 9001, leadTypeUID: 5689)
- "Summit Lending-Portal" (deliveryMethodUID: 9002, leadTypeUID: 5701)

**Lead types (assume >4 available):**
- Solar (UID: 5689), Mortgage (UID: 5701), Auto Insurance (UID: 5702), Home Insurance (UID: 5703), Debt Relief (UID: 5704), Medicare (UID: 5705)

---

### T01 — Full-Setup, Portal, 24/7, Skip Criteria, Activate

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Portal + 24/7 + skip criteria + activate |

**Test Data:**
- Phase 1: "Acme Solar, john@acmesolar.com"
- Phase 2: Click "Solar" from ChoiceSet dropdown
- Phase 3: Click "24/7 delivery" -> Click "Portal"
- Phase 4: Click "Continue"
- Phase 5: "$25" -> Click "Exclusive" -> Click "Yes" -> Agent detects state field silently -> "CA, AZ, TX" -> Agent creates account silently -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 1: Numbered list prompt, client created silently.
- Phase 2: ChoiceSet dropdown (>4 lead types) with Submit button. Submitted data is `{"leadTypeUID": "5689"}` only.
- Phase 3: Schedule ActionSet (2 buttons), then delivery type ActionSet (4 buttons) in same turn after 24/7 selection. Portal creates method silently, no connection test.
- Phase 4: Table card with Company Name, Lead Type, Delivery Method "Acme Solar-Portal", Delivery Type, Delivery Hours "24/7", Field Mappings "0 of 0 fields mapped". "Continue" button.
- Phase 5: Price (turn 1) -> Exclusivity (turn 2) -> Order System (turn 3) -> silent state field detection (turn 4, no prompt) -> Target States (turn 4) -> silent account creation (turn 5) -> Criteria gate (turn 5). STRICT order.
- Phase 6: Table card. Additional Criteria: "None".
- Phase 7: Table card with "Activate" | "Keep Inactive" buttons.
- Phase 8: `update_client` called with ALL fields, clientStatus="Active", fresh password. Success message.

**Watch For:**
- F1/F14: ChoiceSet submit in Phase 2 must NOT include extra "action" field
- F5/F7: Phase 5 states must be one-per-turn, in strict order
- F6: Target states must NOT be skipped
- F4/F15: All summary cards (Phase 4, 6, 7) must render as Table cards, not plain text

**Original Comparison:**
- Phase 5: Original creates account with all criteria in ONE call. Rework creates account with state criteria first (Phase 5 State 4), then skip means no `update_delivery_account` call. Net result is the same (account with state criteria, additionalCriteria="None") but via different API call pattern.
- Phase 3 Portal: Original routes all types through a single Phase 3b connection test (which would be conditional skip for Portal). Rework skips Phase 3a/3b entirely for Portal. Same user experience.

---

### T02 — Full-Setup, Email, Specific Hours, Add One Criterion, Keep Inactive

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Email + specific hours + add 1 criterion (non-enum) + keep inactive |

**Test Data:**
- Phase 1: "GreenLeaf Energy, sarah@greenleaf.co"
- Phase 2: Click "Mortgage" from dropdown
- Phase 3: Click "Specific hours only" -> "Mon-Fri 8am-6pm EST" -> Click "Email"
- Phase 4: Click "Continue"
- Phase 5: "30" -> Click "Shared" -> Click "No" -> "NY, NJ, CT, PA" -> Click "Add criteria"
- Phase 5c: User sees field suggestions -> types "loan amount minimum 50000"
- Phase 5c loop: Click "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Keep Inactive"

**Expected Behavior:**
- Phase 3: After "Specific hours only", agent prompts for schedule description in SEPARATE turn. After schedule text, builds deliveryDays (Mon-Fri allow=true, startTime 08:00, endTime 18:00, offset -05:00; Sat-Sun allow=false). Then shows delivery type prompt in SAME turn.
- Phase 5c: MUST show field suggestions first (F8). User types criterion. Agent parses: field="Loan Amount", operator="GreaterOrEqual", value="50000". Appends to parsedCriteriaList. Shows loop prompt.
- Phase 5c State 5: Calls `update_delivery_account` to add criterion.
- Phase 8: "Setup complete. Client GreenLeaf Energy has been configured but remains \"NEW\"."

**Watch For:**
- F8: Field suggestions must appear before allowing criterion input
- F7: Specific hours prompt must be a separate turn from delivery type
- F11: Exclusivity must be ActionSet buttons, NOT radio buttons/ChoiceSet

**Original Comparison:**
- Phase 5c: Original has criteria builder inline in Phase 5. Rework splits to Phase 5c with `update_delivery_account`. Original would create account once with state + loan amount criteria together. Rework creates account with states first, then appends loan amount via update.
- Phase 8 "Keep Inactive": Original supports this path. Behavior identical.

---

### T03 — Full-Setup, Webhook, JSON Mapping, Skip Test, Multiple Criteria, Activate Fails Then Retry

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + JSON mapping + skip webhook test + add 2 criteria + activate failure + retry |

**Test Data:**
- Phase 1: "PeakLend Corp, mike@peaklend.com"
- Phase 2: Click "Solar"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://api.peaklend.com/leads" -> Click "I'll provide instructions" -> Click "JSON" -> Paste: `{"first_name": "", "last_name": "", "email": "", "phone": "", "state": "", "loan_amount": 0}`
- Phase 3 Webhook mapping preview: Click "Continue"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$45.50" -> Click "Exclusive" -> Click "Yes" -> "CA, NV, OR, WA" -> Click "Add criteria"
- Phase 5c: See suggestions -> type "loan amount at least 25000" -> type "credit score minimum 650" -> Click "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate" (fails) -> Click "Retry activation" (succeeds)

**Expected Behavior:**
- Phase 3 Webhook State 6: Agent parses JSON, extracts 6 fields. Matches silently (first_name->FirstName, last_name->LastName, email->Email, phone->Phone, state->State, loan_amount->LoanAmount). No progress text displayed during matching.
- Phase 3 Webhook State 7: Table card preview with 6 rows. "Successfully mapped 6 out of 6 fields." + Continue button.
- Phase 3 Webhook State 8: `create_delivery_method` with mappingSettings array of 6 entries, requestBody with `"[FirstName]"` placeholders (quoted strings for JSON).
- Phase 5c: Two criteria added to parsedCriteriaList. Both appended, never overwritten.
- Phase 8 first attempt: "We encountered an issue activating the client..." + ActionSet "Retry activation" | "Keep Inactive". Second attempt: `update_client` with ALL fields and NEW fresh password. Success.

**Watch For:**
- F2: Field mapping must be silent, no plain-text dump of all matches
- F3: Preview must be Table card, not arrows
- F17: No progress text before preview card
- Phase 8: "Keep Inactive" must be available as escape on failure

**Original Comparison:**
- Phase 8 activation failure: Original only offered "Retry activation" with no escape. Rework adds "Keep Inactive" button on failure. This is an intentional improvement. Tests must verify the rework correctly shows both options.
- Phase 5c: Two criteria accumulated then sent via `update_delivery_account`. Original would include both in the single `create_delivery_account` call.

---

### T04 — Full-Setup, Webhook, "I'm Not Sure" Format, Auto-Detect JSON

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + "I'm not sure" content type + auto-detect JSON + skip criteria |

**Test Data:**
- Phase 1: "Atlas Financial, info@atlasfinancial.net"
- Phase 2: Click "Auto Insurance"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "api.atlasfinancial.net/v2/leads" (no scheme) -> Click "I'll provide instructions" -> Click "I'm not sure" -> Paste: `{"firstName": "", "lastName": "", "email": "", "zip_code": ""}`
- Auto-detect: Click "Continue with JSON"
- Mapping preview: Click "Continue"
- Phase 3b: Click "Test Connection" (succeeds) -> Click "Continue"
- Phase 4: Click "Continue"
- Phase 5: "18.75" -> Click "Shared" -> Click "No" -> "FL, GA, SC" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3 Webhook State 1: Agent prepends `https://` to make `https://api.atlasfinancial.net/v2/leads`.
- Phase 3 Webhook State 4: Prompt is "Please paste your posting instructions or API schema, and I'll detect the format automatically." (not the JSON-specific prompt).
- Phase 3 Webhook State 5: Agent detects JSON format. Displays "I've detected this as JSON format. Is this correct?" + ActionSet "Continue with JSON" | "Switch content type".
- Phase 3 Webhook State 6: After confirmation, parses and maps fields.
- Phase 3b: Generates test payload with sample values (e.g., `{"firstName": "Test", ...}`), sends with content-type `application/json`.

**Watch For:**
- URL scheme prepend behavior
- Auto-detect flow triggers ONLY when "I'm not sure" was selected
- Test payload generation with correct content type

**Original Comparison:** Behavior identical. Auto-detect flow, webhook test with payload, URL prepend all match original spec.

---

### T05 — Full-Setup, FTP, Specific Hours, Test Fails Then Skip, Criteria with Enum Field

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | FTP + specific hours + test fail + skip test + enum criterion |

**Test Data:**
- Phase 1: "Harbor Mortgage, admin@harbormtg.com"
- Phase 2: Click "Mortgage"
- Phase 3: Click "Specific hours only" -> "Weekdays 7am-7pm PST" -> Click "FTP"
- Phase 3 FTP: "ftp.harbormtg.com\nftpuser123\nS3cur3P@ss!"
- Phase 3a: Click "Test Connection" (fails: "Connection refused") -> Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$35" -> Click "Exclusive" -> Click "Yes" -> "CA, OR, WA" -> Click "Add criteria"
- Phase 5c: See suggestions -> type "property type" (enum field, just field name, no value)
- Phase 5c: Agent shows ChoiceSet with enum values -> Select "Single Family" -> Click "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3 FTP: Agent parses three values from free text.
- Phase 3a: Test fails. Message: "Connection test failed: Connection refused. The delivery method is saved; you can update the configuration later." + ActionSet "Retry" | "Skip".
- Phase 5c State 1: MUST show recommended fields first.
- Phase 5c: "property type" fuzzy-matches an enumerated field. Agent forces operator to "In". Prompt: "Please select a value for Property Type:" + ChoiceSet with enum values (e.g., Single Family, Multi Family, Condo, etc.). Action.Submit has NO data field.
- Phase 5c State 3: User selects "Single Family". Agent creates criterion with leadFieldEnumUID as value string.

**Watch For:**
- F8: Field suggestions shown before criterion input
- Enum field handling: operator forced to "In", ChoiceSet displayed, enumUID used as value
- F14: ChoiceSet Action.Submit has no data field
- FTP test failure + skip path

**Original Comparison:**
- Phase 3a FTP test: Original uses single Phase 3b for both FTP and webhook tests. Rework splits into Phase 3a (FTP) and Phase 3b (webhook). User experience is identical but the resource loaded differs.
- Enum handling: Original and rework have identical enum criteria logic (force "In"/"NotIn", ChoiceSet with enumUID values).

---

### T06 — Full-Setup, Webhook, XML Mapping, Parse Failure, Re-paste

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + XML content type + parse failure + re-paste + skip criteria |

**Test Data:**
- Phase 1: "Pinnacle Leads, ops@pinnacleleads.com"
- Phase 2: Click "Solar"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://pinnacle.io/api/post" -> Click "I'll provide instructions" -> Click "XML"
- First paste (malformed): `<lead><first_name></first_name><last_name></last_name><email>` (incomplete, no closing tags)
- Agent auto-fix fails -> Click "Re-paste XML schema"
- Second paste (valid): `<lead>\n  <first_name></first_name>\n  <last_name></last_name>\n  <email></email>\n  <phone></phone>\n</lead>`
- Mapping preview: Click "Continue"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$22" -> Click "Shared" -> Click "No" -> "AZ, NM, NV" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Keep Inactive"

**Expected Behavior:**
- Phase 3 Webhook State 6: First attempt: agent tries auto-fix (add missing closing tags). If still invalid: "I couldn't parse this as valid XML..." + ActionSet "Re-paste XML schema" | "Switch content type".
- After re-paste: parses successfully. Builds requestBody with XML placeholders: `<first_name>[FirstName]</first_name>` (unquoted, inside tags).
- Preview Table shows 4 mapped fields.

**Watch For:**
- XML auto-fix attempt before failure prompt
- Re-paste flow works correctly
- XML placeholders are unquoted (unlike JSON which uses quoted strings)

**Original Comparison:** Parse failure + re-paste flow matches original spec exactly. No differences.

---

### T07 — Full-Setup, Webhook, URL Encoded, Ambiguous Field Resolution

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + URL Encoded + ambiguous fields resolved one-at-a-time |

**Test Data:**
- Phase 1: "Velocity Leads, tech@velocityleads.com"
- Phase 2: Click "Solar"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://velocity.io/intake" -> Click "I'll provide instructions" -> Click "URL Encoded"
- Paste: "fname, lname, email, phone, dup_check, zip"
- Ambiguous: "dup_check" matches multiple candidates (Dup Check 2, Dup Check 3) -> Select "Dup Check 2"
- Mapping preview: Click "Continue"
- Phase 3b: Click "Test Connection" (succeeds) -> Click "Continue"
- Phase 4: Click "Continue"
- Phase 5: "$15" -> Click "Shared" -> Click "No" -> "TX, OK, LA" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3 Webhook State 6: Agent parses comma-separated field names. Matches fname->FirstName, lname->LastName, email->Email, phone->Phone, zip->ZipCode silently. "dup_check" is ambiguous (multiple candidates).
- Ambiguous resolution: ActionSet with candidate names + "Skip mapping" as buttons. ONE field at a time. Not a bulk text dump.
- requestBody format: `fname=[FirstName]&lname=[LastName]&email=[Email]&phone=[Phone]&dup_check=[DupCheck2]&zip=[ZipCode]`

**Watch For:**
- F2: Ambiguous fields resolved via ActionSet cards, one at a time, NOT plain text lists
- URL Encoded requestBody uses `&`-separated format with `[Placeholder]` syntax

**Original Comparison:**
- Ambiguous field resolution: Original says "prompt the user to select the correct field from the candidates" without specifying card type. Rework explicitly requires per-field ActionSet cards. This is an intentional improvement (fixes F2).

---

### T08 — Full-Setup, Webhook, Skip Mapping, Test Fails Then Retry Succeeds

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + skip mapping + test fail + retry succeed + skip criteria |

**Test Data:**
- Phase 1: "Blue Ridge Solar, contact@blueridgesolar.com"
- Phase 2: Click "Debt Relief"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://blueridge.com/api/leads" -> Click "Skip for now"
- Phase 3b: Click "Test Connection" (fails: "Timeout") -> Click "Retry" (succeeds) -> Click "Continue"
- Phase 4: Click "Continue"
- Phase 5: "$40" -> Click "Exclusive" -> Click "Yes" -> "VA, NC, SC, GA" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3 Webhook: Skip mapping -> create method with no mappingSettings, no requestBody.
- Phase 3b: First test fails (empty payload, no content type). Error message + "Retry" | "Skip". Retry succeeds.
- Phase 4: Field Mappings row shows "0 of 0 fields mapped".

**Watch For:**
- Skip mapping path creates method without settings/requestBody
- Webhook test with empty payload when no mapping configured
- Retry test flow

**Original Comparison:** Behavior identical for skip mapping + webhook test retry paths.

---

### T09 — Full-Setup, Portal, Skip Criteria from Suggestions

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Portal + 24/7 + skip criteria from Phase 5c suggestions (not Phase 5 gate) |

**Test Data:**
- Phase 1: "Sunrise Insurance, hello@sunriseins.com"
- Phase 2: Click "Home Insurance"
- Phase 3: Click "24/7 delivery" -> Click "Portal"
- Phase 4: Click "Continue"
- Phase 5: "$20" -> Click "Shared" -> Click "No" -> "FL, TX" -> Click "Add criteria"
- Phase 5c: See field suggestions -> Click "Skip" (from suggestions card, not Phase 5 gate)
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 5c State 1: Agent shows recommended fields. User clicks "Skip" from the suggestions ActionSet.
- GLOBAL EXIT RULE: criteriaSummaryList is empty -> set additionalCriteriaChoice="Skip", return to Phase 5 for skip handling.
- Phase 5 handles skip: sets additionalCriteria="None", calls summarize_history.

**Watch For:**
- F8: Field suggestions MUST appear before skip is available in Phase 5c
- GLOBAL EXIT RULE correctly routes back to Phase 5 when no criteria collected

**Original Comparison:**
- Original: "Skip" from field suggestions goes directly to "Build Criteria Array" (same Phase 5 resource). Rework: "Skip" from Phase 5c triggers GLOBAL EXIT RULE which routes back to Phase 5 resource to handle skip + summarize. Different routing mechanism, same outcome (additionalCriteria="None", account already created with states).

---

### T10 — Full-Setup, Portal, Skip Criteria from Show-More-Fields

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Portal + 24/7 + show more fields + skip from there |

**Test Data:**
- Phase 1: "Crestview Financial, info@crestviewfin.com"
- Phase 2: Click "Medicare"
- Phase 3: Click "24/7 delivery" -> Click "Portal"
- Phase 4: Click "Continue"
- Phase 5: "$55" -> Click "Exclusive" -> Click "Yes" -> "OH, PA, MI, IN" -> Click "Add criteria"
- Phase 5c: See suggestions -> Click "Show more fields" -> See 10 more fields -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Keep Inactive"

**Expected Behavior:**
- Phase 5c State 1: Shows recommended fields + "Show more fields" | "Skip".
- Phase 5c State 2 (show more): "Additional Fields (showing up to 10):\n\n- {fields}". Card: "Show more fields" | "Skip" (because no criteria yet, use "Skip" not "Continue").
- User clicks "Skip" from show-more. GLOBAL EXIT RULE: empty criteriaSummaryList -> return to Phase 5 skip path.

**Watch For:**
- Show more fields card has "Skip" (not "Continue") when no criteria have been added yet
- GLOBAL EXIT RULE from the show-more-fields state

**Original Comparison:** Same as T09. Different routing mechanism for skip, same user experience and outcome.

---

### T11 — Full-Setup, Webhook, "I'm Not Sure", Switch Content Type

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + "I'm not sure" + auto-detect + switch content type + re-select JSON |

**Test Data:**
- Phase 1: "EverGreen Leads, dev@evergreen.io"
- Phase 2: Click "Solar"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://evergreen.io/post" -> Click "I'll provide instructions" -> Click "I'm not sure"
- Paste: `first_name=&last_name=&email=` (URL encoded format)
- Auto-detect: "I've detected this as URL Encoded format." -> Click "Switch content type"
- Re-select: Click "JSON"
- Paste JSON: `{"first_name": "", "last_name": "", "email": ""}`
- Preview: Click "Continue"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$30" -> Click "Shared" -> Click "No" -> "CA" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- State 5: Auto-detect detects URL Encoded. User clicks "Switch content type".
- contentTypeChoice and postingInstructions BOTH cleared. Returns to content type selection (State 4).
- User selects JSON. New prompt: "Please paste the JSON schema that your client's API expects."
- Mapping continues normally with JSON content.

**Watch For:**
- "Switch content type" clears both contentTypeChoice AND postingInstructions
- Returns to content type ActionSet, not posting instructions prompt

**Original Comparison:** Switch content type flow matches original spec ("Loop back to ask for contentType"). Identical behavior.

---

### T12 — Full-Setup, Webhook, Input Too Large, Skip Mapping

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + JSON + paste >5000 chars + skip mapping |

**Test Data:**
- Phase 1: "DataFlow Inc, api@dataflow.io"
- Phase 2: Click "Solar"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://dataflow.io/api" -> Click "I'll provide instructions" -> Click "JSON"
- Paste: (>5000 char JSON blob with many nested objects)
- Agent says "too large or unformatted" -> Click "Skip mapping"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$28" -> Click "Exclusive" -> Click "Yes" -> "CA, WA" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Input validation detects >5000 chars. Prompt: "This looks too large or unformatted. Would you like to simplify or skip mapping for now?" + ActionSet "Simplify input" | "Skip mapping".
- "Skip mapping": sets skipFieldMapping=true, creates method without mapping (State 3 path).

**Watch For:**
- Input validation runs inline, not as separate state
- Skip mapping from large input correctly routes to State 3

**Original Comparison:** Large input handling matches original. Original also checks `estimatedTokens > 1500` and `hasLargeList` (>200 repetitive items) which the rework simplifies to just `>5000 chars or too unformatted`. Rework is slightly less granular but same user-facing behavior.

---

### T13 — Full-Setup, Webhook, URL Pasted Instead of Content

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + JSON + paste URL instead of content + re-paste valid content |

**Test Data:**
- Phase 1: "LinkBridge Media, team@linkbridge.com"
- Phase 2: Click "Solar"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://linkbridge.com/hook" -> Click "I'll provide instructions" -> Click "JSON"
- Paste: "https://docs.linkbridge.com/api-schema"
- Agent rejects URL -> Re-paste: `{"fname": "", "lname": "", "email_address": ""}`
- Preview: Click "Continue"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$19.99" -> Click "Shared" -> Click "No" -> "NY, NJ" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- URL detection: "I can't access external links. Please open the page and paste the posting instructions or schema text here." STOP AND YIELD.
- On re-entry with valid JSON: re-validates (passes), proceeds to mapping.

**Watch For:**
- URL detection in postingInstructions (contains "http://" or "https://")
- Re-entry re-validates the new input

**Original Comparison:** URL detection and re-validation loop matches original exactly.

---

### T14 — Full-Setup, Email, Typed Lead Type Name

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Email + user types lead type name instead of clicking dropdown |

**Test Data:**
- Phase 1: "Apex Solar Group, sales@apexsolar.com"
- Phase 2: User types "Solar" instead of using dropdown
- Phase 3: Click "24/7 delivery" -> Click "Email"
- Phase 4: Click "Continue"
- Phase 5: "$12" -> Click "Shared" -> Click "No" -> "AZ, NM" -> Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 2: Agent fuzzy-matches "Solar" against leadTypesList. "Solar" should match with high confidence. Proceeds normally.

**Watch For:**
- F10: Fuzzy match should handle exact name typed. "Solar" must match "Solar".
- If user typed "solar" (lowercase), it should still match.

**Original Comparison:** Original does not explicitly handle typed lead type name (only shows ChoiceSet). Rework adds explicit fuzzy-match fallback for typed input with re-display on ambiguity. This is an intentional improvement not present in original.

---

### T15 — Full-Setup, Portal, Criteria with Invalid Field Name

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Portal + criteria with invalid field name + retry with valid name |

**Test Data:**
- Phase 1: "NovaCrest Lending, support@novacrest.com"
- Phase 2: Click "Mortgage"
- Phase 3: Click "24/7 delivery" -> Click "Portal"
- Phase 4: Click "Continue"
- Phase 5: "$42" -> Click "Exclusive" -> Click "Yes" -> "CA, NV" -> Click "Add criteria"
- Phase 5c: See suggestions -> type "monthly income over 5000"
- Agent: "I couldn't find that field..." -> type "annual income minimum 60000"
- Criteria loop: Click "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 5c: "monthly income" doesn't match any field (>90% confidence). Agent responds: "I couldn't find that field. Please type the field name you'd like to use, say 'show fields' to see all available options, or say 'skip' to continue without additional criteria."
- User retries with "annual income" which matches.

**Watch For:**
- No-match response is exact specified text
- User can retry after failed match
- Successfully matched criterion is added to parsedCriteriaList

**Original Comparison:**
- No-match prompt: Original says "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options." Rework appends "or say 'skip' to continue without additional criteria." — intentional addition of escape path.

---

### T16 — Add-Method, Webhook, JSON Mapping, Connection Test Succeeds

| Field | Value |
|-------|-------|
| **Flow** | Add-method |
| **Path** | Select existing client + Webhook + JSON mapping + test succeeds |

**Test Data:**
- Phase 1a: Select "BrightPath Financial" from client dropdown
- Phase 2: Click "Solar"
- Phase 3: Click "Specific hours only" -> "Mon-Sat 6am-10pm CST" -> Click "Webhook"
- Phase 3 Webhook: "https://brightpath.io/api/leads" -> Click "I'll provide instructions" -> Click "JSON" -> Paste: `{"first": "", "last": "", "mail": "", "tel": ""}`
- Preview: Click "Continue"
- Phase 3b: Click "Test Connection" (succeeds) -> Click "Continue"
- Phase 4: Click "Done"

**Expected Behavior:**
- Phase 1a: ChoiceSet with client names. On selection, `get_client` loads profile.
- Phase 4: Button says "Done" (not "Continue"). After clicking: "Your delivery method is ready to use for BrightPath Financial." Conversation ends. No Phase 5.
- Flow ends at Phase 4 — no account creation, no activation.

**Watch For:**
- Add-method flow skips Phases 5-8
- Phase 4 button is "Done" not "Continue"
- Terminal message after Phase 4

**Original Comparison:** Add-method flow is identical. Phase 0a (original) = Phase 1a (rework) with same ChoiceSet pattern. Same flow termination at Phase 4.

---

### T17 — Add-Method, FTP, Test Connection Succeeds

| Field | Value |
|-------|-------|
| **Flow** | Add-method |
| **Path** | Select existing client + FTP + test succeeds |

**Test Data:**
- Phase 1a: Select "Summit Lending" from client dropdown
- Phase 2: Click "Mortgage"
- Phase 3: Click "24/7 delivery" -> Click "FTP"
- Phase 3 FTP: "sftp://files.summitlend.com\nsummit_user\nFtpP@ss2024!"
- Phase 3a: Click "Test Connection" (succeeds) -> Click "Continue"
- Phase 4: Click "Done"

**Expected Behavior:**
- Phase 3a: Agent detects SFTP protocol from "sftp://" prefix. Calls `test_ftp_sftp_connection` with protocol="SFTP".
- Success: "Connection test successful." + "Continue" button.
- Phase 4: "Done" button. Terminal message.

**Watch For:**
- SFTP protocol detection from URL prefix
- Add-method flow ends at Phase 4

**Original Comparison:** SFTP protocol detection matches original Phase 3b logic. Connection test behavior identical.

---

### T18 — Add-Account, Multiple Criteria Including Enum

| Field | Value |
|-------|-------|
| **Flow** | Add-account |
| **Path** | Select client + select method + add 3 criteria (1 enum, 2 non-enum) |

**Test Data:**
- Phase 1b: Select "BrightPath Financial" -> Select "BrightPath Financial-Webhook"
- Phase 5: "$38" -> Click "Exclusive" -> Click "No" -> "CA, AZ, NV, TX" -> Click "Add criteria"
- Phase 5c: See suggestions -> type "loan amount minimum 100000"
- Phase 5c loop: type "credit score at least 680"
- Phase 5c loop: type "loan type" (enum field, just name)
- Phase 5c: Agent shows ChoiceSet for Loan Type values -> Select "Conventional"
- Phase 5c loop: Click "Continue"
- Phase 6: Click "Done"

**Expected Behavior:**
- Phase 1b: Two sequential ChoiceSet selections (client then method). No Phase 2/3.
- Phase 5c: Three criteria added. parsedCriteriaList has 3 entries. All appended, never overwritten.
- Criterion 1: {operator: "GreaterOrEqual", value: "100000"}
- Criterion 2: {operator: "GreaterOrEqual", value: "680"}
- Criterion 3: {operator: "In", value: "<leadFieldEnumUID>"} (enum field, UID as value string)
- Phase 5c State 5: `update_delivery_account` called with all 3 criteria.
- Phase 6: "Done" button. Terminal: "Your delivery account is ready to use for BrightPath Financial."

**Watch For:**
- Add-account flow starts at Phase 1b, goes to Phase 5, ends at Phase 6
- Multiple criteria accumulated correctly (APPEND, not overwrite)
- Enum field criterion uses leadFieldEnumUID as value
- Phase 6 button is "Done" not "Continue"

**Original Comparison:**
- Add-account flow: Original Phase 0b = Rework Phase 1b. Same ChoiceSet pattern for client and method selection.
- Key difference: Original creates account with ALL criteria (state + 3 additional) in ONE `create_delivery_account` call. Rework creates account with states first, then calls `update_delivery_account` with the 3 additional criteria. Same end state, different API call pattern.

---

### T19 — Add-Account, Skip Criteria at Phase 5 Gate

| Field | Value |
|-------|-------|
| **Flow** | Add-account |
| **Path** | Select client + select method + skip criteria at Phase 5 gate (never enter 5c) |

**Test Data:**
- Phase 1b: Select "Summit Lending" -> Select "Summit Lending-Portal"
- Phase 5: "$25" -> Click "Shared" -> Click "Yes" -> "OH, MI, IN" -> Click "Skip"
- Phase 6: Click "Done"

**Expected Behavior:**
- Phase 5 State 5: "Would you like to add additional lead criteria, or skip?" + ActionSet "Add criteria" | "Skip".
- User clicks "Skip": additionalCriteria="None". Agent calls summarize_history. Proceeds to Phase 6.
- Phase 5c is NEVER entered.
- Phase 6: Additional Criteria shows "None". "Done" button. Terminal message.

**Watch For:**
- Skip at Phase 5 gate does NOT enter Phase 5c
- Additional Criteria displays "None" in summary

**Original Comparison:** Original has skip inline in Phase 5 criteria section. Rework has explicit Phase 5 State 5 "criteria gate" with ActionSet before entering Phase 5c. Same user-facing experience (ActionSet "Skip" button). Original behavior had field suggestions shown BEFORE the skip prompt, while rework asks the skip/add question FIRST then shows suggestions only if "Add criteria" chosen. This is an intentional change — the original forced field suggestions before asking, which meant the user saw fields before being asked if they wanted criteria at all. Rework defers field display until the user opts in.

---

### T20 — Full-Setup, Webhook, Enum Criterion with Value Provided, Activate Fails, Keep Inactive

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + skip mapping + criterion with enum field + value provided in text + activate fails + keep inactive |

**Test Data:**
- Phase 1: "Clearwater Insurance, ops@clearwater.com"
- Phase 2: Click "Home Insurance"
- Phase 3: Click "24/7 delivery" -> Click "Webhook"
- Phase 3 Webhook: "https://clearwater.com/leads" -> Click "Skip for now"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$50" -> Click "Exclusive" -> Click "Yes" -> "FL, GA, AL, SC" -> Click "Add criteria"
- Phase 5c: See suggestions -> type "property type single family" (enum field WITH value)
- Phase 5c loop: Click "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate" (fails) -> Click "Keep Inactive"

**Expected Behavior:**
- Phase 5c: "property type single family" -> Agent parses field="Property Type", value="single family". Field is enumerated. Agent forces operator to "In". "single family" fuzzy-matches enum value "Single Family" at >85%. Resolves to leadFieldEnumUID silently. NO ChoiceSet displayed. Criterion appended directly.
- Phase 8: Activation fails. "We encountered an issue activating the client..." + "Retry activation" | "Keep Inactive". User picks "Keep Inactive". Terminal: "Setup complete. Client Clearwater Insurance has been configured but remains \"NEW\"."

**Watch For:**
- Enum field with value provided: fuzzy match >85% resolves silently (no ChoiceSet)
- Operator forced to "In" for enum fields regardless of what was parsed
- Phase 8 failure with "Keep Inactive" escape path
- Fresh password generated for each activation attempt

**Original Comparison:**
- Phase 8 "Keep Inactive" on failure: Original only offers "Retry activation" with no escape (critical UX defect). Rework adds "Keep Inactive" fallback. This test verifies the fix.
- Enum with value fuzzy-match: Original and rework have identical logic (fuzzy >85% resolves silently, else show ChoiceSet).

---

## Test Coverage Matrix

| Feature | Tests |
|---------|-------|
| **Flow: Full-Setup** | T01-T15, T20 |
| **Flow: Add-Method** | T16, T17 |
| **Flow: Add-Account** | T18, T19 |
| **Delivery: Portal** | T01, T09, T10, T15 |
| **Delivery: Email** | T02, T14 |
| **Delivery: FTP** | T05, T17 |
| **Delivery: Webhook** | T03, T04, T06-T08, T11-T13, T16, T20 |
| **Schedule: 24/7** | T01, T03, T04, T06-T15, T20 |
| **Schedule: Specific hours** | T02, T05, T16 |
| **Webhook: JSON mapping** | T03, T04, T13, T16 |
| **Webhook: XML mapping** | T06 |
| **Webhook: URL Encoded** | T07 |
| **Webhook: "I'm not sure"** | T04, T11 |
| **Webhook: Skip mapping** | T08, T20 |
| **Webhook: Parse failure** | T06 |
| **Webhook: URL pasted** | T13 |
| **Webhook: Input too large** | T12 |
| **Webhook: Switch content type** | T11 |
| **Webhook: Ambiguous fields** | T07 |
| **Connection test: Skip** | T03, T08 (3b), T04, T06 (3b) |
| **Connection test: Succeed** | T04, T08 (retry), T16, T17 |
| **Connection test: Fail + Skip** | T05 |
| **Connection test: Fail + Retry** | T08 |
| **Criteria: Skip at P5 gate** | T01, T08, T19 |
| **Criteria: Skip from suggestions** | T09 |
| **Criteria: Skip from show-more** | T10 |
| **Criteria: 1 non-enum** | T02 |
| **Criteria: 2 non-enum** | T03 |
| **Criteria: Enum (just name)** | T05, T18 |
| **Criteria: Enum (with value)** | T20 |
| **Criteria: Invalid field name** | T15 |
| **Criteria: Multiple mixed** | T18 |
| **Phase 2: Dropdown click** | T01-T13, T16-T20 |
| **Phase 2: Typed name** | T14 |
| **Phase 8: Activate success** | T01, T04, T07, T14 |
| **Phase 8: Keep Inactive** | T02, T10 |
| **Phase 8: Fail + Retry** | T03 |
| **Phase 8: Fail + Keep Inactive** | T20 |

---

## Tests Where Rework Differs From Original

These tests exercise rework-specific behavior that does NOT exist in the original:

| Test | Rework-Only Behavior | Rationale |
|------|---------------------|-----------|
| T01-T20 (Phase 5) | Account created with states first, criteria added via `update_delivery_account` | Two-call pattern ensures account exists before iterative criteria builder |
| T03, T20 (Phase 8) | "Keep Inactive" shown on activation failure | Fixes original defect: no escape from retry loop |
| T09, T10 (Phase 5c) | GLOBAL EXIT RULE routes back to Phase 5 resource on skip with empty criteria | Original handled skip inline; rework uses cross-resource routing |
| T14 (Phase 2) | Typed lead type name fuzzy-matched | Original only supported ChoiceSet click |
| T15 (Phase 5c) | No-match prompt includes "or say 'skip'" escape | Original no-match prompt had no skip suggestion |
| T19 (Phase 5) | Skip/Add criteria gate BEFORE field suggestions | Original showed field suggestions first, then offered skip |
| T07 (Webhook) | Ambiguous fields resolved via per-field ActionSet | Original unspecified card type for ambiguous resolution |

---

## Issue Regression Checklist

Every test should verify these former issues are resolved:

| Issue | What to check | Relevant tests |
|-------|---------------|----------------|
| F1 | ChoiceSet Action.Submit has NO data field | T01-T20 (Phase 2), T05/T18 (enum ChoiceSet) |
| F2 | Ambiguous fields use per-field ActionSet, not text dump | T07 |
| F3 | Mapping preview is Table card, not text arrows | T03, T04, T06, T07, T13, T16 |
| F4 | Summary cards render as valid JSON Table cards | All tests at Phase 4, 6, 7 |
| F5 | Phase 5 states in strict order (price first) | All tests reaching Phase 5 |
| F6 | Target states collected and included in account | All tests reaching Phase 5 |
| F7 | Each Phase 5 prompt is a separate turn | All tests reaching Phase 5 |
| F8 | Phase 5c shows field suggestions first | T02, T03, T05, T09, T10, T15, T18, T20 |
| F9 | Show more fields displays actual field names | T10 |
| F10 | Typed lead type name fuzzy-matches correctly | T14 |
| F11 | Exclusivity rendered as ActionSet buttons, not radio | All tests reaching Phase 5 |
| F13 | States evaluated in order, not batched/skipped | All tests |
| F14 | Action.Submit data field absent on all ChoiceSet cards | All ChoiceSet interactions |
| F15 | JSON template cards render without malformation | All Table card phases |
| F16 | DTO passed as object, not JSON string | All tool calls |
| F17 | No progress text before mapping preview card | T03, T04, T06, T07, T13, T16 |

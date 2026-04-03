# Delivery Setup Agent — Test Plan (Original Stabilized)

---

## Structural Notes

This test plan is for `/delivery-original-stabilized/` — the original linear-script format with context-management fixes from the rework.

Key structural differences from the rework test plan:
- **No Phase 3 router split** — Phase 3 is monolithic, handles all delivery types in one resource with IF branches
- **No Phase 5c split** — criteria builder is inline in Phase 5. Account created ONCE with all criteria (state + additional) in a single `create_delivery_account` call
- **No Phase 3a/3b split** — single Phase 3b handles both FTP and webhook tests, using `connectionTestMode` flag
- **Single anchor** — all `summarize_history` calls use `DELIVERY_SETUP_START`, carrying ALL accumulated state
- **Phase 4 and Phase 6** now have `summarize_history` calls for clean context boundaries

---

## T01 — Full-Setup, Portal, 24/7, Skip Criteria, Activate

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Portal + 24/7 + skip criteria + activate |

**Test Data:**
- Phase 1: "Acme Solar, john@acmesolar.com"
- Phase 2: Select lead type from dropdown
- Phase 3: Click "24/7 delivery" → Click "Portal"
- Phase 3b: connectionTestMode="none" → skips directly to summarize
- Phase 4: Click "Continue"
- Phase 5: "$25" → Click "Exclusive" → Click "Yes" → "CA, AZ, TX" → Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 1: Single prompt (not duplicated), client created silently
- Phase 2: `display_lead_types_choice` tool used (not manual get_lead_types + card)
- Phase 3: Schedule ActionSet first, then delivery type ActionSet. Portal creates method silently. connectionTestMode="none" retained.
- Phase 3b: connectionTestMode="none" → goes directly to summarize_history (no test prompt shown)
- Phase 4: Table card with summary. "Continue" button. summarize_history called after click.
- Phase 5: Price → Exclusivity → Order System → Target States → Field suggestions → Skip. STRICT order. Target states MUST NOT be skipped.
- Phase 6: Table card. isExclusive shows "Exclusive" (not "true"). useOrder shows "Enabled" (not "true"). "Continue" button. summarize_history called after click.
- Phase 7: Table card with "Activate" | "Keep Inactive"
- Phase 8: update_client with ALL fields, clientStatus="Active", fresh password. Success message.

**Watch For:**
- Phase 1 prompt NOT duplicated
- Phase 5 target states NOT skipped
- Phase 5 steps in strict sequential order (price before exclusivity before order before states before criteria)
- Phase 4 and 6 summaries appear in correct position (before next phase, not after)
- Boolean display: "Exclusive"/"Shared", "Enabled"/"Disabled"

---

## T02 — Full-Setup, Email, Specific Hours, Add One Criterion, Keep Inactive

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Email + specific hours + 1 non-enum criterion + keep inactive |

**Test Data:**
- Phase 1: "GreenLeaf Energy, sarah@greenleaf.co"
- Phase 2: Select "Mortgage" (or LendingTree)
- Phase 3: Click "Specific hours only" → "Mon-Fri 8am-6pm EST" → Click "Email"
- Phase 3b: connectionTestMode="none" → skips test
- Phase 4: Click "Continue"
- Phase 5: "30" → Click "Shared" → Click "No" → "NY, NJ, CT, PA" → type "loan amount minimum 50000" → "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Keep Inactive"

**Expected Behavior:**
- Phase 3: "Specific hours only" → separate prompt for schedule text → user types "Mon-Fri 8am-6pm EST" → schedule parsed with ISO format and timezone → delivery type prompt in same or next turn
- Phase 3: Email creates method silently. connectionTestMode="none".
- Phase 5: Field suggestions MUST appear before criterion input. User types "loan amount minimum 50000". Agent parses: field="LoanAmount", operator="GreaterOrEqual", value="50000". Criteria loop asks "Would you like to add another criterion?" → user says "Continue" → exits loop.
- Phase 5: Account created with state criteria + loan amount criterion in ONE call.
- Phase 6: Additional Criteria shows the loan amount criterion. Exclusivity shows "Shared". Order System shows "Disabled".
- Phase 8: "Setup complete. Client GreenLeaf Energy has been configured but remains \"NEW\"."

**Watch For:**
- Schedule parsing with correct timezone offset
- Field suggestions shown before criterion input
- Criteria loop asks again after adding criterion (doesn't auto-exit)
- "Keep Inactive" path works correctly
- Conversational prompts (URL, schedule, posting instructions) use plain text, NOT adaptive cards

---

## T03 — Full-Setup, Webhook, JSON Mapping, Test Connection, Multiple Criteria, Activate

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + JSON mapping + test succeeds + 2 criteria + activate |

**Test Data:**
- Phase 1: "PeakLend Corp, mike@peaklend.com"
- Phase 2: Select lead type
- Phase 3: Click "24/7 delivery" → Click "Webhook"
- Phase 3 Webhook: "https://api.peaklend.com/leads" → Click "I'll provide instructions" → Click "JSON" → Paste JSON schema
- Phase 3 Mapping preview: Click "Continue"
- Phase 3b: Click "Test Connection" (succeeds)
- Phase 4: Click "Continue"
- Phase 5: "$45.50" → Click "Exclusive" → Click "Yes" → "CA, NV, OR, WA" → type "loan amount at least 25000" → type "annual income minimum 50000" → "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3 Webhook: URL collected via plain text (no card). Mapping choice via ActionSet. Content type via ActionSet. Schema via plain text.
- Phase 3 Schema validation: Agent uses best judgment to interpret JSON. Missing braces auto-fixed. Only rejects truly unintelligible input.
- Phase 3 Field mapping: Silent processing. Table card preview with mapped fields. "Continue" button.
- Phase 3 Method creation: requestBody template with [SystemFieldName] placeholders. connectionTestMode="webhook" retained.
- Phase 3b: connectionTestMode="webhook" → offers test. Test succeeds → goes directly to summarize (no re-prompt). Connection test prompt shown EXACTLY ONCE.
- Phase 5: Two criteria added. Criteria loop re-enters after each (asks "Would you like to add another?"). Only exits on explicit "Continue".
- Phase 5: Account created with state criteria + 2 additional criteria in ONE call. criteriaPayload has 3 entries total.
- Phase 6: Shows all criteria in Additional Criteria field.
- Phase 8: Activation succeeds.

**Watch For:**
- JSON schema validation is lenient (auto-fixes, doesn't reject parseable input)
- Skip mapping option available if validation fails
- Connection test shown once (not twice)
- Multiple criteria accumulated (appended, not overwritten)
- Criteria loop only exits on explicit user action

---

## T04 — Full-Setup, Webhook, Skip Mapping, Skip Test, Skip Criteria

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + skip mapping + skip test + skip criteria (fastest path) |

**Test Data:**
- Phase 1: "QuickSetup Inc, quick@setup.com"
- Phase 2: Select lead type
- Phase 3: Click "24/7 delivery" → Click "Webhook" → "https://quick.io/leads" → Click "Skip for now"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$100" → Click "Shared" → Click "No" → "TX" → Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3: Method created with settings=null, requestBody=null. mappedCount=0, totalCount=0. connectionTestMode="webhook".
- Phase 3b: Skip → straight to summarize.
- Phase 4: Field Mappings shows "0 of 0 fields mapped".
- Phase 5: Target states asked (not skipped). Criteria skipped → additionalCriteria="None". Account created with state criteria only.
- Phase 6: Additional Criteria shows "None".

---

## T05 — Full-Setup, FTP, SFTP Protocol, Test Fails Then Retry

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | FTP + SFTP protocol + test fails + retry + succeeds + skip criteria |

**Test Data:**
- Phase 1: "Harbor Mortgage, admin@harbormtg.com"
- Phase 2: Select lead type
- Phase 3: Click "Specific hours only" → "Weekdays 7am-7pm PST" → Click "FTP" → "sftp://ftp.harbormtg.com\nftpuser123\nS3cur3P@ss!"
- Phase 3b: Click "Test Connection" (fails) → Click "Retry" (succeeds)
- Phase 4: Click "Continue"
- Phase 5: "$35" → Click "Exclusive" → Click "Yes" → "CA, OR, WA" → Click "Skip"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 3: FTP branch. connectionTestMode="ftp" retained.
- Phase 3b: connectionTestMode="ftp" used (not URL heuristic). Protocol detected as "SFTP" (sftp:// prefix). Tool uses {protocol} not {detectedProtocol}.
- Phase 3b: Test fails → "Retry" loops to tool call directly (does NOT re-show "Would you like to test?" prompt). Test succeeds → goes to summarize. summarize_history called EXACTLY ONCE.
- Phase 3b summary carries ALL Phase 3 state (deliveryMethodUID, ftpUser, ftpPassword, etc.)

---

## T06 — Full-Setup, Webhook, Enum Criterion

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + skip mapping + enum criterion with ChoiceSet |

**Test Data:**
- Phase 1: "EnumTest Co, enum@test.com"
- Phase 2: Select lead type (one with enumerated fields)
- Phase 3: Click "24/7 delivery" → Click "Webhook" → URL → Click "Skip for now"
- Phase 3b: Click "Skip"
- Phase 4: Click "Continue"
- Phase 5: "$50" → Click "Exclusive" → Click "No" → "FL, GA" → type "property type" (enum field, no value)
- Phase 5: Agent shows ChoiceSet dropdown with enum values → user selects → criteria loop asks again → "Continue"
- Phase 6: Click "Continue"
- Phase 7: Click "Activate"

**Expected Behavior:**
- Phase 5: "property type" fuzzy-matches an enumerated field. Agent forces operator to "In". Displays ChoiceSet dropdown with enum values (value=leadFieldEnumUID, title=display text). Action.Submit has NO data field.
- Phase 5: User selects from dropdown. leadFieldEnumUID stored as criterion value string.
- Phase 5: Criteria loop asks "Would you like to add another?" after enum selection (doesn't auto-exit).
- Phase 5: Account created with state + property type criterion.

---

## T07 — Add-Method Flow, Email

| Field | Value |
|-------|-------|
| **Flow** | Add-method |
| **Path** | Select existing client → Email |

**Test Data:**
- Action: create-delivery-method.md entry
- Phase 0a: Select existing client from dropdown
- Phase 2: Select lead type
- Phase 3: Click "24/7 delivery" → Click "Email"
- Phase 3b: connectionTestMode="none" → skip
- Phase 4: Click "Done"

**Expected Behavior:**
- Action file sets flowIntent="add-method" directly (no Phase 0 intent detection)
- Phase 0a: get_clients called. If empty, "No clients found" message. Otherwise ChoiceSet dropdown.
- Phase 0a summary carries: flowIntent, clientUID, companyName, email, clientStatus, timeZoneName, timeOffset
- Phase 4: Button says "Done" (not "Continue"). Conversation ends with success message. No Phase 5.

---

## T08 — Add-Account Flow, Multiple Criteria

| Field | Value |
|-------|-------|
| **Flow** | Add-account |
| **Path** | Select existing client + method → multiple criteria |

**Test Data:**
- Action: create-delivery-account.md entry
- Phase 0b: Select existing client → Select existing delivery method
- Phase 5: "$100" → Click "Shared" → Click "Yes" → "AZ, NV" → type "loan amount > 500000" → type "property value > 300000" → "Continue"
- Phase 6: Click "Done"

**Expected Behavior:**
- Action file sets flowIntent="add-account" directly
- Phase 0b: Both get_clients and get_delivery_methods called. Empty list handling for both.
- Phase 0b summary carries all 10 variables
- Phase 5: Criteria loop with 2 criteria. Both appended to criteriaPayload.
- Phase 6: Button says "Done". Conversation ends. No Phase 7/8.

---

## T09 — Partial Input Handling

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Partial input on Phase 1 |

**Test Data:**
- Phase 1: First message: "test@email.com" (email only, no company name)
- Phase 1: Second message: "TestCompany"

**Expected Behavior:**
- Phase 1: First response retains email, asks ONLY for company name (not both fields again)
- Phase 1: Second response has both fields, proceeds to create client

---

## T10 — Webhook, "I'm Not Sure" Format, Auto-Detect

| Field | Value |
|-------|-------|
| **Flow** | Full-setup |
| **Path** | Webhook + "I'm not sure" content type + auto-detect |

**Test Data:**
- Phase 3 Webhook: URL → "I'll provide instructions" → "I'm not sure" → Paste JSON without braces: `"first_name": "John", "last_name": "Doe", "email": "test@test.com"`

**Expected Behavior:**
- Phase 3: Agent detects JSON format. Shows "I've detected this as JSON format. Is this correct?" with ActionSet.
- User confirms → agent fixes missing braces silently → proceeds to field mapping
- Schema validation is lenient — does NOT reject input with missing braces

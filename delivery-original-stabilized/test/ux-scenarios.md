# Original Delivery Setup — Expected UX Scenarios

This document captures every user-facing interaction in the **original** `/delivery` system. Use as the baseline reference for comparing against actual agent behavior and against the rework.

---

## Entry Points

All three action files (create-single-client, create-delivery-method, create-delivery-account) are **identical** — they all run Phase 0 intent detection. The action file does NOT enforce a specific flow; the agent must detect or ask.

---

## Phase 0 — Flow Intent Detection

**Scenario 0.1: Intent detected silently**
- Agent detects intent from user's initial message (e.g., "create client" → full-setup, "create delivery method" → add-method, "create delivery account" → add-account).
- No prompt shown. Agent routes directly.

**Scenario 0.2: Intent unclear**
- Agent says: "I can help you with:\n\n1. Setting up a new client (full setup)\n2. Adding a delivery method to an existing client\n3. Adding a delivery account using an existing method\n\nWhich would you like to do?"
- Input: free text.
- Agent waits.

**Routing after intent resolved:**
- full-setup → Phase 1
- add-method → Phase 0a
- add-account → Phase 0b

---

## Flow 1: Full Setup

### Phase 1 — Create Client

**Scenario 1.1: Collect company name and email**
- Agent says: "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email"
- Input: free text (two values).
- Agent waits.

**Scenario 1.2: Create client**
- Agent calls `create_client` silently.
- On success: moves to Phase 2. No explicit error/retry prompt defined in original.

**Note:** Original Phase 1 has no partial-field handling (no separate prompts for missing companyName vs email). It asks for both in a single prompt and expects both back.

---

### Phase 2 — Select Lead Type

**Scenario 2.1: Display lead type selector**
- Agent calls `get_lead_types`.
- Agent says: "Please select a Lead Type for this client."
- Card: ActionSet (≤4 options) or Input.ChoiceSet compact (>4 options).
- Agent waits.

**Note:** Original Phase 2 has no explicit retry/ambiguous-selection handling. It simply asks, waits, and retains the choice.

---

### Phase 3 — Create Delivery Method

#### Step 3.1: Delivery Schedule

**Scenario 3.1: Schedule choice**
- Agent says: "First, let's set the delivery schedule.\n\nWould you like leads delivered 24/7, or only during specific hours?"
- Card: ActionSet — "24/7 delivery" | "Specific hours only"
- Agent waits.

**Scenario 3.2: User picks "Specific hours only"**
- Agent says: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)"
- Input: free text.
- Agent waits.
- Agent parses schedule, builds deliveryDays array (7 entries).

**Scenario 3.3: User picks "24/7 delivery"**
- Agent builds deliveryDays array (all days allowed, UTC). No extra prompt.

#### Step 3.2: Delivery Type

**Scenario 3.4: Delivery type choice**
- Agent says: "How would you like your leads delivered?\n\n• Webhook – sends lead data via HTTP POST\n• Portal – client accesses leads via web portal\n• FTP – uploads lead files to a server\n• Email – delivers leads to an inbox"
- Card: ActionSet — "Portal" | "Webhook" | "Email" | "FTP" (button order differs from bullet order)
- Agent waits.

---

### Phase 3 — Portal Branch

**Scenario 3P.1: Create portal method**
- No user input needed.
- Agent calls `create_delivery_method` silently.
- Moves to Phase 3b.

---

### Phase 3 — Email Branch

**Scenario 3E.1: Create email method**
- No user input needed.
- Agent calls `create_delivery_method` silently.
- Moves to Phase 3b.

---

### Phase 3 — FTP Branch

**Scenario 3F.1: Collect FTP credentials**
- Agent says: "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"
- Input: free text (three values).
- Agent waits.

**Scenario 3F.2: Create FTP method**
- Agent calls `create_delivery_method` silently.
- Moves to Phase 3b.

---

### Phase 3 — Webhook Branch

**Scenario 3W.1: Collect webhook URL**
- Agent says: "What's your webhook URL where we should send the leads?"
- Input: free text.
- Agent waits.
- Agent prepends `https://` if scheme missing.

**Scenario 3W.2: Field mapping choice**
- Agent says: "Would you like to configure field mappings?\n\nIf you have posting instructions or API documentation, I can automatically extract the field mappings."
- Card: ActionSet — "I'll provide instructions" | "Skip for now"
- Agent waits.

**Scenario 3W.3: User picks "Skip for now"**
- Agent calls `create_delivery_method` (no mapping, settings=null, requestBody=null).
- Moves to Phase 3b.

**Scenario 3W.4: User picks "I'll provide instructions"**
- Agent calls `get_lead_type(leadTypeUID)` to load leadFields.
- Proceeds to content type selection.

**Scenario 3W.5: Content type choice**
- Agent says: "What content type should this delivery use?"
- Card: ActionSet — "URL Encoded" | "JSON" | "XML" | "I'm not sure"
- Agent waits.

**Scenario 3W.6a: Posting instructions prompt (content type = "I'm not sure")**
- Agent says: "Please paste your posting instructions or API schema, and I'll detect the format automatically."
- Agent waits.

**Scenario 3W.6b: Posting instructions prompt (content type = "URL Encoded")**
- Agent says: "Please provide field names (comma or line-separated) or a request body example."
- Agent waits.

**Scenario 3W.6c: Posting instructions prompt (content type = "JSON" or "XML")**
- Agent says: "Please paste the {contentType} schema that your client's API expects."
- Agent waits.

**Scenario 3W.7: User pastes a URL instead of content**
- Agent says: "I can't access external links. Please open the page and paste the posting instructions or schema text here."
- Agent waits. On re-entry, re-validates.

**Scenario 3W.8: Input too large (>5000 chars or >1500 tokens or >200 repetitive items)**
- Agent says: "This looks too large or unformatted. Would you like to simplify or skip mapping for now?"
- Card: ActionSet — "Simplify input" | "Skip mapping"
- Agent waits.

**Scenario 3W.8a: User picks "Simplify input"**
- Agent says: "Please paste a smaller, well-formatted excerpt (e.g., the request body and a short field list)."
- Agent waits. On re-entry, re-validates.

**Scenario 3W.8b: User picks "Skip mapping"**
- Sets skipFieldMapping=true. Jumps to Scenario 3W.3 (create without mapping).

**Scenario 3W.9: Auto-detect confirmation (only if "I'm not sure")**
- Agent detects format (JSON object/array → "JSON", XML markup → "XML", otherwise → "URL Encoded").
- Agent says: "I've detected this as {detectedFormat} format. Is this correct?"
- Card: ActionSet — "Continue with {detectedFormat}" | "Switch content type"
- Agent waits.

**Scenario 3W.9a: User picks "Continue with {detectedFormat}"**
- Sets contentType to detected format. Proceeds to validation/mapping.

**Scenario 3W.9b: User picks "Switch content type"**
- Returns to Scenario 3W.5 (content type choice).

**Scenario 3W.10: JSON/XML parse failure**
- Agent first attempts auto-fix (missing braces, trailing commas, unescaped quotes, single→double quotes).
- If parsing still fails after auto-fix:
- Agent says: "I couldn't parse this as valid {contentType}. Would you like to:\n• Fix and re-paste the {contentType} schema\n• Switch to a different content type"
- Card: ActionSet — "Re-paste {contentType} schema" | "Switch content type"
- Agent waits.

**Scenario 3W.10a: User picks "Re-paste"**
- Returns to posting instructions prompt for that content type.

**Scenario 3W.10b: User picks "Switch content type"**
- Returns to Scenario 3W.5 (content type choice).

**Scenario 3W.11: Ambiguous field mapping**
- Agent prompts user to select correct system field from candidates.
- Agent waits.

**Scenario 3W.12: No confident field match**
- Agent asks user to clarify that delivery field.
- Agent waits.

**Scenario 3W.13: Mapping preview**
- Card: Table with columns System Field | Delivery Field | Status
- One row per mapped field showing "✓ Mapped".
- Message: "Successfully mapped {mappedCount} out of {totalCount} fields."
- Button: "Continue"
- Agent waits.

**Scenario 3W.14: Create webhook method with mapping**
- Agent calls `create_delivery_method` with mappingSettings + requestBody.
- Moves to Phase 3b.

**Note:** Original Phase 3 has no explicit error/retry messages for create_delivery_method failure. The rework adds these.

---

### Phase 3b — Test Connection

**Eligibility:** Only shown if deliveryType = "FTP" OR (deliveryType = "HttpPost" AND deliveryAddress starts with "http").

**Note:** Portal and Email skip Phase 3b entirely because Portal has deliveryAddress="Portal" and Email has deliveryAddress={email} — neither starts with "http". This means the test prompt is only shown for FTP and Webhook.

**Scenario 3b.1: Offer test**
- Agent says: "Would you like to test the connection to your endpoint before continuing?"
- Card: ActionSet — "Test Connection" | "Skip"
- Agent waits.

**Scenario 3b.2: User picks "Skip"**
- Moves to Phase 4 (summary).

**Scenario 3b.3: User picks "Test Connection" (FTP)**
- Agent detects protocol (FTP or SFTP from deliveryAddress).
- Agent calls `test_ftp_sftp_connection`.
- **Known bug:** Source uses `{detectedProtocol}` but the PROCESS step stores it as `protocol`. Variable name mismatch.

**Scenario 3b.4: User picks "Test Connection" (Webhook)**
- If mimeContentType and requestBody known: generates testPayload with sample values (string→"Test", email→"test@example.com", phone→"5551234567", numeric→"50.00", bool→"true", date→"2025-01-01") and calls `test_webhook_connection` with contentType and payload.
- Otherwise: calls `test_webhook_connection` with empty payload.

**Scenario 3b.5: Test succeeds**
- Agent says: "✓ Connection test successful."
- Card: ActionSet — "Continue"
- Agent waits. On click: moves to Phase 4.

**Scenario 3b.6: Test fails**
- Agent says: "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later."
- Card: ActionSet — "Retry" | "Skip"
- Agent waits.

**Scenario 3b.6a: User picks "Retry"**
- Re-runs the test.

**Scenario 3b.6b: User picks "Skip"**
- Moves to Phase 4 (summary).

---

### Phase 4 — Delivery Method Summary

**Scenario 4.1: Display summary card**
- Card: Table showing Company Name, Lead Type, Delivery Method, Delivery Type, Delivery Hours, Field Mappings ({mappedCount} of {totalCount} fields mapped).
- Button: "Continue" (full-setup) or "Done" (add-method).
- Agent waits.

**Scenario 4.2: Full-setup — user clicks "Continue"**
- Moves to Phase 5.

**Scenario 4.3: Add-method — user clicks "Done"**
- Agent says: "✓ Your delivery method is ready to use for {companyName}."
- Conversation ends.

---

### Phase 5 — Create Delivery Account

**Scenario 5.1: Collect price**
- Agent says: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead."
- Input: free text.
- Agent waits.

**Scenario 5.2: Collect exclusivity**
- Agent says: "Will this client receive exclusive or shared leads?"
- Card: ActionSet — "Exclusive" | "Shared"
- Agent waits.

**Scenario 5.3: Collect order system preference**
- Agent says: "Would you like to enable the Order System for this client?"
- Card: ActionSet — "Yes" | "No"
- Agent waits.

**Scenario 5.4: Load lead fields and detect state field (silent)**
- Agent calls `get_lead_type(leadTypeUID)` to retrieve leadTypeName and leadFields.
- Detects state field (specialBit → name exact → name contains).
- If confidence < 5%: prompts user to confirm.

**Scenario 5.5: Collect target states**
- Agent says: "Which states do you want to target? (e.g., CA, AZ, TX)"
- Input: free text.
- Agent waits.

**Scenario 5.6: Show field suggestions (MANDATORY)**
- Agent says: "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {field1}\n• {field2}\n• ..."
- If extraFieldCount > 0, appends: "\nThere are {extraFieldCount} more fields available.\n\nWould you like to add criteria, see more fields, or skip?"
- If extraFieldCount = 0, appends: "\nWould you like to add criteria or skip?"
- Card: ActionSet — "Show more fields" | "Skip" (if extraFieldCount > 0) or just "Skip"
- Agent waits.

**Scenario 5.6a: User types a criterion directly**
- Agent parses natural-language criterion.
- Fuzzy-matches field name against leadFieldsMap (>90% confidence).
- Validates operator against field type rules.
- If field is enumerated: see Scenario 5.6d/5.6e below.
- Adds to parsedCriteria. Proceeds to criteria loop (Scenario 5.7).

**Scenario 5.6b: User picks "Show more fields"**
- Agent says: "Additional Fields (showing up to 10):\n\n• {field1}\n• {field2}\n• ..."
- Card: ActionSet — "Show more fields" | "Skip"
- Agent waits. Returns to Scenario 5.6.

**Scenario 5.6c: User picks "Skip"**
- Retains additionalCriteria="None". Proceeds to account creation (Scenario 5.8).

**Scenario 5.6d: Enumerated field — no valid value provided**
- Agent says: "I see you want to filter by {fieldName}.\nPlease select a value:"
- Card: Input.ChoiceSet compact with enum values, placeholder "Select a value" + Submit button.
- Agent waits.

**Scenario 5.6e: Enumerated field — only field name mentioned**
- Agent says: "Please select a value for {fieldName}:"
- Card: Input.ChoiceSet compact with enum values, placeholder "Select a value" + Submit button.
- Agent waits.

**Scenario 5.6f: No field match**
- Agent says: "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options."
- Agent waits.

**Scenario 5.7: Criteria loop (after first criterion added)**
- Agent says: "Would you like to add another criterion, see more fields, or continue?"
- Card: ActionSet — "Show more fields" | "Continue" (if more extra fields remain) or just "Continue"
- Agent waits.

**Scenario 5.7a: User provides another criterion**
- Returns to parsing (Scenario 5.6a).

**Scenario 5.7b: User picks "Show more fields"**
- Shows additional fields, returns to criteria loop.

**Scenario 5.7c: User picks "Continue" (or says continue/done/no)**
- Builds additionalCriteria summary string. Proceeds to account creation.

**Scenario 5.8: Create delivery account**
- Agent calls `get_usa_states`, matches targetStates, builds criteriaPayload (state criterion first, then any parsed criteria).
- Agent calls `create_delivery_account`.
- On success: moves to Phase 6.

**KEY DIFFERENCE from rework:** In the original, the account is created AFTER all criteria are collected (single create call with full criteriaPayload). In the rework, the account is created with state-only criteria first, then additional criteria are added via `update_delivery_account`.

---

### Phase 6 — Delivery Account Summary

**Known bug:** The Phase 5 source file contains a duplicate Phase 6 definition that routes to itself (`NEXT_PHASE: mcp://resource/phase-6-delivery-account-summary`). The standalone Phase 6 resource file correctly routes to Phase 7.

**Scenario 6.1: Display summary card**
- Card: Table showing Company Name, Lead Type, Delivery Method, Price per Lead, Lead Exclusivity, Target States, Additional Criteria, Order System.
- Boolean display: isExclusive/useOrder shown per data_normalization rules.
- Button: "Continue" (full-setup) or "Done" (add-account).
- Agent waits.

**Scenario 6.2: Full-setup — user clicks "Continue"**
- Moves to Phase 7.

**Scenario 6.3: Add-account — user clicks "Done"**
- Agent says: "✓ Your delivery account is ready to use for {companyName}."
- Conversation ends.

**Note:** Original Phase 6 does NOT explicitly format booleans as "Exclusive"/"Shared" or "Yes"/"No" in the card template — it passes raw `{isExclusive}` and `{useOrder}` and relies on `data_normalization` rules. The rework makes this explicit.

---

### Phase 7 — Client Summary (full-setup only)

**Scenario 7.1: Display summary card**
- Card: Table showing Company Name, Contact Email, Client Status, Lead Type, Delivery Method (with ID), Delivery Account (with ID).
- Message: "Review your configuration above. Would you like to activate the client now?"
- Buttons: "Activate" | "Keep Inactive"
- Agent waits.

**Scenario 7.2: User clicks "Activate" or "Keep Inactive"**
- Moves to Phase 8.

---

### Phase 8 — Activation (full-setup only)

**Scenario 8.1: Activate client**
- Agent calls `update_client`.
- **Known issue:** Source TOOL_DEFAULTS lists fields in a flat format alongside clientUID, but a CRITICAL note says "updateClientDto must be passed as a native object" — the intent is nested, but the syntax is ambiguous. The rework makes the nesting explicit.
- On success: "✓ Setup complete. Your lead delivery system is now \"ACTIVE\" for {companyName}."
- Conversation ends.

**Scenario 8.2: Activation fails**
- Agent says: "We encountered an issue activating the client. Would you like to try again?"
- Card: ActionSet — "Retry activation"
- Agent waits.

**Scenario 8.2a: User picks "Retry activation"**
- Re-calls `update_client` with same parameters.

**Scenario 8.3: Keep inactive**
- Agent says: "✓ Setup complete. Client {companyName} has been configured but remains \"NEW\". You can activate it later when ready."
- Conversation ends.

---

## Flow 2: Add Method

### Phase 0a — Select Client

**Scenario 0a.1: clientUID already in context**
- Skips selector, uses existing clientUID.

**Scenario 0a.2: No clientUID — show client selector**
- Agent calls `get_clients`.
- Agent says: "Which client would you like to create a delivery method for?"
- Card: Input.ChoiceSet compact (companyName as display, clientUID as value).
- Agent waits.

**Scenario 0a.3: Client selected**
- Agent calls `get_client(clientUID)` to load profile.
- Moves to Phase 2.

**Note:** Original Phase 0a has no error handling for get_clients or get_client failure, and no ambiguous-selection retry. The rework adds these.

### Then: Phase 2 → Phase 3 → Phase 3b → Phase 4 (Done)

Same scenarios as Flow 1, except Phase 4 button is "Done" and conversation ends.

---

## Flow 3: Add Account

### Phase 0b — Select Client and Delivery Method

**Scenario 0b.1: clientUID already in context**
- Skips client selector.

**Scenario 0b.2: No clientUID — show client selector**
- Agent calls `get_clients`.
- Agent says: "Which client would you like to create a delivery account for?"
- Card: Input.ChoiceSet compact.
- Agent waits.

**Scenario 0b.3: Client selected — show delivery method selector**
- Agent calls `get_client(clientUID)` to load profile.
- Agent calls `get_delivery_methods(clientUID)`.
- Agent says: "Which delivery method would you like to use?"
- Card: Input.ChoiceSet compact (deliveryMethodName as display, deliveryMethodUID as value).
- Agent waits.

**Scenario 0b.4: Method selected**
- Retains deliveryMethodUID, deliveryMethodName, leadTypeUID. Moves to Phase 5.

**Note:** Original Phase 0b has no error handling for empty lists, failed API calls, or ambiguous selections. The rework adds all of these.

### Then: Phase 5 → Phase 6 (Done)

Same scenarios as Flow 1, except Phase 6 button is "Done" and conversation ends.

---

## Known Bugs in Original (from architecture-issues.md)

| # | Severity | Issue |
|---|----------|-------|
| 1 | Critical | Phase 5 file contains a duplicate Phase 6 definition that routes to itself instead of Phase 7 |
| 2 | Critical | Phase 3/3b have ambiguous summary ownership — Phase 3b summary may erase Phase 3 state |
| 3 | High | Multiple additional criteria may not be preserved in payload — earlier criteria can be dropped |
| 4 | High | FTP test uses `{detectedProtocol}` but process stores as `protocol` (variable name mismatch) |
| 5 | Medium | Connection test eligibility is heuristic (checks if deliveryAddress starts with "http") |
| 6 | Medium | All three action entry points are identical — don't enforce intent |
| 7 | Medium | Phase 8 updateClientDto contract ambiguous (TOOL_DEFAULTS flat format vs CRITICAL note requiring nested object) |

---

## Key Differences from Rework

| Aspect | Original `/delivery` | Rework `/delivery-rework` |
|--------|---------------------|--------------------------|
| Phase 0 | Shared intent detector across all 3 actions | Removed; each action routes directly to its phase |
| Phase 1 | Single prompt for both fields | Separate prompts per missing field |
| Phase 3 | One monolithic file | Split into portal/email/ftp/webhook files |
| Phase 3b | Single combined test file | Split into ftp-test/webhook-test files |
| Error handling | Minimal — no explicit retry prompts in most phases | Every phase has explicit error messages and STOP AND YIELD |
| Ambiguous selection | No retry handling | Re-displays selector with error message |
| Empty lists | Not handled | Explicit messages ("I couldn't find any...") |
| Account creation | One `create_delivery_account` call with all criteria | Create with state-only, then `update_delivery_account` for additional |
| Phase 6 routing | Bug: duplicate definition routes to self | Fixed: single definition routes to Phase 7 |
| Phase 8 DTO | Ambiguous — flat TOOL_DEFAULTS but CRITICAL note says nest in updateClientDto | Explicitly nested in updateClientDto |
| Boolean display | Relies on data_normalization rules | Explicit mapping (Exclusive/Shared, Yes/No) |

---

## Summary: All STOP AND YIELD Points

| # | Phase | Trigger |
|---|-------|---------|
| 1 | 0 | Waiting for flow intent clarification |
| 2 | 1 | Waiting for companyName and email |
| 3 | 2 | Waiting for lead type selection |
| 4 | 3 | Waiting for schedule choice |
| 5 | 3 | Waiting for specific hours description |
| 6 | 3 | Waiting for delivery type choice |
| 7 | 3-FTP | Waiting for FTP credentials |
| 8 | 3-Webhook | Waiting for webhook URL |
| 9 | 3-Webhook | Waiting for field mapping choice |
| 10 | 3-Webhook | Waiting for content type choice |
| 11 | 3-Webhook | Waiting for posting instructions |
| 12 | 3-Webhook | URL detected in input, asking for raw text |
| 13 | 3-Webhook | Input too large, asking to simplify or skip |
| 14 | 3-Webhook | Waiting for simplified input |
| 15 | 3-Webhook | Waiting for auto-detect confirmation |
| 16 | 3-Webhook | JSON/XML parse failed, asking to re-paste or switch |
| 17 | 3-Webhook | Ambiguous field mapping, asking user to select |
| 18 | 3-Webhook | No confident match, asking for clarification |
| 19 | 3-Webhook | Waiting for user to click Continue on mapping preview |
| 20 | 3b | Waiting for test choice |
| 21 | 3b | Test succeeded, waiting for Continue |
| 22 | 3b | Test failed, waiting for Retry or Skip |
| 23 | 4 | Waiting for Continue or Done |
| 24 | 5 | Waiting for price |
| 25 | 5 | Waiting for exclusivity choice |
| 26 | 5 | Waiting for order system choice |
| 27 | 5 | Waiting for target states |
| 28 | 5 | Waiting for response to field suggestions |
| 29 | 5 | Showing additional fields, waiting for response |
| 30 | 5 | Enum field, waiting for value selection |
| 31 | 5 | No field match, waiting for clarification |
| 32 | 5 | Criteria loop, waiting for add/show/continue |
| 33 | 6 | Waiting for Continue or Done |
| 34 | 7 | Waiting for Activate or Keep Inactive |
| 35 | 8 | Activation failed, waiting for Retry |
| 36 | 0a | Waiting for client selection |
| 37 | 0b | Waiting for client selection |
| 38 | 0b | Waiting for delivery method selection |

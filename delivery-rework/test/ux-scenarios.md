# Delivery Setup — Expected UX Scenarios

This document captures every user-facing interaction across all three flow variants. Use it to compare expected behavior against actual agent behavior during testing.

---

## Entry Points

| Flow | Action Resource | First Phase |
|------|----------------|-------------|
| Full Setup | rw-create-single-client | Phase 1 |
| Add Method | rw-create-delivery-method | Phase 1a |
| Add Account | rw-create-delivery-account | Phase 1b |

---

## Flow 1: Full Setup

### Phase 1 — Create Client

**Scenario 1.1: Both fields missing**
- Agent says: "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email"
- Input: free text
- Agent waits.

**Scenario 1.2: Only companyName missing**
- Agent says: "Please provide the Company Name."
- Agent waits.

**Scenario 1.3: Only email missing**
- Agent says: "Please provide the Contact Email."
- Agent waits.

**Scenario 1.4: Both fields provided, create client**
- Agent calls `create_client` silently.
- On success: retains clientUID, moves to Phase 2.
- On repairable failure: agent retries once silently.
- On persistent failure: "I ran into an issue creating the client.\n\nPlease confirm the company name and contact email and I'll try again."
- Agent waits.

---

### Phase 2 — Select Lead Type

**Scenario 2.1: Display lead type selector**
- Agent calls `get_lead_types`.
- Agent says: "Please select a Lead Type for this client."
- Card: ActionSet (≤4 options) or Input.ChoiceSet compact (>4 options).
- Agent waits.

**Scenario 2.2: User types instead of clicking**
- Agent fuzzy-matches against leadTypesList.
- If ambiguous or low confidence: "I couldn't match that selection.\n\nPlease select a Lead Type for this client."
- Re-displays selector card. Agent waits.

**Scenario 2.3: Selection confirmed**
- Agent retains leadTypeUID, leadTypeName. Moves to Phase 3.

---

### Phase 3 — Delivery Method Router

#### Step 3.1: Delivery Schedule

**Scenario 3.1: Schedule choice**
- Agent says: "First, let's set the delivery schedule.\n\nWould you like leads delivered 24/7, or only during specific hours?"
- Card: ActionSet — "24/7 delivery" | "Specific hours only"
- Agent waits.

**Scenario 3.2: User picks "Specific hours only"**
- Agent says: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)"
- Input: free text.
- Agent waits.
- Agent parses schedule, builds deliveryDays array.

**Scenario 3.3: User picks "24/7 delivery"**
- Agent builds deliveryDays array (all days allowed, UTC). No extra prompt.

#### Step 3.2: Delivery Type

**Scenario 3.4: Delivery type choice**
- Agent says: "How would you like your leads delivered?\n\n• Webhook – sends lead data via HTTP POST\n• Portal – client accesses leads via web portal\n• FTP – uploads lead files to a server\n• Email – delivers leads to an inbox"
- Card: ActionSet — "Portal" | "Webhook" | "Email" | "FTP" (button order differs from bullet order above)
- Agent waits.

**Scenario 3.5: deliveryDays already built but deliveryTypeChoice missing (re-entry)**
- Agent re-displays the delivery type prompt and card.
- Agent waits.

**Scenario 3.6: User types invalid delivery type (not Portal/Webhook/Email/FTP)**
- Agent clears deliveryTypeChoice and re-displays the delivery type prompt and card.
- Agent waits.

---

### Phase 3 — Portal Branch

**Scenario 3P.1: Create portal method**
- No user input needed.
- Agent calls `create_delivery_method` silently.
- On success: moves to Phase 4 (summary). No connection test for Portal.
- On failure after retry: "I ran into an issue creating the delivery method.\n\nPlease try again."
- Agent waits.

---

### Phase 3 — Email Branch

**Scenario 3E.1: Create email method**
- No user input needed.
- Agent calls `create_delivery_method` silently.
- On success: moves to Phase 4 (summary). No connection test for Email.
- On failure after retry: "I ran into an issue creating the delivery method.\n\nPlease try again."
- Agent waits.

---

### Phase 3 — FTP Branch

**Scenario 3F.1: Collect FTP credentials**
- Agent says: "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"
- Input: free text (three values).
- Agent waits.

**Scenario 3F.2: Create FTP method**
- Agent calls `create_delivery_method` silently.
- On success: moves to Phase 3a (FTP connection test).
- On failure after retry: "I ran into an issue creating the delivery method.\n\nPlease try again."
- Agent waits.

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
- Agent calls `create_delivery_method` (no mapping).
- On success: moves to Phase 3b (webhook test).
- On failure after retry: "I ran into an issue creating the delivery method.\n\nPlease try again." Agent waits.

**Scenario 3W.4: User picks "I'll provide instructions"**
- Agent calls `get_lead_type(leadTypeUID)` to load leadFields.
- On failure: "I ran into an issue loading the lead fields for mapping.\n\nPlease try again." Agent waits.
- On success: proceeds to content type selection.

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
- Agent says: "Please paste the {contentTypeChoice} schema that your client's API expects."
- Agent waits.

**Scenario 3W.7: User pastes a URL instead of content**
- Agent says: "I can't access external links. Please open the page and paste the posting instructions or schema text here."
- Agent waits. On re-entry, re-validates.

**Scenario 3W.8: Input too large (>5000 chars) or unformatted**
- Agent says: "This looks too large or unformatted. Would you like to simplify or skip mapping for now?"
- Card: ActionSet — "Simplify input" | "Skip mapping"
- Agent waits.

**Scenario 3W.8a: User picks "Simplify input"**
- Agent says: "Please paste a smaller, well-formatted excerpt (e.g., the request body and a short field list)."
- Agent waits. On re-entry, re-validates.

**Scenario 3W.8b: User picks "Skip mapping"**
- Sets skipFieldMapping=true. Jumps to Scenario 3W.3 (create without mapping).

**Scenario 3W.9: Auto-detect confirmation (only if "I'm not sure")**
- Agent detects format from content (JSON/XML/URL Encoded).
- Agent says: "I've detected this as {detectedFormat} format. Is this correct?"
- Card: ActionSet — "Continue with {detectedFormat}" | "Switch content type"
- Agent waits.

**Scenario 3W.9a: User picks "Continue with {detectedFormat}"**
- Sets contentTypeChoice to detected format. Proceeds to parsing.

**Scenario 3W.9b: User picks "Switch content type"**
- Clears contentTypeChoice and postingInstructions. Returns to Scenario 3W.5.

**Scenario 3W.10: JSON/XML parse failure**
- Agent first attempts auto-fix (missing braces, trailing commas, unescaped quotes, single→double quotes).
- If parsing still fails after auto-fix:
- Agent says: "I couldn't parse this as valid {contentTypeChoice}. Would you like to:\n• Fix and re-paste the {contentTypeChoice} schema\n• Switch to a different content type"
- Card: ActionSet — "Re-paste {contentTypeChoice} schema" | "Switch content type"
- Agent waits.

**Scenario 3W.10a: User picks "Re-paste"**
- Agent says: "Please paste the {contentTypeChoice} schema that your client's API expects."
- Agent waits. Returns to validation.

**Scenario 3W.10b: User picks "Switch content type"**
- Clears contentTypeChoice and postingInstructions. Returns to Scenario 3W.5.

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
- On success: moves to Phase 3b (webhook test).
- On failure after retry: "I ran into an issue creating the delivery method.\n\nPlease try again." Agent waits.

---

### Phase 3a — FTP Connection Test

**Scenario 3a.1: Offer test**
- Agent says: "Would you like to test the connection to your endpoint before continuing?"
- Card: ActionSet — "Test Connection" | "Skip"
- Agent waits.

**Scenario 3a.2: User picks "Skip"**
- Moves to Phase 4 (summary).

**Scenario 3a.3: User picks "Test Connection"**
- Agent detects protocol (FTP or SFTP from deliveryAddress).
- Agent calls `test_ftp_sftp_connection`.

**Scenario 3a.4: Test succeeds**
- Agent says: "✓ Connection test successful."
- Card: ActionSet — "Continue"
- Agent waits. On click: moves to Phase 4.

**Scenario 3a.5: Test fails**
- Agent says: "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later."
- Card: ActionSet — "Retry" | "Skip"
- Agent waits.

**Scenario 3a.5a: User picks "Retry"**
- Re-runs the test (back to Scenario 3a.3).

**Scenario 3a.5b: User picks "Skip"**
- Moves to Phase 4 (summary).

---

### Phase 3b — Webhook Connection Test

**Scenario 3b.1: Offer test**
- Agent says: "Would you like to test the connection to your endpoint before continuing?"
- Card: ActionSet — "Test Connection" | "Skip"
- Agent waits.

**Scenario 3b.2: User picks "Skip"**
- Moves to Phase 4 (summary).

**Scenario 3b.3: User picks "Test Connection"**
- If mimeContentType and requestBody known: generates testPayload with sample values and calls `test_webhook_connection` with contentType and payload.
- Otherwise: calls `test_webhook_connection` with empty payload.

**Scenario 3b.4: Test succeeds**
- Agent says: "✓ Connection test successful."
- Card: ActionSet — "Continue"
- Agent waits. On click: moves to Phase 4.

**Scenario 3b.5: Test fails**
- Agent says: "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later."
- Card: ActionSet — "Retry" | "Skip"
- Agent waits.

**Scenario 3b.5a: User picks "Retry"**
- Re-runs the test (back to Scenario 3b.3).

**Scenario 3b.5b: User picks "Skip"**
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
- Agent normalizes (e.g., "$25" → 25.00).

**Scenario 5.2: Collect exclusivity**
- Agent says: "Will this client receive exclusive or shared leads?"
- Card: ActionSet — "Exclusive" | "Shared"
- Agent waits.

**Scenario 5.3: Collect order system preference**
- Agent says: "Would you like to enable the Order System for this client?"
- Card: ActionSet — "Yes" | "No"
- Agent waits.

**Scenario 5.4: Load lead fields (silent)**
- Agent calls `get_lead_type(leadTypeUID)` to retrieve leadTypeName and leadFields.
- On failure: "I ran into an issue loading the lead type fields.\n\nPlease try again." Agent waits.

**Scenario 5.5: Detect state field (silent)**
- Agent detects state field from leadFields (specialBit → name exact → name contains).
- If confident: retains stateFieldUID silently.
- If ambiguous: prompts user to confirm which field. Card: compact field selector. Agent waits.

**Scenario 5.6: Collect target states**
- Agent says: "Which states do you want to target? (e.g., CA, AZ, TX)"
- Input: free text.
- Agent waits.
- Agent normalizes to uppercase USPS codes.

**Scenario 5.7: Create delivery account (silent)**
- Agent calls `get_usa_states`, matches target states, builds criteria payload.
- Agent calls `create_delivery_account`.
- On failure after retry: "I ran into an issue creating the delivery account.\n\nPlease try again." Agent waits.

**Scenario 5.8: Ask about additional criteria**
- Agent says: "Would you like to add additional lead criteria, or skip?"
- Card: ActionSet — "Add criteria" | "Skip"
- Agent waits.

**Scenario 5.8a: User picks "Skip"**
- Retains additionalCriteria="None". Moves to Phase 6.

**Scenario 5.8b: User picks "Add criteria"**
- Loads Phase 5c (criteria builder). leadFields stays in working memory.

---

### Phase 5c — Criteria Builder

**Scenario 5c.1: Show field suggestions**
- Agent says: "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {field1}\n• {field2}\n• ..."
- If extraFieldCount > 0, appends: "\nThere are {extraFieldCount} more fields available.\n\nYou can type a criterion directly, show more fields, or skip."
- If extraFieldCount = 0, appends: "\nYou can type a criterion directly or skip."
- Card: ActionSet — "Show more fields" | "Skip" (if extraFieldCount > 0) or just "Skip" (if extraFieldCount = 0)
- Agent waits.

**Scenario 5c.1a: User picks "Skip" from suggestions**
- Retains additionalCriteria="None". Moves to Phase 6.

**Scenario 5c.2: User picks "Show more fields"**
- Agent says: "Additional Fields (showing up to 10):\n\n• {field1}\n• {field2}\n• ..."
- Card varies by criteria state:
  - No criteria yet: ActionSet — "Show more fields" | "Skip"
  - Criteria exist: ActionSet — "Show more fields" | "Continue"
- Agent waits.

**Scenario 5c.3: User types a criterion directly**
- Agent parses natural language (field name + operator + value).
- Fuzzy-matches field name against leadFieldsMap.

**Scenario 5c.3a: No confident field match**
- Agent says: "I couldn't find that field. Please type the field name you'd like to use, say 'show fields' to see all available options, or say 'skip' to continue without additional criteria."
- Agent waits.

**Scenario 5c.3b: Enumerated field, no valid value provided**
- Agent says: "I see you want to filter by {fieldName}.\nPlease select a value:"
- Card: Input.ChoiceSet compact with enum values + Submit button.
- Agent waits.

**Scenario 5c.3c: Enumerated field, only field name mentioned**
- Agent says: "Please select a value for {fieldName}:"
- Card: Input.ChoiceSet compact with enum values + Submit button.
- Agent waits.

**Scenario 5c.3d: Enumerated field, value fuzzy-matches (>85%)**
- Agent resolves to leadFieldEnumUID silently.
- Appends criterion. Proceeds to loop prompt.

**Scenario 5c.3e: Non-enum field, valid criterion**
- Agent creates parsed criteria object silently.
- Appends criterion. Proceeds to loop prompt.

**Scenario 5c.4: Enum selection received**
- Agent creates criterion with selected enumFieldChoice.
- Clears enum state. Proceeds to loop prompt.

**Scenario 5c.5: Criteria loop prompt**
- Agent says: "Would you like to add another criterion, see more fields, or continue?"
- Card: ActionSet — "Show more fields" | "Continue" (if more extra fields remain) or just "Continue" (if no more fields).
- Agent waits.

**Scenario 5c.5a: User provides another criterion**
- Returns to Scenario 5c.3.

**Scenario 5c.5b: User picks "Show more fields"**
- Returns to Scenario 5c.2.

**Scenario 5c.5c: User picks "Continue" (or says continue/done/no)**
- Agent calls `update_delivery_account` to add parsed criteria.
- On failure after retry: "I ran into an issue updating the delivery account.\n\nPlease try again." Agent waits.
- On success: moves to Phase 6.

---

### Phase 6 — Delivery Account Summary

**Scenario 6.1: Display summary card**
- Card: Table showing Company Name, Lead Type, Delivery Method, Price per Lead, Lead Exclusivity (Exclusive/Shared), Target States, Additional Criteria, Order System (Yes/No).
- Button: "Continue" (full-setup) or "Done" (add-account).
- Agent waits.

**Scenario 6.2: Full-setup — user clicks "Continue"**
- Moves to Phase 7.

**Scenario 6.3: Add-account — user clicks "Done"**
- Agent says: "✓ Your delivery account is ready to use for {companyName}."
- Conversation ends.

---

### Phase 7 — Client Summary (full-setup only)

**Scenario 7.1: Display summary card**
- Card: Table showing Company Name, Contact Email, Client Status, Lead Type, Delivery Method (with ID), Delivery Account (with ID).
- Message: "Review your configuration above. Would you like to activate the client now?"
- Buttons: "Activate" | "Keep Inactive"
- Agent waits.

**Scenario 7.2: User clicks "Activate" or "Keep Inactive"**
- Agent retains clientSummaryChoice. Moves to Phase 8.

---

### Phase 8 — Activation (full-setup only)

**Scenario 8.1: Activate client**
- Agent calls `update_client` (clientStatus="Active", fresh password, ALL fields resent).
- On success: "✓ Setup complete. Your lead delivery system is now \"ACTIVE\" for {companyName}."
- Conversation ends.

**Scenario 8.2: Activation fails**
- Agent says: "We encountered an issue activating the client.\n\nWould you like to try again, or keep the client inactive?"
- Card: ActionSet — "Retry activation" | "Keep Inactive"
- Agent waits.

**Scenario 8.2a: User picks "Retry activation"**
- Agent re-calls `update_client` with ALL fields and fresh password.
- Same success/failure handling as Scenario 8.1/8.2.

**Scenario 8.2b: User picks "Keep Inactive" after failure**
- Proceeds to Scenario 8.3 (keep inactive message).

**Scenario 8.3: Keep inactive**
- Agent says: "✓ Setup complete. Client {companyName} has been configured but remains \"NEW\". You can activate it later when ready."
- Conversation ends.

---

## Flow 2: Add Method

### Phase 1a — Select Client

**Scenario 1a.1: clientUID already in context**
- Skips to Phase 1a.2.

**Scenario 1a.2: No clientUID — show client selector**
- Agent calls `get_clients`.
- If list empty: "I couldn't find any clients.\n\nPlease create a client before continuing." Agent waits.
- Agent says: "Which client would you like to create a delivery method for?"
- Card: Input.ChoiceSet compact (companyName as display, clientUID as value).
- Agent waits.

**Scenario 1a.3: User types instead of clicking**
- Agent fuzzy-matches. If ambiguous: "I couldn't match that selection.\n\nWhich client would you like to create a delivery method for?"
- Re-displays selector. Agent waits.

**Scenario 1a.4: Client selected**
- Agent calls `get_client(clientUID)` to load profile.
- On failure: "I ran into an issue loading that client.\n\nPlease try again." Agent waits.
- On success: retains profile fields. Moves to Phase 2.

### Then: Phase 2 → Phase 3 → Phase 3a/3b → Phase 4 (Done)

Same scenarios as Flow 1, except Phase 4 button is "Done" and conversation ends after the summary.

---

## Flow 3: Add Account

### Phase 1b — Select Client and Delivery Method

**Scenario 1b.1: clientUID already in context**
- Skips to Phase 1b.3.

**Scenario 1b.2: No clientUID — show client selector**
- Agent calls `get_clients`.
- If list empty: "I couldn't find any clients.\n\nPlease create a client before continuing." Agent waits.
- Agent says: "Which client would you like to create a delivery account for?"
- Card: Input.ChoiceSet compact.
- Agent waits.

**Scenario 1b.3: Client selected — show delivery method selector**
- Agent calls `get_client(clientUID)` to load profile.
- Agent calls `get_delivery_methods(clientUID)`.
- If list empty: "I couldn't find any delivery methods for this client.\n\nPlease create a delivery method before adding a delivery account." Agent waits.
- Agent says: "Which delivery method would you like to use?"
- Card: Input.ChoiceSet compact (deliveryMethodName as display, deliveryMethodUID as value).
- Agent waits.

**Scenario 1b.4: User types instead of clicking (client or method)**
- Same fuzzy-match + re-display pattern as 1a.3 but with account-specific prompts.

**Scenario 1b.5: Method selected**
- Retains deliveryMethodUID, deliveryMethodName, leadTypeUID. Moves to Phase 5.

### Then: Phase 5 → Phase 5c (optional) → Phase 6 (Done)

Same scenarios as Flow 1, except Phase 6 button is "Done" and conversation ends after the summary.

---

## Summary: All STOP AND YIELD Points

| # | Phase | Trigger |
|---|-------|---------|
| 1 | 1 | Waiting for companyName and/or email |
| 2 | 1 | create_client failed after retry |
| 3 | 2 | Waiting for lead type selection |
| 4 | 2 | Ambiguous typed selection, re-displaying selector |
| 5 | 3 | Waiting for schedule choice |
| 6 | 3 | Waiting for specific hours description |
| 7 | 3 | Waiting for delivery type choice |
| 8 | 3-Portal | create_delivery_method failed after retry |
| 9 | 3-Email | create_delivery_method failed after retry |
| 10 | 3-FTP | Waiting for FTP credentials |
| 11 | 3-FTP | create_delivery_method failed after retry |
| 12 | 3-Webhook | Waiting for webhook URL |
| 13 | 3-Webhook | Waiting for field mapping choice |
| 14 | 3-Webhook | get_lead_type failed |
| 15 | 3-Webhook | Waiting for content type choice |
| 16 | 3-Webhook | Waiting for posting instructions |
| 17 | 3-Webhook | URL detected in input, asking for raw text |
| 18 | 3-Webhook | Input too large, asking to simplify or skip |
| 19 | 3-Webhook | Waiting for simplified input |
| 20 | 3-Webhook | Waiting for auto-detect confirmation |
| 21 | 3-Webhook | JSON/XML parse failed, asking to re-paste or switch |
| 22 | 3-Webhook | Waiting for re-pasted schema |
| 23 | 3-Webhook | Ambiguous field mapping, asking user to select |
| 24 | 3-Webhook | No confident match, asking for clarification |
| 25 | 3-Webhook | Waiting for user to click Continue on mapping preview |
| 26 | 3-Webhook | create_delivery_method failed after retry |
| 27 | 3a | Waiting for FTP test choice |
| 28 | 3a | Test succeeded, waiting for Continue |
| 29 | 3a | Test failed, waiting for Retry or Skip |
| 30 | 3b | Waiting for webhook test choice |
| 31 | 3b | Test succeeded, waiting for Continue |
| 32 | 3b | Test failed, waiting for Retry or Skip |
| 33 | 4 | Waiting for Continue or Done |
| 34 | 5 | Waiting for price |
| 35 | 5 | Waiting for exclusivity choice |
| 36 | 5 | Waiting for order system choice |
| 37 | 5 | get_lead_type failed |
| 38 | 5 | Ambiguous state field, asking user to confirm |
| 39 | 5 | Waiting for target states |
| 40 | 5 | create_delivery_account failed after retry |
| 41 | 5 | Waiting for additional criteria choice |
| 42 | 5c | Waiting for response to field suggestions |
| 43 | 5c | Showing additional fields, waiting for response |
| 44 | 5c | No field match, asking for clarification |
| 45 | 5c | Enum field, waiting for value selection |
| 46 | 5c | Criteria loop, waiting for add/show/continue |
| 47 | 5c | update_delivery_account failed after retry |
| 48 | 6 | Waiting for Continue or Done |
| 49 | 7 | Waiting for Activate or Keep Inactive |
| 50 | 8 | Activation failed, waiting for Retry or Keep Inactive |
| 51 | 1a | Waiting for client selection |
| 52 | 1a | get_client failed |
| 53 | 1b | Waiting for client selection |
| 54 | 1b | Waiting for delivery method selection |

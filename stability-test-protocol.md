# Stability Test Protocol — Delivery Setup Agent

## Purpose

This protocol defines a single comprehensive test scenario to be executed **50 times** against each of two instruction versions:

- **Version A:** `delivery-original-stabilized/` (monolithic Phase 3, inline criteria, XML-style summaries)
- **Version B:** `delivery-rework/` (split Phase 3 router + type files, Phase 5c criteria builder, markdown summaries)

The goal is to measure **consistency and reliability** across repeated runs. Every run uses the same inputs to isolate non-deterministic LLM behavior from instruction defects.

**Expected duration per run:** 8-15 minutes (depending on model response time and connection test latency).

**What constitutes a PASS:** All 24 checkpoints marked Y. No skipped phases, no hallucinated data, no wrong sequence.

**What constitutes a FAIL:** Any checkpoint marked N. Record which checkpoint(s) failed and add notes describing the deviation.

---

## Exact Test Data

All values below are fixed for every run except the run number.

### Identifiers (vary per run)

| Field | Template | Example (Run 07) |
|-------|----------|-------------------|
| Company Name | `StabilityTest-{RR}` | `StabilityTest-07` |
| Contact Email | `stability{RR}@test.com` | `stability07@test.com` |

Where `{RR}` is the zero-padded run number (01-50).

### Phase 2 — Lead Type

Select whichever lead type appears first in the dropdown. Record the lead type name for the run so criteria field names can be verified.

### Phase 3 — Delivery Schedule

Select: **Specific hours only**

Type exactly:
```
Mon-Fri 9am-5pm PST
```

Expected parsing:
- Monday through Friday: allow=true, startTime `YYYY-01-01T09:00:00-08:00`, endTime `YYYY-01-01T17:00:00-08:00`
- Saturday and Sunday: allow=false

### Phase 3 — Delivery Type

Select: **Webhook**

### Phase 3 — Webhook URL

Type exactly:
```
https://httpbin.org/post
```

### Phase 3 — Field Mapping Choice

Select: **I'll provide instructions**

### Phase 3 — Content Type

Select: **JSON**

### Phase 3 — Posting Instructions (JSON with intentional imperfections)

Paste exactly (note: no outer braces, trailing comma, single quotes on one value --- tests lenient parsing):
```
  "lead": {
    "first_name": "John",
    "last_name": "Doe",
    "email_address": "john@example.com",
    "phone": "5551234567"
  },
  "details": {
    "loan_amount": 250000,
    "property_value": 400000,
    "property_type": 'SFR',
    "state": "CA"
  },
  "metadata": {
    "source": "web",
    "timestamp": "2025-01-01T00:00:00Z"
  },
```

This input tests:
1. **Missing outer braces** — must be auto-fixed silently
2. **Trailing comma** after last element — must be auto-fixed silently
3. **Single quotes** on `'SFR'` — must be auto-fixed silently
4. **Hierarchical/nested structure** — `lead.first_name`, `details.loan_amount` etc. must be preserved in requestBody
5. **Unmapped fields** (metadata.source, metadata.timestamp) — should be excluded from requestBody per "include only mapped fields"

### Phase 3 — Field Mapping Preview

After the AI shows the mapping table, select: **Continue**

### Phase 3b — Connection Test

Select: **Test Connection**

(httpbin.org/post will return 200 with the posted body, so the test should succeed.)

### Phase 4 — Method Summary

Select: **Continue**

### Phase 5 — Price

Type exactly:
```
$37.50
```

Expected normalization: 37.50

### Phase 5 — Exclusivity

Select: **Exclusive**

### Phase 5 — Order System

Select: **No**

### Phase 5 — Target States (natural language)

Type exactly:
```
California, Texas, Florida
```

Expected normalization: CA, TX, FL

### Phase 5 — Field Suggestions / Criteria Gate

When field suggestions are shown, examine the recommended fields list. Then:

**Criterion 1** — type exactly (do NOT select a button):
```
loan amount at least 500000
```

Expected parsing:
- Field: LoanAmount (or closest match)
- Operator: GreaterOrEqual
- Value: 500000

**Criterion 2** — when the criteria loop re-prompts, type exactly:
```
property type
```

This should be an **enumerated field**. Expected behavior:
- AI detects isEnumerated=true
- AI forces operator to "In"
- AI displays a ChoiceSet dropdown with enum values
- Select the **first option** in the dropdown

**Criterion 3** — when the criteria loop re-prompts again, type exactly:
```
credit score more than 620
```

Expected parsing:
- Field: CreditScore (or closest match like SelfCreditRating)
- Operator: Greater (or GreaterOrEqual depending on field type)
- Value: 620

**Exit criteria loop** — when the criteria loop re-prompts, type exactly:
```
done
```

This tests text-based exit (not the "Continue" button).

### Phase 5 — Criteria Gate (rework only)

If the rework version asks "Would you like to add additional lead criteria, or skip?" via an ActionSet card BEFORE showing field suggestions, select: **Add criteria**

### Phase 6 — Account Summary

Select: **Continue**

### Phase 7 — Client Summary

Select: **Activate**

### Phase 8 — Activation

Verify the success message appears.

---

## Step-by-Step Execution Script

Execute these steps in order. For each step, the table shows what to do, what should happen, and what to verify.

### Phase 1 — Create Client

**Platform action names:**
- Stabilized (Version A): **Create Single Client (Original)**
- Rework (Version B): **Create Single Client (Reworked)**

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 1 | Start the flow via the corresponding action's "Start Chat" button | AI loads the action file and Phase 1 resource | The setup prompt appears |
| 2 | Type: `StabilityTest-{RR}, stability{RR}@test.com` | AI displays exactly: "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email" THEN collects the values and calls create_client silently | **Check P1-PROMPT**: The intro prompt appears exactly ONCE (not duplicated). AI does not ask for the fields again. |
| 3 | (no action — AI proceeds) | AI calls create_client, then summarize_history, then loads Phase 2 | Client created silently. No error messages. |

### Phase 2 — Lead Type Selection

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 4 | (AI shows lead type selector) | AI calls display_lead_types_choice (stabilized) or display_lead_types_choice / get_lead_types + display_adaptive_card (rework). Shows dropdown or buttons with lead types. | **Check P2-DROPDOWN**: Lead types appear as a ChoiceSet dropdown or ActionSet buttons (NOT plain text bullets). |
| 5 | Select the first lead type from the list | AI retains leadTypeUID and leadTypeName. Calls summarize_history. Loads Phase 3. | Lead type accepted without re-prompt. |

### Phase 3 — Schedule

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 6 | (AI shows schedule choice) | AI displays: "First, let's set the delivery schedule..." with ActionSet: "24/7 delivery" \| "Specific hours only" | Two buttons appear |
| 7 | Select: **Specific hours only** | AI prompts: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)" | Plain text prompt (no card) |
| 8 | Type: `Mon-Fri 9am-5pm PST` | AI parses schedule. Builds deliveryDays array (Mon-Fri allow=true 9am-5pm PST, Sat-Sun allow=false). Then shows delivery type prompt with 4 buttons. | **Check P3-SCHEDULE**: Schedule accepted without error. AI proceeds to delivery type question. |

### Phase 3 — Delivery Type + Webhook Setup

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 9 | Select: **Webhook** | AI prompts for webhook URL (plain text) | **Check P3-WEBHOOK-URL**: Plain text prompt, no card |
| 10 | Type: `https://httpbin.org/post` | AI accepts URL. Prompts for field mapping choice with ActionSet: "I'll provide instructions" \| "Skip for now" | URL accepted |
| 11 | Select: **I'll provide instructions** | AI loads lead fields silently. Prompts for content type with ActionSet: "URL Encoded" \| "JSON" \| "XML" \| "I'm not sure" | Four content type buttons shown |
| 12 | Select: **JSON** | AI prompts: "Please paste the JSON schema that your client's API expects." | Plain text prompt |
| 13 | Paste the JSON test data (see Exact Test Data above) | AI silently fixes missing braces, trailing comma, single quotes. Extracts fields. Fuzzy-matches to lead type fields. Builds requestBody preserving hierarchy. Shows Field Mapping Preview table card. | **Check P3-JSON-LENIENT**: AI does NOT show a parse error. Does NOT ask to re-paste or switch format. |
| | | | **Check P3-MAPPING-TABLE**: Preview is rendered as an Adaptive Card Table (not plain text arrows). Shows System Field, Delivery Field, Status columns. |
| | | | **Check P3-MAPPED-COUNT**: "Successfully mapped X out of Y fields" text appears. X and Y are integers > 0. Y should be approximately 8-10 (the non-metadata fields). |
| 14 | Select: **Continue** | AI creates delivery method with contentType, mappingSettings, requestBody. Calls summarize_history. | Method created silently |

### Phase 3b — Connection Test

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 15 | (AI shows test prompt) | AI prompts: "Would you like to test the connection to your endpoint before continuing?" with ActionSet: "Test Connection" \| "Skip" | **Check P3B-TEST-ONCE**: This prompt appears exactly ONCE |
| 16 | Select: **Test Connection** | AI generates test payload from requestBody, calls test_webhook_connection to httpbin.org/post. Test should succeed (200 response). AI shows "Connection test successful." | Test executes. Success message shown. AI does NOT re-ask the test question. |
| 17 | Select: **Continue** (rework) or (AI auto-proceeds in stabilized) | AI calls summarize_history, loads Phase 4 | Proceeds to Phase 4 |

### Phase 4 — Delivery Method Summary

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 18 | (AI shows summary card) | AI displays Adaptive Card Table with: Company Name, Lead Type, Delivery Method (StabilityTest-{RR}-Webhook), Delivery Type (HttpPost), Delivery Hours (Mon-Fri 9am-5pm PST or equivalent), Field Mappings (X of Y mapped). Button: "Continue" | **Check P4-SUMMARY**: Summary card appears as Adaptive Card table (not plain text). Card appears BEFORE Phase 5 content (not after). |
| 19 | Select: **Continue** | AI calls summarize_history. Loads Phase 5. | Proceeds to Phase 5 |

### Phase 5 — Delivery Account Setup

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 20 | (AI asks for price) | AI prompts: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead." | **Check P5-PRICE-FIRST**: Price is asked FIRST. Not exclusivity, not order system, not states. |
| 21 | Type: `$37.50` | AI normalizes to 37.50. Proceeds to ask exclusivity. | Price accepted |
| 22 | (AI asks exclusivity) | AI prompts: "Will this client receive exclusive or shared leads?" with ActionSet: "Exclusive" \| "Shared" | **Check P5-EXCL-SECOND**: Exclusivity asked SECOND (after price, before order system) |
| 23 | Select: **Exclusive** | AI retains isExclusive=true. Proceeds to order system. | |
| 24 | (AI asks order system) | AI prompts: "Would you like to enable the Order System for this client?" with ActionSet: "Yes" \| "No" | **Check P5-ORDER-THIRD**: Order system asked THIRD (after exclusivity, before states) |
| 25 | Select: **No** | AI retains useOrder=false. Loads lead fields silently. Detects state field. Prompts for target states. | |
| 26 | (AI asks target states) | AI prompts: "Which states do you want to target? (e.g., CA, AZ, TX)" | **Check P5-STATES-ASKED**: Target states question appears. NOT skipped. |
| 27 | Type: `California, Texas, Florida` | AI normalizes to CA, TX, FL. Calls get_usa_states, matches, builds state criteria. | **Check P5-STATES-NORMALIZED**: AI does not reject the input. Does not ask for abbreviations. Proceeds normally. |

### Phase 5 — Criteria (Stabilized: inline; Rework: Phase 5c)

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 28 | (AI shows field suggestions) | AI displays recommended fields list (5 fields) with text like "Based on your {leadTypeName} lead type, here are the most common criteria fields..." Shows ActionSet with "Show more fields" \| "Skip" (or similar) | **Check P5-FIELD-SUGGESTIONS**: Field suggestions are shown. List contains field names relevant to the lead type. Does NOT contain contact/personal fields (FirstName, LastName, Email, Phone) or the state field. |
| | | *Rework only: If the AI asks "Would you like to add additional lead criteria, or skip?" BEFORE showing field suggestions, select "Add criteria" first, then verify suggestions appear next.* | |
| 29 | Type: `loan amount at least 500000` | AI parses: field=LoanAmount (or closest match), operator=GreaterOrEqual, value=500000. Adds to criteria. Shows criteria loop prompt: "Would you like to add another criterion, see more fields, or continue?" | **Check P5-CRITERION-1**: AI correctly identifies the field. Shows parsed criterion in plain English (e.g., "Loan Amount at least 500,000"). Loop prompt appears. |
| 30 | Type: `property type` | AI detects enumerated field. Forces operator to "In". Displays ChoiceSet dropdown with enum values from leadFieldEnums. | **Check P5-ENUM-CHOICESET**: A ChoiceSet dropdown appears (style=compact) with the actual enum values for property type. NOT plain text. NOT radio buttons. |
| 31 | Select the **first option** from the ChoiceSet dropdown | AI stores the leadFieldEnumUID as criterion value. Adds to criteria. Shows criteria loop prompt again. | |
| 32 | Type: `credit score more than 620` | AI parses: field=CreditScore (or closest numeric field), operator=Greater or GreaterOrEqual, value=620. Adds to criteria. Shows criteria loop prompt again. | **Check P5-CRITERION-3**: Third criterion parsed correctly. This verifies the criteria loop re-enters properly after the enum ChoiceSet interaction. |
| 33 | Type: `done` | AI exits the criteria loop. Joins criteria into summary string. | **Check P5-DONE-EXITS**: The word "done" exits the loop. AI does NOT treat it as a field name. AI does NOT show "I couldn't find that field." |

### Phase 5 — Account Creation (version-dependent timing)

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 34 | (AI creates/updates account) | **Stabilized:** create_delivery_account called with state criteria + all 3 additional criteria in one call. **Rework:** create_delivery_account was already called after states (with state criteria only). Now update_delivery_account is called with the 3 additional criteria. | Account created/updated without error. |

### Phase 6 — Delivery Account Summary

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 35 | (AI shows account summary card) | AI displays Adaptive Card Table with: Company Name, Lead Type, Delivery Method, Price per Lead ($37.50), Lead Exclusivity, Target States, Additional Criteria, Order System | **Check P6-BOOLEANS**: Lead Exclusivity shows "Exclusive" (NOT "true"). Order System shows "Disabled" (stabilized) or "No" (rework) (NOT "false"). |
| | | | **Check P6-SUMMARY**: Card appears as an Adaptive Card table, not plain text. Card appears BEFORE Phase 7 content. |
| 36 | Select: **Continue** | AI calls summarize_history. Loads Phase 7. | |

### Phase 7 — Client Summary

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 37 | (AI shows client summary card) | AI displays Adaptive Card Table with: Company Name, Contact Email, Client Status (New), Lead Type, Delivery Method, Delivery Account. Two buttons: "Activate" \| "Keep Inactive" | **Check P7-SUMMARY**: Summary card shown as Adaptive Card table. Contains all 6 rows. |
| 38 | Select: **Activate** | AI loads Phase 8. | |

### Phase 8 — Activation

| Step | User Action | Expected AI Behavior | Verification |
|------|-------------|----------------------|--------------|
| 39 | (AI activates client) | AI calls update_client with clientStatus="Active" and all required fields (companyName, email, username, password, timezone). Fresh password generated. | **Check P8-ACTIVATION**: Success message appears: "Setup complete. Your lead delivery system is now \"ACTIVE\" for StabilityTest-{RR}." No further prompts or suggestions. |

---

## Scoring Matrix

Copy this table for each run. Mark Y (pass) or N (fail) for each checkpoint.

### Version: _________________ (stabilized / rework)

| Run | P1-PROMPT | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 02 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 03 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 04 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 05 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 06 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 07 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 08 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 09 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 10 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 11 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 12 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 13 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 14 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 15 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 16 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 17 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 18 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 19 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 20 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 21 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 22 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 23 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 24 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 25 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 26 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 27 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 28 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 29 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 30 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 31 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 32 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 33 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 34 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 35 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 36 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 37 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 38 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 39 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 40 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 41 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 42 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 43 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 44 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 45 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 46 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 47 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 48 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 49 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 50 | | | | | | | | | | | | | | | | | | | | | | | | | |

### Column Key

| Column | Full Name | What to Check |
|--------|-----------|---------------|
| P1-PROMPT | Phase 1 prompt not duplicated | Intro prompt appears exactly once, not twice |
| P2-DROP | Phase 2 lead type dropdown works | Lead types shown as ChoiceSet/ActionSet, not plain text. Selection accepted without re-prompt. |
| P3-SCHED | Phase 3 schedule parsed correctly | "Mon-Fri 9am-5pm PST" accepted without error. AI proceeds to delivery type. |
| P3-WURL | Phase 3 webhook URL accepted | URL accepted via plain text. Mapping choice card shown next. |
| P3-JSON | Phase 3 JSON parsed leniently | Missing braces, trailing comma, and single quotes all silently fixed. No parse error shown. |
| P3-TABLE | Phase 3 field mapping preview shown as table | Mapping preview rendered as Adaptive Card Table with 3 columns (System Field, Delivery Field, Status). NOT plain text arrows. |
| P3-COUNT | Phase 3 mapped count correct | "Successfully mapped X out of Y fields" shown. X > 0, Y > 0. |
| P3B-TEST | Phase 3b connection test shown once | "Would you like to test the connection?" prompt appears exactly once. After success, no re-prompt. |
| P4-SUMM | Phase 4 summary card in correct position | Summary table card displayed. Appears BEFORE any Phase 5 content. |
| P5-PRICE | Phase 5 price asked first | Price prompt is the first question in Phase 5. |
| P5-EXCL | Phase 5 exclusivity asked second | Exclusivity prompt comes after price and before order system. |
| P5-ORDER | Phase 5 order system asked third | Order system prompt comes after exclusivity and before target states. |
| P5-STATE | Phase 5 target states asked | Target states question appears. NOT skipped. NOT hallucinated. |
| P5-NORM | Phase 5 states normalized correctly | "California, Texas, Florida" accepted. AI does not reject or ask for abbreviations. |
| P5-FIELD | Phase 5 field suggestions shown | Recommended fields list shown with actual field names. Does not contain personal/contact fields. |
| P5-CR1 | Phase 5 criterion 1 parsed correctly | "loan amount at least 500000" mapped to correct field with GreaterOrEqual operator. |
| P5-ENUM | Phase 5 enum criterion shows ChoiceSet | "property type" triggers a ChoiceSet dropdown with enum values. Not plain text. |
| P5-CR3 | Phase 5 criterion 3 loop re-enters | After enum ChoiceSet selection, criteria loop re-prompts. Third criterion accepted and parsed. |
| P5-DONE | Phase 5 "done" exits loop | Typing "done" exits the criteria loop. Not treated as a field name. |
| P6-BOOL | Phase 6 booleans formatted | Exclusivity shows "Exclusive"/"Shared" (not true/false). Order System shows human-readable label (not true/false). |
| P6-SUMM | Phase 6 summary card in correct position | Summary table card displayed. Appears BEFORE any Phase 7 content. |
| P7-SUMM | Phase 7 summary card shown | Client setup summary table card with Activate/Keep Inactive buttons. |
| P8-ACT | Phase 8 activation succeeds | Success message shown. No errors. Conversation ends cleanly. |
| RESULT | Overall pass/fail | PASS if all 24 checks are Y. FAIL if any is N. |

---

## Aggregate Summary Table

After completing all 50 runs for a version, fill in this summary.

| Metric | Stabilized (Version A) | Rework (Version B) |
|--------|------------------------|---------------------|
| Total runs | 50 | 50 |
| Full PASS count | | |
| Full PASS rate (%) | | |
| P1-PROMPT fail count | | |
| P2-DROP fail count | | |
| P3-SCHED fail count | | |
| P3-WURL fail count | | |
| P3-JSON fail count | | |
| P3-TABLE fail count | | |
| P3-COUNT fail count | | |
| P3B-TEST fail count | | |
| P4-SUMM fail count | | |
| P5-PRICE fail count | | |
| P5-EXCL fail count | | |
| P5-ORDER fail count | | |
| P5-STATE fail count | | |
| P5-NORM fail count | | |
| P5-FIELD fail count | | |
| P5-CR1 fail count | | |
| P5-ENUM fail count | | |
| P5-CR3 fail count | | |
| P5-DONE fail count | | |
| P6-BOOL fail count | | |
| P6-SUMM fail count | | |
| P7-SUMM fail count | | |
| P8-ACT fail count | | |
| Most common failure | | |
| Avg duration (min) | | |

---

## Known Differences Between Versions

These are intentional structural differences. When evaluating results, do NOT count these as failures unless the version-specific expected behavior itself fails.

### Phase 2 — Lead Type Selection

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| Tool | `display_lead_types_choice` (single tool call) | `display_lead_types_choice` (same tool, same behavior) |
| Behavior | Identical | Identical |

### Phase 3 — Delivery Method Architecture

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| File structure | Single monolithic `phase-3-create-delivery-method.md` (182 lines) handles schedule, type selection, and ALL delivery type branches (Portal, Webhook, Email, FTP) with IF blocks | Router file `rw-phase-3-create-delivery-method.md` collects schedule + type, then routes to separate files: `rw-phase-3-webhook.md`, `rw-phase-3-email.md`, `rw-phase-3-ftp.md`, `rw-phase-3-portal.md` |
| Phase 3 resource loads | 1 resource load | 2 resource loads (router + type-specific) |
| Known risk | F12: AI may stall after URL input due to monolithic file length | Separate files reduce context per resource load |

### Phase 3b — Connection Test

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| After success | Auto-proceeds to summarize (no Continue button) | Shows "Continue" button, waits for user to proceed |
| Test step count | Steps 15-16 (2 interactions) | Steps 15-17 (3 interactions: test prompt, test result, Continue) |

### Phase 5 — Account Creation and Criteria

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| Account creation timing | Account created ONCE at the end of Phase 5, after ALL criteria (states + additional) are collected | Account created EARLY in Phase 5 with state criteria only. Additional criteria added later via `update_delivery_account` in Phase 5c. |
| Tool calls | 1x `create_delivery_account` (with everything) | 1x `create_delivery_account` (states only) + 1x `update_delivery_account` (additional criteria) |
| Criteria builder location | Inline in Phase 5 | Separate resource `rw-phase-5c-criteria-builder.md` |
| Criteria gate | Field suggestions shown immediately after states. User types criteria or selects "Skip". | Separate criteria gate: "Would you like to add additional lead criteria, or skip?" ActionSet shown first. If "Add criteria", loads Phase 5c which shows field suggestions. |
| Exit behavior | "Continue"/"done"/"no" exits loop | GLOBAL EXIT RULE: "skip"/"done"/"continue"/"none"/"no" exits at ANY yield point in Phase 5c |

### Phase 6 — Boolean Display

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| isExclusive display | "Exclusive" / "Shared" | "Exclusive" / "Shared" (same) |
| useOrder display | "Enabled" / "Disabled" | "Yes" / "No" |

### Phase 7 — Client Summary

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| Delivery Method row | Shows `{deliveryMethodName} (ID: {deliveryMethodUID})` | Shows `{deliveryMethodName}` (no ID) |
| Delivery Account row | Shows `{companyName}-Account (ID: {deliveryAccountUID})` | Shows `{companyName}-Account` (no ID) |

### Phase 8 — Activation Failure

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| Failure options | "Retry activation" only (no escape) | "Retry activation" \| "Keep Inactive" |

### Summarization Format

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| Format | XML-style: `<summary><completed>...</completed><current_state>...</current_state><next_instructions>...</next_instructions></summary>` | Markdown-style: `# Phase N Complete`, `# Current System State` with `* key: value` bullets, `# Next Instructions` |

### Instruction Format

| Aspect | Stabilized | Rework |
|--------|------------|--------|
| Instruction format | Linear imperative with STEP markers: STEP 1, STEP 2, etc. Uses PROMPT, ASK, WAIT, TOOL, RETAIN notation. | Flat steps within states: Step 1, Step 2, etc. Uses prose ("Call the tool", "Prompt the user exactly as follows"). STOP AND YIELD directive at each user interaction. |
| Phase 5 structure | Single flat sequence (STEP 1-11) with inline tool calls and criteria loop | Single flat state (Steps 1-10) with inline tool calls. Criteria builder in separate Phase 5c resource. |
| Known risk | Steps sometimes run out of order (less often due to simpler notation) | LLM may skip non-interactive steps (tool calls) between user prompts |

---

## Tester Notes

1. **Lead type dependency:** The specific criteria fields available (and whether "property type" is enumerated, whether "credit score" exists) depend on the lead type selected in Phase 2. If the selected lead type does not have these fields, adapt the criteria inputs to match available fields and note the substitution.

2. **httpbin.org availability:** If httpbin.org is down, the connection test will fail. This is expected. Mark P3B-TEST as Y if the test prompt appeared once and the failure was handled gracefully (Retry/Skip shown). The checkpoint tests prompt behavior, not endpoint availability.

3. **Enum field availability:** If the selected lead type has no enumerated fields, the P5-ENUM checkpoint cannot be tested. Mark it as N/A and note in the Notes column. Pick a lead type that has enumerated fields when possible.

4. **Timing:** Start a timer when you begin step 1. Stop when the activation message appears (step 39). Record duration in the Notes column.

5. **Rework criteria gate:** The rework version has an extra interaction before field suggestions: "Would you like to add additional lead criteria, or skip?" If this appears, select "Add criteria" and note that it appeared. This is expected rework behavior, not a failure.

6. **Screenshots:** For any FAIL, capture a screenshot of the failure point. Name it `{version}-run{RR}-{checkpoint}.png` (e.g., `rework-run07-P5-ENUM.png`).

---

## Part 2: Edge Case Test Battery (50 runs per version)

After completing Part 1 (50 standard runs), execute these edge case scenarios 50 times each. Use the same run numbering (EC-01 through EC-50).

### Edge Case Scenario — Content Type Mismatch + Complex Posting Instructions

This scenario tests guardrails around content type detection, parse failure recovery, hierarchical posting instructions, and criteria processing.

### Exact Test Data

#### Phase 1-2: Same as Part 1
Use `EdgeCase-{RR}, edgecase{RR}@test.com`. Select first lead type.

#### Phase 3 — Schedule + Delivery Type
Select: **24/7 delivery** (fast path)
Select: **Webhook**

#### Phase 3 — Webhook URL
Type: `https://httpbin.org/post`

#### Phase 3 — Field Mapping Choice
Select: **I'll provide instructions**

#### Phase 3 — Content Type: SELECT XML (intentional mismatch)
Select: **XML**

#### Phase 3 — Paste JSON as XML (content type mismatch test)
Paste exactly (this is valid JSON, NOT XML — tests mismatch detection):
```
{
  "lead": {
    "contact": {
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "primary_phone": "2065551234"
    },
    "address": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zip": "98101"
    },
    "loan_details": {
      "loan_amount": 250000,
      "property_value": 420000,
      "loan_type": ["Refinance"],
      "credit_rating": "Excellent"
    }
  }
}
```

**Expected behavior:**
1. AI detects JSON pasted as XML → "I couldn't parse this as valid XML"
2. Shows ActionSet: "Re-paste XML schema" | "Switch content type"
3. Select: **Switch content type**
4. Content type choice re-appears
5. Select: **JSON**
6. AI prompts for JSON schema
7. Paste the SAME JSON again
8. AI parses successfully, shows mapping preview

#### Phase 3 — After Successful Parse
Continue through mapping preview → connection test (Skip) → Phase 4 summary

#### Phase 5 — Full Account Setup (same as Part 1)
Price: `$50`
Exclusivity: **Shared**
Order System: **Yes**
Target States: `New York, California, Arizona` (tests 3-state normalization: NY, CA, AZ)

#### Phase 5 — Complex Criteria Sequence

**Criterion 1** (enumerated field — test ChoiceSet):
```
property type
```
Select first enum value from dropdown.

**Criterion 2** (range/between operator):
```
loan amount between 100000 and 500000
```

**Criterion 3** (invalid field name — test error recovery):
```
favorite color blue
```
Expected: "I couldn't find that field" error message with recovery options.
Then type: `skip` to test GLOBAL EXIT (rework) or "done" (stabilized).

**OR if field is found**, type `done` to exit loop.

#### Phase 6-8
Continue → Activate

### Edge Case Checkpoints

| Column | Full Name | What to Check |
|--------|-----------|---------------|
| EC-MISMATCH | Content type mismatch detected | AI shows "I couldn't parse as valid XML" (NOT hangs, NOT auto-fixes to JSON) |
| EC-SWITCH | Switch content type works | Content type choice re-appears after "Switch content type" |
| EC-REPARSE | JSON re-parse succeeds | Same JSON parses successfully when correct content type (JSON) is selected |
| EC-HIERARCHY | Nested JSON structure preserved | Mapping preview shows fields from nested paths (lead.contact.first_name, lead.address.state) |
| EC-3STATES | 3-state normalization | "New York, California, Arizona" normalized to NY, CA, AZ (all 3 matched) |
| EC-ENUM | Enum criterion ChoiceSet | "property type" triggers ChoiceSet dropdown with enum values |
| EC-BETWEEN | Between operator parsed | "loan amount between 100000 and 500000" parsed with Between operator |
| EC-NOTFOUND | Invalid field handled gracefully | "favorite color blue" → error message with recovery, NOT treated as valid field |
| EC-EXIT | Exit from criteria works | "skip"/"done" exits the criteria loop cleanly |
| EC-COMPLETE | Flow completes end-to-end | Activation succeeds without errors |

### Edge Case Scoring Matrix

### Version: _________________ (stabilized / rework)

| Run | EC-MISMATCH | EC-SWITCH | EC-REPARSE | EC-HIERARCHY | EC-3STATES | EC-ENUM | EC-BETWEEN | EC-NOTFOUND | EC-EXIT | EC-COMPLETE | Notes |
|-----|-------------|-----------|------------|--------------|------------|---------|------------|-------------|---------|-------------|-------|
| EC-01 | | | | | | | | | | | |
| EC-02 | | | | | | | | | | | |
| EC-03 | | | | | | | | | | | |
| EC-04 | | | | | | | | | | | |
| EC-05 | | | | | | | | | | | |

(Continue to EC-50)

### Edge Case Aggregate Summary

| Metric | Stabilized | Rework |
|--------|-----------|--------|
| Total runs | 50 | 50 |
| EC-MISMATCH pass rate | | |
| EC-SWITCH pass rate | | |
| EC-REPARSE pass rate | | |
| EC-HIERARCHY pass rate | | |
| EC-3STATES pass rate | | |
| EC-ENUM pass rate | | |
| EC-BETWEEN pass rate | | |
| EC-NOTFOUND pass rate | | |
| EC-EXIT pass rate | | |
| EC-COMPLETE pass rate | | |
| Full PASS rate (all 10 checks) | | |

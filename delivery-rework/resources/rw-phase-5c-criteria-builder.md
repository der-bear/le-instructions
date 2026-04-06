═══════════════════════════════════════
CURRENT PHASE: Phase 5c — Criteria Builder
Phase 5 instructions above are complete. Execute ONLY the below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to collect additional criteria from the user, then create the delivery account with ALL criteria (state + additional) in a single call.

Variables carried from Phase 5 (do NOT re-collect — retrieve from conversation above):
clientUID, deliveryMethodUID, companyName, price, isExclusive, useOrder, criteriaPayload, targetStates, stateFieldUID, leadFields, leadTypeName

CRITICAL: Always APPEND to parsedCriteriaList, never overwrite.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Show Field Suggestions**
Execute these steps in order:

1. Build field suggestion lists from leadFields:
   - Exclude contact/personal-information fields
   - Exclude the state field (leadFieldUID = stateFieldUID)
   - Prioritize top relevant industry-specific lead qualification business criteria
   Build two data structures:
   - leadFieldsIndex: lightweight index of ALL fields from leadFields — store only {leadFieldName, leadFieldUID, leadFieldDataType, isEnumerated, leadFieldSpecialBit} per field. Do NOT include leadFieldEnums in this index.
   - leadFieldsMap: detailed map for ONLY the 15 fields in suggestedFields + extraFields — include full metadata including leadFieldEnums.
   When the user names a field not in leadFieldsMap, look it up in leadFieldsIndex. If found, load its full metadata (including leadFieldEnums if enumerated) from the leadFields array.
   Retain: suggestedFields (first 5), extraFields (next 10), extraFieldCount (total remaining count), leadFieldsIndex, leadFieldsMap, parsedCriteriaList = [], criteriaSummaryList = [].

2. Prompt the user exactly as follows: "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {list suggestedFields}\n{if extraFieldCount > 0: "\nThere are " + extraFieldCount + " more fields available.\n\nYou can type a criterion directly, show more fields, or skip." else: "\nYou can type a criterion directly or skip."}"
   Present using display_adaptive_card: IF extraFieldCount > 0: ActionSet "Show more fields" | "Skip". ELSE: ActionSet "Skip".
   **STOP AND YIELD.** Do not hallucinate data.
   - IF "Skip" OR user says "skip"/"done"/"none"/"no": set additionalCriteria = "None", proceed to State 3 (create account with state-only criteria).
   - IF "Show more fields": show more fields, re-present with same buttons. **STOP AND YIELD.** Loop back to this handler.
   - IF user typed a criterion: parse using State 2 rules below.

**State 2: Handle Criteria Input**

This state handles ALL user responses: typed criteria, show-more-fields, enum selections, and the criteria loop prompt. Apply the Criteria Parsing Rules below when parsing any criterion.

FIRST — classify user input:
- IF input matches a card button action ("Show more fields", "Skip", "Done adding criteria", "Add another criterion"): handle as the corresponding button action, NOT as a criterion.
- ONLY if input does not match any button action: attempt to parse as a criterion.

Handle user response:

- IF "Show more fields" OR asked to see more:
  Prompt: "Additional Fields (showing up to 10):\n\n• {list extraFields}"
  Present using display_adaptive_card: IF criteriaSummaryList empty: ActionSet "Show more fields" | "Skip". IF not empty: ActionSet "Show more fields" | "Done adding criteria".
  **STOP AND YIELD.** Loop back to this handler on next response.

- IF user typed a criterion:
  Parse using Criteria Parsing Rules below.
  IF no confident field match: Prompt: "I couldn't find that field. Please type the field name you'd like to use, say 'show fields' to see all available options, or say 'skip' to continue without additional criteria." **STOP AND YIELD.** Do not hallucinate data.
  IF enumerated field:
    Force operator to "In" (or "NotIn" if user intent suggests exclusion).
    ALWAYS show ChoiceSet for value selection — do not auto-resolve from user text.
    IF user provided a value attempt: Prompt "I see you want to filter by {fieldName}.\nPlease select a value:"
    IF user only mentioned field name: Prompt "Please select a value for {fieldName}:"
    Display ChoiceSet via display_adaptive_card: Input.ChoiceSet (style=compact, placeholder="Select a value", do NOT pre-select any value, choices from leadFieldEnums where each choice has value set to the leadFieldEnumUID as a string and title set to the enum display name) + Action.Submit.
    **STOP AND YIELD.** On selection, use the submitted leadFieldEnumUID as the string value in parsedCriteria.value.
  IF non-enumerated field with valid operator and value:
    Create parsed criteria object. Append to parsedCriteriaList. Create plain-English summary, append to criteriaSummaryList.
    Show criteria loop prompt (below).

- IF user selected an enum value from ChoiceSet:
  Create parsed criteria object with leadFieldUID, type="FieldValue", operator from the retained enum operator context (default "In"), value=selected leadFieldEnumUID (as string).
  Append to parsedCriteriaList. Create plain-English summary, append to criteriaSummaryList.
  Show criteria loop prompt (below).

Criteria loop prompt — MANDATORY after every accepted criterion:
  You MUST show this prompt after every criterion is appended. Do NOT advance to State 3 without user confirmation.
  Prompt: "Would you like to add another criterion, see more fields, or continue?"
  Present using display_adaptive_card: ActionSet "Add another criterion" | "Show more fields" | "Done adding criteria" (omit "Show more fields" if no more extra fields remain).
  **STOP AND YIELD.** Do not hallucinate data.
  - IF "Add another criterion" or user typed a criterion → parse as criterion above.
  - IF "Show more fields" → handle as show-more above.
  - IF "Done adding criteria" or user says "done"/"continue"/"no" → State 3.

### Criteria Parsing Rules

1. Parse operator from user input. Check symbols FIRST, then natural language:
   Symbols: >= or ≥ → GreaterOrEqual, <= or ≤ → LessOrEqual, > → Greater, < → Less, = or == → Equal, != or <> → NotEqual
   Natural language: minimum/at least/no less than/or more → GreaterOrEqual, at most/maximum/no more than/up to/or less → LessOrEqual, exactly/equals → Equal, more than/above/over → Greater, less than/below/under → Less, not equal/is not → NotEqual, between → Between, contains → Contains, not/exclude → NotIn
2. Match field name against leadFieldsMap or leadFieldsIndex (fuzzy >90%). Look up leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit.
3. Extract values (comma/or-separated). Auto-correct casing on enum matches.
4. Validate operator against field type:
   **PRIORITY 1 — Special flags override dataType:**
   - isEnumerated=true: ONLY "In", "NotIn"
   - leadFieldSpecialBit='State'/'StandardState': ONLY "In", "NotIn"
   - leadFieldSpecialBit='Zip': "Equal", "NotEqual", "In", "NotIn", "Distance_Compare"
   - leadFieldSpecialBit='PrimaryPhone'/'MobilePhone': "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "In", "NotIn"
   **PRIORITY 2 — By leadFieldDataType:**
   - Int, BigInt, Decimal, Float, Money: "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "In", "NotIn"
   - DateTime: "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "DateCompare"
   - Varchar: "Equal", "NotEqual", "Contains", "DoesNotContain", "In", "NotIn"
   - Bit: "Equal", "NotEqual"
   If parsed operator invalid for field type, use the FIRST valid operator from the list.
5. Enumerated fields: MUST use "In"/"NotIn" only (default to "In"). Single-select only. Validate user values against leadFieldEnums (case-insensitive). The selected leadFieldEnumUID is stored as a string in parsedCriteria.value.
6. Parsed criteria format: `{leadFieldUID: <int>, type: "FieldValue", operator: <string>, value: <string>}`. For "Between" operator, serialize value as pipe-delimited "min|max" (e.g., "100000|500000").

**State 3: Create Account with All Criteria**

1. Build additionalCriteria: if parsedCriteriaList is not empty, join criteriaSummaryList with "; ". Otherwise "None".
2. Build finalCriteriaPayload: start with criteriaPayload (from Phase 5, contains state criterion), then append all items from parsedCriteriaList.
   CRITICAL: finalCriteriaPayload must contain BOTH the state criterion AND all additional criteria.
3. Call create_delivery_account with:
   `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={finalCriteriaPayload}}`
   CRITICAL: createDeliveryAccountDto must be passed as a native object, NOT a JSON string.
   You MUST call create_delivery_account. deliveryAccountUID does not exist yet.
   If the tool fails, repair and retry once. If still fails: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.**
4. Verify deliveryAccountUID > 0.
5. Retain: deliveryAccountUID, price, targetStates, additionalCriteria, isExclusive, useOrder.
6. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 5c Complete — Delivery Account Created with Criteria

# Current System State
* Flow Intent: {flowIntent}
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: {clientStatus}
* Time Zone Name: {timeZoneName}
* Time Offset: {timeOffset}
* Lead Type UID: {leadTypeUID}
* Lead Type Name: {leadTypeName}
* Delivery Method UID: {deliveryMethodUID}
* Delivery Method Name: {deliveryMethodName}
* Delivery Type: {deliveryType}
* Delivery Address: {deliveryAddress}
* FTP User: {ftpUser}
* FTP Password: {ftpPassword}
* MIME Content Type: {mimeContentType}
* Request Body: {requestBody}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: {mappedCount}
* Total Count: {totalCount}
* Connection Test Mode: {connectionTestMode}
* Delivery Account UID: {deliveryAccountUID}
* Price: {price}
* Target States: {targetStates} (USPS abbreviations)
* Additional Criteria: {additionalCriteria}
* Is Exclusive: {isExclusive}
* Use Order: {useOrder}

# Next Instructions
→ Load and execute Phase 6 at mcp://resource/rw-phase-6-delivery-account-summary
```

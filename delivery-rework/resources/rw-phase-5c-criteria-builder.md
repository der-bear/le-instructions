═══════════════════════════════════════
CURRENT PHASE: Phase 5c — Criteria Builder
Phase 5 instructions above are complete. Execute ONLY the below.
═══════════════════════════════════════

Your objective is to collect additional criteria from the user and update the existing delivery account. The account already exists (deliveryAccountUID is known) — use `update_delivery_account` to add criteria.

CRITICAL: Every criterion the user adds must be preserved. Do NOT overwrite parsedCriteriaList — always APPEND to it.

GLOBAL EXIT RULE: If at any YIELD point the user says "skip", "done", "continue", "none", or "no" instead of providing a criterion or selecting a value:
- IF criteriaSummaryList is not empty → proceed to State 5 (update account with collected criteria, then summarize).
- IF criteriaSummaryList is empty → set additionalCriteriaChoice = "Skip", load mcp://resource/rw-phase-5-create-delivery-account — Phase 5 will handle the skip and summarize.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Build and Show Field Suggestions (Do this first)**
* IF suggestedFields is missing:
  1. Build field suggestion lists from leadFields:
     - Exclude contact/personal-information fields
     - Exclude the state field (leadFieldUID = stateFieldUID)
     - Prioritize relevant business-qualification fields
  2. Retain:
     - suggestedFields = first 5 leadFieldName values
     - extraFields = next 10 leadFieldName values
     - extraFieldCount = total remaining field count
     - leadFieldsMap = {leadFieldName → {leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit}}
     - parsedCriteriaList = [] (empty array)
     - criteriaSummaryList = [] (empty array)
  3. CRITICAL: You MUST display the field suggestions prompt below. Do NOT skip this step.
  4. Prompt the user exactly as follows: "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {list suggestedFields}\n{if extraFieldCount > 0: "\nThere are " + extraFieldCount + " more fields available.\n\nYou can type a criterion directly, show more fields, or skip." else: "\nYou can type a criterion directly or skip."}"
  5. Present the choice using display_adaptive_card:
     - IF extraFieldCount > 0: ActionSet with "Show more fields" | "Skip"
     - ELSE: ActionSet with "Skip"
  6. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

**State 2: Parse Typed Criterion or Show More Fields**

When parsing a criterion, follow these rules:
1. Parse natural language to operator keywords: minimum/at least → GreaterOrEqual, exactly → Equal, more than → Greater, less than → Less, between → Between, contains → Contains, etc.
2. Match the field name against leadFieldsMap (fuzzy match >90% confidence).
3. Look up matched field metadata: leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit.
4. Validate the operator against the field type:
   **PRIORITY 1 — Special flags override dataType:**
   - IF isEnumerated = true: ONLY "In", "NotIn"
   - IF leadFieldSpecialBit = 'State' OR 'StandardState': ONLY "In", "NotIn"
   - IF leadFieldSpecialBit = 'Zip': "Equal", "NotEqual", "In", "NotIn", "Distance_Compare"
   - IF leadFieldSpecialBit = 'PrimaryPhone' OR 'MobilePhone': "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "In", "NotIn"
   **PRIORITY 2 — By leadFieldDataType:**
   - Int, BigInt, Decimal, Float, Money: "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "In", "NotIn"
   - DateTime: "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "DateCompare"
   - Varchar: "Equal", "NotEqual", "Contains", "DoesNotContain", "In", "NotIn"
   - Bit: "Equal", "NotEqual"
   If the parsed operator is NOT in the valid list, replace it with the FIRST valid operator from the list.
5. Enumerated field handling:
   - MUST use "In" or "NotIn" only. If another operator was parsed, set it to "In".
   - Single-select only (no multi-select).
   - If user provided values: validate against leadFieldEnums (fuzzy >85%, case-insensitive). If no match, show ChoiceSet.
   - ChoiceSet: Input.ChoiceSet with style=compact, placeholder="Select a value". Choices from leadFieldEnums with value=leadFieldEnumUID, title=value. Include Action.Submit.
   - The selected leadFieldEnumUID is stored as a string in parsedCriteria.value.
6. Parsed criteria object format: `{leadFieldUID: <integer>, type: "FieldValue", operator: <string>, value: <string>}`

* IF the user selected "Show more fields" OR asked to see more fields:
  1. Prompt the user exactly as follows: "Additional Fields (showing up to 10):\n\n• {list extraFields}"
  2. Present the choice using display_adaptive_card:
     - IF criteriaSummaryList is empty: ActionSet with "Show more fields" | "Skip"
     - IF criteriaSummaryList is not empty: ActionSet with "Show more fields" | "Continue"
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF the user provided a criterion directly:
  1. Parse the criterion using the rules above.
  2. IF no field matches confidently: prompt exactly: "I couldn't find that field. Please type the field name you'd like to use, say 'show fields' to see all available options, or say 'skip' to continue without additional criteria." **STOP AND YIELD.** Do not hallucinate data.
  3. IF the matched field is enumerated AND the user did not provide a valid enum value:
     - Force the operator to "In" or "NotIn" only. If anything else, set to "In".
     - Retain enumFieldName and enumFieldOperator.
     - Prompt exactly: "I see you want to filter by {fieldName}.\nPlease select a value:"
     - Display the ChoiceSet selector.
     - **STOP AND YIELD.** Do not hallucinate data. Proceed to State 3 when the user selects a value.
  4. IF the matched field is enumerated AND the user mentioned only the field name:
     - Retain enumFieldName, set enumFieldOperator="In".
     - Prompt exactly: "Please select a value for {fieldName}:"
     - Display the ChoiceSet selector.
     - **STOP AND YIELD.** Do not hallucinate data. Proceed to State 3 when the user selects a value.
  5. ELSE IF the matched field is enumerated AND the user provided a value that fuzzy-matches (>85%):
     - Force operator to "In" or "NotIn" only.
     - Resolve to leadFieldEnumUID. Use this UID as the value.
     - Create parsed criteria object. Append to parsedCriteriaList.
     - Create plain-English summary. Append to criteriaSummaryList.
     - Proceed to State 4.
  6. ELSE (non-enum field):
     - Create parsed criteria object. Append to parsedCriteriaList.
     - Create plain-English summary. Append to criteriaSummaryList.
     - Proceed to State 4.

**State 3: Accept Enum Selection**
* IF enumFieldName is known AND the user has selected a value (enumFieldChoice):
  1. Create parsed criteria object: leadFieldUID from matched field, type="FieldValue", operator={enumFieldOperator}, value={enumFieldChoice} (the leadFieldEnumUID as string).
  2. Append to parsedCriteriaList.
  3. Create plain-English summary. Append to criteriaSummaryList.
  4. Clear: enumFieldName, enumFieldOperator, enumFieldChoice.
  5. Proceed to State 4.

**State 4: Criteria Loop Prompt**
* IF criteriaSummaryList is not empty AND the user has not yet said "continue" or "done":
  1. Prompt the user exactly as follows: "Would you like to add another criterion, see more fields, or continue?"
  2. Present the choice using display_adaptive_card:
     - IF more extra fields remain: ActionSet with "Show more fields" | "Continue"
     - ELSE: ActionSet with "Continue"
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - IF user provides another criterion: go back to State 2 parsing logic.
  - IF user selects "Show more fields": go back to State 2 show-fields logic.
  - IF user selects "Continue" OR says "continue", "done", or "no": proceed to State 5.

**State 5: Update Account with Additional Criteria and Summarize**
* IF the user chose to continue:
  1. Build additionalCriteria: join criteriaSummaryList entries with "; ".
  2. Call the update_delivery_account tool to add the parsed criteria to the existing account:
     `deliveryAccountUID={deliveryAccountUID}, criteria={parsedCriteriaList}`
  3. If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue updating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  4. Retain: additionalCriteria.
  5. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 5c Complete — Criteria Added

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
* Target States: {targetStates}
* Additional Criteria: {additionalCriteria}
* Is Exclusive: {isExclusive}
* Use Order: {useOrder}

# Next Instructions
→ Load and execute Phase 6 at mcp://resource/rw-phase-6-delivery-account-summary
```

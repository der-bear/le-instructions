═══════════════════════════════════════
CURRENT PHASE: Phase 5c — Criteria Builder
Phase 5 instructions above are complete. Execute ONLY the below.
═══════════════════════════════════════

Your objective is to collect additional criteria from the user and update the existing delivery account. The account already exists (deliveryAccountUID is known) — use `update_delivery_account` to add criteria.

CRITICAL: Always APPEND to parsedCriteriaList, never overwrite.

GLOBAL EXIT RULE: At any YIELD, if user says "skip"/"done"/"continue"/"none"/"no":
- If criteriaSummaryList not empty → State 3.
- If empty → set additionalCriteriaChoice="Skip", return to Phase 5.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Show Field Suggestions**
Execute these steps in order:

1. Build field suggestion lists from leadFields:
   - Exclude contact/personal-information fields
   - Exclude the state field (leadFieldUID = stateFieldUID)
   - Prioritize top relevant industry-specific lead qualification business criteria
   Retain: suggestedFields (first 5), extraFields (next 10), extraFieldCount (total remaining), leadFieldsMap = {leadFieldName → {leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit}}, parsedCriteriaList = [], criteriaSummaryList = [].

2. Prompt the user exactly as follows: "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {list suggestedFields}\n{if extraFieldCount > 0: "\nThere are " + extraFieldCount + " more fields available.\n\nYou can type a criterion directly, show more fields, or skip." else: "\nYou can type a criterion directly or skip."}"
   Present using display_adaptive_card: IF extraFieldCount > 0: ActionSet "Show more fields" | "Skip". ELSE: ActionSet "Skip".
   **STOP AND YIELD.** Do not hallucinate data.

**State 2: Handle Criteria Input**

This state handles ALL user responses: typed criteria, show-more-fields, enum selections, and the criteria loop prompt. Apply the Criteria Parsing Rules below when parsing any criterion.

Handle user response:

- IF "Show more fields" OR asked to see more:
  Prompt: "Additional Fields (showing up to 10):\n\n• {list extraFields}"
  Present using display_adaptive_card: IF criteriaSummaryList empty: ActionSet "Show more fields" | "Skip". IF not empty: ActionSet "Show more fields" | "Continue".
  **STOP AND YIELD.** Loop back to this handler on next response.

- IF user typed a criterion:
  Parse using Criteria Parsing Rules below.
  IF no confident field match: Prompt: "I couldn't find that field. Please type the field name you'd like to use, say 'show fields' to see all available options, or say 'skip' to continue without additional criteria." **STOP AND YIELD.** Do not hallucinate data.
  IF enumerated field AND no valid value provided or only field name mentioned:
    Force operator to "In". Prompt: "Please select a value for {fieldName}:" (or "I see you want to filter by {fieldName}.\nPlease select a value:" if user attempted a value). Display ChoiceSet via display_adaptive_card: Input.ChoiceSet (style=compact, placeholder="Select a value", choices from leadFieldEnums with value=leadFieldEnumUID, title=value) + Action.Submit. **STOP AND YIELD.** On selection, use selected leadFieldEnumUID as string value.
  IF enumerated field AND user value fuzzy-matches enum (>85%): resolve to leadFieldEnumUID, force operator to "In"/"NotIn".
  Create parsed criteria object. Append to parsedCriteriaList. Create plain-English summary, append to criteriaSummaryList.
  Show criteria loop prompt (below).

- IF user selected an enum value from ChoiceSet:
  Create parsed criteria object with leadFieldUID, type="FieldValue", operator from retained enumFieldOperator, value=selected leadFieldEnumUID (as string).
  Append to parsedCriteriaList. Create plain-English summary, append to criteriaSummaryList.
  Show criteria loop prompt (below).

Criteria loop prompt (after each criterion added):
  Prompt: "Would you like to add another criterion, see more fields, or continue?"
  Present using display_adaptive_card: IF more extra fields remain: ActionSet "Show more fields" | "Continue". ELSE: ActionSet "Continue".
  **STOP AND YIELD.** Do not hallucinate data.
  - IF another criterion → handle as "user typed a criterion" above.
  - IF "Show more fields" → handle as show-more above.
  - IF "Continue"/"done"/"no" → State 3.

### Criteria Parsing Rules

1. Parse natural language to operator keywords: minimum/at least → GreaterOrEqual, exactly → Equal, more than → Greater, less than → Less, between → Between, contains → Contains.
2. Match field name against leadFieldsMap (fuzzy >90%). Look up leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit.
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
5. Enumerated fields: MUST use "In"/"NotIn" only (default to "In"). Single-select only. Validate user values against leadFieldEnums (fuzzy >85%, case-insensitive). The selected leadFieldEnumUID is stored as a string in parsedCriteria.value.
6. Parsed criteria format: `{leadFieldUID: <int>, type: "FieldValue", operator: <string>, value: <string>}`. For "Between" operator, serialize value as pipe-delimited "min|max" (e.g., "100000|500000").

**State 3: Update Account with Additional Criteria**

1. Build additionalCriteria: join criteriaSummaryList with "; ".
2. Call update_delivery_account: `deliveryAccountUID={deliveryAccountUID}, criteria={parsedCriteriaList}`
   CRITICAL: Send the FULL parsedCriteriaList array, not just the latest criterion.
   If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue updating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
3. Retain: additionalCriteria.
4. Immediately call the summarize_history tool.

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

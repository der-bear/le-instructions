# Phase 5: Criteria Builder

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 5 criteria-builder resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to handle the additional-criteria loop (show more fields, parse criteria, handle enum selections), then build the final criteria payload, create the delivery account, and hand off to Phase 6.

CRITICAL: Every criterion the user adds must be preserved in the final payload. Do NOT overwrite parsedCriteriaList — always APPEND to it.

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
  4. Prompt the user exactly as follows: "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {list suggestedFields}\n{if extraFieldCount > 0: \"\nThere are \" + extraFieldCount + \" more fields available.\n\nWould you like to add criteria, see more fields, or skip?\" else: \"\nWould you like to add criteria or skip?\"}"
  5. Present the choice using display_adaptive_card:
     - IF extraFieldCount > 0: ActionSet with "Show more fields" | "Skip"
     - ELSE: ActionSet with "Skip"
  6. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - IF the user selects "Skip" from the suggestions card:
    1. Retain additionalCriteria = "None".
    2. Call the get_usa_states tool. Match the normalized targetStates to the returned USA-state list (match abbreviation, exact match, case-insensitive). Collect the corresponding stateUID values.
    3. Create stateUIDArray. Serialize as pipe-delimited string: stateUIDArray.join('|').
    4. Build criteriaPayload with only the state criterion:
       `[{leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: "<pipe-delimited stateUID string>"}]`
    5. Call the create_delivery_account tool with these defaults:
       `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
    6. If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
    7. Retain: deliveryAccountUID, price, targetStates, additionalCriteria, isExclusive, useOrder.
    8. Immediately call the summarize_history tool (use the same Summarization Requirements at the end of this file).

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
   - If user provided values: validate against leadFieldEnums (fuzzy >85%, case-insensitive). Auto-correct casing if match found. If no match or low confidence, show the ChoiceSet selector.
   - ChoiceSet: Input.ChoiceSet with style=compact, placeholder="Select a value". Choices from leadFieldEnums with value=leadFieldEnumUID, title=value. Include Action.Submit.
   - The selected leadFieldEnumUID is stored as a string in parsedCriteria.value.
6. Parsed criteria object format: `{leadFieldUID: <integer>, type: "FieldValue", operator: <string>, value: <string>}`

* IF the user selected "Show more fields" OR asked to see more fields:
  1. Prompt the user exactly as follows: "Additional Fields (showing up to 10):\n\n• {list extraFields}"
  2. Present the choice using display_adaptive_card:
     - IF criteriaSummaryList is empty AND more extra fields remain: ActionSet with "Show more fields" | "Skip"
     - IF criteriaSummaryList is empty AND no more extra fields: ActionSet with "Skip"
     - IF criteriaSummaryList is not empty AND more extra fields remain: ActionSet with "Show more fields" | "Continue"
     - IF criteriaSummaryList is not empty AND no more extra fields: ActionSet with "Continue"
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF the user provided a criterion directly:
  1. Parse the criterion using the Criteria Parsing Rules above.
  2. IF no field matches confidently: prompt exactly: "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options." **STOP AND YIELD.** Do not hallucinate data.
  3. IF the matched field is enumerated AND the user did not provide a valid enum value:
     - Force the operator to "In" or "NotIn" only. If the parsed operator is anything else, set it to "In".
     - Retain enumFieldName and enumFieldOperator (now corrected).
     - Prompt exactly: "I see you want to filter by {fieldName}.\nPlease select a value:"
     - Display the ChoiceSet selector per the enum rules above.
     - **STOP AND YIELD.** Do not hallucinate data. Proceed to State 3 when the user selects a value.
  4. IF the matched field is enumerated AND the user mentioned only the field name (no operator or value):
     - Retain enumFieldName, set enumFieldOperator="In".
     - Prompt exactly: "Please select a value for {fieldName}:"
     - Display the ChoiceSet selector per the enum rules above.
     - **STOP AND YIELD.** Do not hallucinate data. Proceed to State 3 when the user selects a value.
  5. ELSE IF the matched field is enumerated AND the user provided a value that fuzzy-matches an enum entry (>85%, case-insensitive):
     - Force the operator to "In" or "NotIn" only. If the parsed operator is anything else, set it to "In".
     - Resolve the matched enum entry's leadFieldEnumUID. Use this UID (as a string) for the criteria value — do NOT use the raw user text.
     - Create the parsed criteria object with value=leadFieldEnumUID.
     - Append it to parsedCriteriaList.
     - Create a plain-English summary (translate operator to friendly text, e.g., GreaterOrEqual → "at least").
     - Append it to criteriaSummaryList.
     - Proceed to State 4.
  6. ELSE (non-enum field):
     - Create the parsed criteria object.
     - Append it to parsedCriteriaList.
     - Create a plain-English summary (translate operator to friendly text, e.g., GreaterOrEqual → "at least").
     - Append it to criteriaSummaryList.
     - Proceed to State 4.

**State 3: Accept Enum Selection**
* IF enumFieldName is known AND the user has selected a value (enumFieldChoice):
  1. Create the parsed criteria object: leadFieldUID from the matched field, type="FieldValue", operator={enumFieldOperator}, value={enumFieldChoice} (this is the leadFieldEnumUID as a string).
  2. Append it to parsedCriteriaList.
  3. Create a plain-English summary. Append to criteriaSummaryList.
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

**State 5: Build Payload, Create Account, and Summarize**
* IF the user chose to continue AND deliveryAccountUID is missing:
  1. Build additionalCriteria: join all criteriaSummaryList entries with "; " if criteria exist, otherwise "None".
  2. Call the get_usa_states tool. Match the normalized targetStates to the returned USA-state list (match abbreviation, exact match, case-insensitive). Collect the corresponding stateUID values.
  3. Create stateUIDArray by collecting all matched stateUID values. Serialize as pipe-delimited string: stateUIDArray.join('|').
  4. Build criteriaPayload array:
     - FIRST element: the state criterion: `{leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: "<pipe-delimited stateUID string>"}`
     - REMAINING elements: every item from parsedCriteriaList, in order.
  5. Call the create_delivery_account tool with these defaults:
     `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
  6. If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  7. Retain: deliveryAccountUID, price, targetStates, additionalCriteria, isExclusive, useOrder.
  8. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
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

# Conversation Anchor
* DELIVERY_SETUP_START

# Next Instructions
Fetch and execute instructions from: mcp://resource/rw-phase-6-delivery-account-summary
```

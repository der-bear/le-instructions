═══════════════════════════════════════
<current_phase>Phase 5 — Create Delivery Account</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

 PROMPT: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead."
 ASK [conversational]: price
 WAIT for user input

 PROMPT: "Will this client receive exclusive or shared leads?"
 ASK [adaptive_card]: ActionSet (Exclusive | Shared)
 WAIT for user choice

 IF "Exclusive":
   isExclusive = true
 ELSE:
   isExclusive = false

 PROMPT: "Would you like to enable the Order System for this client?"
 ASK [adaptive_card]: ActionSet (Yes | No)
 WAIT for user choice

 IF "Yes":
   useOrder = true
 ELSE:
   useOrder = false

 TOOL: get_lead_type(leadTypeUID) → data.leadTypeName as leadTypeName, data.leadFields as leadFields
 RETAIN: leadTypeName, leadFields

 PROCESS (Silent - State Detection):
   - Detect state field from leadFields priority:
       1) leadFieldSpecialBit in {'State','StandardState'}
       2) leadFieldName='state' (case-insensitive)
       3) leadFieldName contains 'state'
     Do not process lower tiers once matched.
     Semantically validate selected field represents US state; if confidence <5%, confirm correct field with user.
     Remember as stateFieldUID.

 PROMPT: "Which states do you want to target? (e.g., CA, AZ, TX)"
 ASK [conversational]: targetStates
 WAIT for user input
 
 CRITICAL: Field suggestion steps are MANDATORY. You MUST execute all steps below before proceeding to Build Criteria Array. Do NOT skip field suggestions.

 PROCESS (Silent - Build Field Suggestions):
   - Build field suggestion lists:
       - Exclude contact/personal information lead fields
       - Exclude state field where leadFieldUID = stateFieldUID (already collected)
       - Prioritize top relevant industry-specific lead qualification business criteria
       - RETAIN:
           suggestedFields = first 5 leadFieldName (or all if fewer)
           extraFields = next 10 leadFieldName (or fewer if not available)
           extraFieldCount = total remaining field count
           leadFieldsMap = {leadFieldName → {leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit}}

 CRITICAL: MUST display the field suggestions prompt below. Do NOT skip this step.
 PROMPT (MANDATORY): "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {list suggestedFields}\n{if extraFieldCount > 0: "\nThere are " + extraFieldCount + " more fields available.\n\nWould you like to add criteria, see more fields, or skip?" else: "\nWould you like to add criteria or skip?"}"
 ASK [adaptive_card]: ActionSet (Show more fields | Skip) if extraFieldCount > 0, else ActionSet (Skip)
 WAIT for user choice

 PROCESS (Handle User Response):
   - If user types criterion directly (e.g., "credit rating excellent") → parse it via Criteria Parsing, APPEND result to criteriaPayload array, add display entry to criteriaList, go to Criteria Loop
   - If user selects "Show more fields" OR asks to see more AND extraFields available:
       PROMPT: "Additional Fields (showing up to 10):\n\n• {list extraFields}"
       SUGGEST [adaptive_card]: ActionSet (Show more fields | Skip)
       WAIT for user choice, loop back to Handle User Response
   - If user selects "Skip" OR says "none"/"skip"/"no" → RETAIN additionalCriteria = "None", GO TO Build Criteria Array

 CRITICAL: The Criteria Loop MUST repeat until the user EXPLICITLY selects "Continue" or says "continue"/"done"/"no". After each criterion is added, always re-enter the loop to ask for more. Do NOT exit the loop automatically after adding a criterion.

 PROCESS (Criteria Loop):
   - PROMPT: "Would you like to add another criterion, see more fields, or continue?"
   - SUGGEST [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
   - WAIT for user input
   - If provides new criterion directly (typed) → parse it via Criteria Parsing, APPEND result to criteriaPayload array, add display entry to criteriaList → LOOP BACK to start of Criteria Loop (ask again)
   - If selects "Show more fields" OR asks to see more AND extraFields available:
       PROMPT: "Additional Fields (showing up to 10):\n• {list extraFields}"
       SUGGEST [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
       LOOP BACK to start of Criteria Loop
   - ONLY IF selects "Continue" OR says "continue"/"done"/"no" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array

 PROCESS (Criteria Parsing):
   - Parse natural language to operator keywords (minimum/at least→GreaterOrEqual, exactly→Equal, etc.)
   - Match field leadFieldName (fuzzy >90%)
   - Lookup matched field metadata from leadFields: leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit
   - Extract values (comma/or-separated, normalize per <data_normalization>)
   - Validate operator compatibility (check in priority order):
       PRIORITY 1 - Special flags override dataType:
         - IF isEnumerated=true: ONLY "In", "NotIn"
         - IF leadFieldSpecialBit='State' OR 'StandardState': ONLY "In", "NotIn"
         - IF leadFieldSpecialBit='Zip': "Equal", "NotEqual", "In", "NotIn", "Distance_Compare"
         - IF leadFieldSpecialBit='PrimaryPhone' OR 'MobilePhone': "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "In", "NotIn"
       PRIORITY 2 - By leadFieldDataType:
         - Int, BigInt, Decimal, Float, Money: "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "In", "NotIn"
         - DateTime: "Equal", "NotEqual", "Greater", "Less", "GreaterOrEqual", "LessOrEqual", "Between", "DateCompare"
         - Varchar: "Equal", "NotEqual", "Contains", "DoesNotContain", "In", "NotIn"
         - Bit: "Equal", "NotEqual"
       - If parsed operator is not in the valid list for this field type, replace it with the first valid operator from the list

   IF field matched AND isEnumerated = true:
     - Enumerated fields MUST use "In" or "NotIn" operators only
     - Use single-select dropdown (no multi-select support in current MCP schema)
     - Check the parsed operator value:
         IF operator is "In" OR operator is "NotIn": keep as-is
         ELSE IF user intent suggests exclusion (not, except, exclude): set operator to "NotIn"
         ELSE: set operator to "In"
     IF values provided by user:
       - Validate extracted values against leadFieldEnums (fuzzy >85%, case-insensitive)
       - Auto-correct casing if all match
       - IF no match OR low confidence:
           PROMPT: "I see you want to filter by {fieldName}.\nPlease select a value:"
           DISPLAY [adaptive_card]: Input.ChoiceSet with leadFieldEnums + Action.Submit
             - style: compact
             - placeholder: "Select a value"
           WAIT for selection

     IF no values provided (user only mentioned field name):
       PROMPT: "Please select a value for {fieldName}:"
       DISPLAY [adaptive_card]: Input.ChoiceSet with leadFieldEnums + Action.Submit
         - style: compact
         - placeholder: "Select a value"
         - choices: populate from leadFieldEnums array with value=leadFieldEnumUID, title=value
       WAIT for selection

     - Take the leadFieldEnumUID value returned from the ChoiceSet selection
     - Store this value as a string for use in parsedCriteria.value field

   IF field not matched:
     PROMPT: "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options."

   - Create parsedCriteria object with these fields:
       leadFieldUID: (integer from matched field)
       type: "FieldValue"
       operator: (string value like "In", "Equal", "Between", etc.)
       value: (string - single enumUID for enums, direct value for scalar fields)
   - Create criteriaList entry for display: translate operator to plain English (GreaterOrEqual→"at least")
   - APPEND this parsedCriteria to criteriaPayload array (do NOT overwrite previous entries)
   - Add display entry to criteriaList array for summary display

 PROCESS (Build Criteria Array):
     NOTE: State fields allow multiple values via conversational input. Enumerated fields use single-select only.

     - Normalize targetStates to uppercase USPS codes (per <data_normalization>)
     - TOOL: get_usa_states() → returns array [{stateUID, abbr, name}, ...]
     - Filter returned array: match abbr against targetStates (exact match, case-insensitive)
     - Extract stateUID from each matched state object
     - Create stateUIDArray by collecting all extracted stateUID values
     - Serialize stateUIDArray as pipe-delimited string using array.join('|')
     - Create state criterion object: {leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: (pipe-delimited stateUID string)}
     - Insert state criterion as the FIRST element of criteriaPayload array (before any additional criteria already in the array)
     - RETAIN criteriaPayload

 TOOL: create_delivery_account → data as deliveryAccountUID
 TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}
 CRITICAL: createDeliveryAccountDto must be passed as an object, NOT a JSON string
 RETAIN: deliveryAccountUID, price, targetStates, additionalCriteria, isExclusive, useOrder

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 5 — Delivery Account Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}, deliveryAccountUID={deliveryAccountUID}, price={price}, targetStates={targetStates}, additionalCriteria={additionalCriteria}, isExclusive={isExclusive}, useOrder={useOrder}</current_state><next_instructions>Load and execute Phase 6 from mcp://resource/phase-6-delivery-account-summary</next_instructions></summary>"

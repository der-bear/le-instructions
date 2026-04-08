═══════════════════════════════════════
<current_phase>Phase 5b — Criteria Builder</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 Variables available from accumulated prior summaries (do NOT re-collect):
   From Phase 2 summary: leadTypeUID, leadTypeName
   From Phase 5 summary: deliveryAccountUID, stateFieldUID, targetStates, price, isExclusive, useOrder

 NOTE: The delivery account already exists (deliveryAccountUID was returned by create_delivery_account in Phase 5 with the state criterion already persisted). This phase collects ADDITIONAL criteria and appends them to the existing account via update_delivery_account. Do NOT rebuild the state criterion. Do NOT call create_delivery_account.

 TOOL: get_lead_type(leadTypeUID) → data.leadFields as leadFields
 RETAIN: leadFields
 NOTE: leadFields is re-fetched here rather than carried through a summary (avoids serializing a structured array into the summary template).

 CRITICAL: Field suggestion steps are MANDATORY. You MUST execute all steps below before proceeding to Append Criteria. Do NOT skip field suggestions.

 PROCESS: Initialize additionalCriteriaArray = [] (empty array) and criteriaList = [] (empty array). These arrays accumulate criteria across the entire criteria loop. Do NOT reset them at any point.

 PROCESS (Build Field Suggestions):
   - Build field suggestion lists:
       - Exclude contact/personal-information fields (names, emails, phones, addresses, SSN, date of birth, etc.)
       - Exclude tracking/system fields (IDs, timestamps, assignment dates)
       - Exclude state field where leadFieldUID = stateFieldUID (already persisted on the account)
       - Prioritize the most relevant industry-specific criteria for lead qualification and segmentation
       - RETAIN:
           suggestedFields = first 5 leadFieldName (or all if fewer)
           extraFields = next 10 leadFieldName (or fewer if not available)
           extraFieldCount = total remaining field count
           leadFieldsMap = {leadFieldName → {leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit}}

 CRITICAL: MUST display the field suggestions prompt below. Do NOT skip this step.

 PROMPT (MANDATORY): "Based on your {leadTypeName} lead type, here are the most common criteria fields:\n\nRecommended Fields:\n\n• {list suggestedFields}\n{if extraFieldCount > 0: \"\nThere are \" + extraFieldCount + \" more fields available.\n\nYou can type a criterion directly, see more fields, or skip.\" else: \"\nYou can type a criterion directly or skip.\"}"
 ASK [adaptive_card]: ActionSet (Show more fields | Skip) if extraFieldCount > 0, else ActionSet (Skip)
 WAIT for user choice

 PROCESS (Handle User Response):
   - If user types text containing a field name or qualifier (e.g., "credit rating excellent") → parse it via Criteria Parsing (which appends to additionalCriteriaArray and adds a display entry to criteriaList), then go to Criteria Loop
   - If user selects "Show more fields" OR asks to see more AND extraFields available:
       PROMPT: "Additional Fields (showing up to 10):\n\n• {list extraFields}"
       ASK [adaptive_card]: ActionSet (Show more fields | Skip)
       WAIT for user choice, loop back to Handle User Response
   - If user selects "Skip" OR says "none"/"skip"/"no"/"done" → RETAIN additionalCriteria = "None" → GO TO Append Criteria

 CRITICAL: The Criteria Loop MUST repeat until the user EXPLICITLY selects "Continue" or says "continue"/"done"/"no". After each criterion is added, always re-enter the loop to ask for more. Do NOT exit the loop automatically after adding a criterion.
 
 PROCESS (Criteria Loop - after first criterion added):
   - PROMPT: "Would you like to add another criterion, see more fields, or continue?"
   - ASK [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
   - WAIT for user input
   - Check for exit intent BEFORE parsing as criterion: if user selects "Continue" OR input matches (case-insensitive) "continue", "done", "no", "no more", "that's all", or "finish" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → RETAIN additionalCriteriaArray → GO TO Append Criteria
   - If selects "Show more fields" OR asks to see more AND extraFields available:
       PROMPT: "Additional Fields (showing up to 10):\n• {list extraFields}"
       ASK [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
       WAIT for user choice
       Loop back to Criteria Loop
   - Otherwise, if user provides new criterion directly (typed) → parse it via Criteria Parsing (which appends to additionalCriteriaArray and adds a display entry to criteriaList), LOOP BACK to Criteria Loop

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
   - APPEND this parsedCriteria to additionalCriteriaArray (do NOT overwrite previous entries)
   - Add this entry to criteriaList array for summary display

 PROCESS (Append Criteria):
     NOTE: The state criterion is already persisted on the delivery account from Phase 5. Do NOT include it here — update_delivery_account APPENDS to the existing criteria list.
     CRITICAL: additionalCriteriaArray contains ONLY the criteria collected in this phase (no state criterion).

     IF additionalCriteriaArray is empty:
       - RETAIN: additionalCriteria = "None"
       - Skip the update_delivery_account call and proceed directly to summarize_history.

     IF additionalCriteriaArray is non-empty:
       TOOL: update_delivery_account
       TOOL_DEFAULTS: deliveryAccountUID={deliveryAccountUID}, criteria={additionalCriteriaArray}
       CRITICAL: criteria must be passed as a native array of criterion objects, NOT a JSON string
       RETAIN: additionalCriteria (the "; "-joined display string built from criteriaList above)

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 5b — Additional Criteria Appended</completed><current_state>additionalCriteria={additionalCriteria}</current_state><next_instructions>Load and execute Phase 6 from mcp://resource/split-phase-6-delivery-account-summary</next_instructions></summary>"

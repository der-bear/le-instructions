═══════════════════════════════════════
<current_phase>Phase 5 — Create Delivery Account</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════
 
 PROMPT: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead."
 ASK [conversational]: price
 WAIT for user input
 RETAIN: price

 PROMPT: "Will this client receive exclusive or shared leads?"
 ASK [adaptive_card]: ActionSet (Exclusive | Shared)
 WAIT for user choice

 IF "Exclusive":
   RETAIN: isExclusive = true
 ELSE:
   RETAIN: isExclusive = false

 PROMPT: "Would you like to enable the Order System for this client?"
 ASK [adaptive_card]: ActionSet (Yes | No)
 WAIT for user choice

 IF "Yes":
   RETAIN: useOrder = true
 ELSE:
   RETAIN: useOrder = false

 PROMPT: "Which states do you want to target? (e.g., CA, AZ, TX)"
 ASK [conversational]: targetStates - required
 WAIT for user input.
 RETAIN: targetStates

 TOOL: get_lead_type(leadTypeUID) → data.leadTypeName as leadTypeName, data.leadFields as leadFields
 RETAIN: leadTypeName, leadFields

 PROCESS (State Detection):
   - Detect state field from leadFields priority:
       1) leadFieldSpecialBit in {'State','StandardState'}
       2) leadFieldName contains 'state'
     Do not process lower tiers once matched.
     If confidence <5%, confirm correct field with user.
     RETAIN: stateFieldUID

 CRITICAL: Display the prompt below and STOP. Do NOT read or execute anything below this line until the user replies with their target states.

 PROCESS (Build State Criterion):
     - Normalize targetStates to uppercase USPS codes (per <data_normalization>)
     - TOOL: get_usa_states() → returns array [{stateUID, abbr, name}, ...]
     - Filter returned array: match abbr against targetStates (exact match, case-insensitive)
     - Extract stateUID from each matched state object
     - Create stateUIDArray by collecting all extracted stateUID values
     - Serialize stateUIDArray as pipe-delimited string using array.join('|')
     - Create state criterion object: {leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: (pipe-delimited stateUID string)}
     - Set criteriaPayload = [state criterion]
     - RETAIN criteriaPayload

 TOOL: create_delivery_account → data as deliveryAccountUID
 TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}
 CRITICAL: createDeliveryAccountDto must be passed as an object, NOT a JSON string
 RETAIN: deliveryAccountUID

 CRITICAL: Display the card below and STOP. Do NOT auto-select. Do NOT execute either IF block below until the user responds.
 PROMPT: "Would you like to add additional lead criteria, or continue with state targeting only?"
 ASK [adaptive_card]: ActionSet (Add criteria | Continue with state targeting only)
 WAIT for user choice. Do NOT execute the IF blocks below until the user selects.

 IF "Add criteria":
   TOOL: summarize_history - mandatory
   TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 5 — Delivery Account Created, Adding Criteria</completed><current_state>deliveryAccountUID={deliveryAccountUID}, stateFieldUID={stateFieldUID}, targetStates={targetStates}, price={price}, isExclusive={isExclusive}, useOrder={useOrder}</current_state><next_instructions>Load and execute Phase 5b from mcp://resource/split-phase-5b-criteria-builder</next_instructions></summary>"

 IF "Continue with state targeting only":
   RETAIN: additionalCriteria = "None"
   TOOL: summarize_history - mandatory
   TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 5 — Delivery Account Created</completed><current_state>deliveryAccountUID={deliveryAccountUID}, price={price}, targetStates={targetStates}, additionalCriteria={additionalCriteria}, isExclusive={isExclusive}, useOrder={useOrder}</current_state><next_instructions>Load and execute Phase 6 from mcp://resource/split-phase-6-delivery-account-summary</next_instructions></summary>"

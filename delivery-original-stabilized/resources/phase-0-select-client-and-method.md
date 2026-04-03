═══════════════════════════════════════
<current_phase>Phase 0b — Select Client & Method</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

 IF clientUID is defined:
   USE clientUID from context
 ELSE:
   TOOL: get_clients → data as clientsList (clientUID, companyName, clientStatus)
   IF clientsList is empty:
     PROMPT: "No clients found. Please create a client first."
     END
   PROMPT: "Which client would you like to create a delivery account for?"
   ASK [adaptive_card]: companyName (clientUID) (Input.ChoiceSet with style=compact)
   WAIT for user choice
   RETAIN: clientUID

 TOOL: get_client(clientUID) → data.companyName as companyName, data.email as email, data.clientStatus as clientStatus, data.timeZoneName as timeZoneName, data.timeOffset as timeOffset
 RETAIN: companyName, email, clientStatus, timeZoneName, timeOffset

 TOOL: get_delivery_methods(clientUID) → data as deliveryMethodsList (deliveryMethodUID, data.name as deliveryMethodName, data.leadTypeUID as leadTypeUID)
 IF deliveryMethodsList is empty:
   PROMPT: "No delivery methods found for this client. Please create a delivery method first."
   END
 PROMPT: "Which delivery method would you like to use?"
 ASK [adaptive_card]: deliveryMethodName (deliveryMethodUID) (Input.ChoiceSet with style=compact)
 WAIT for user choice
 RETAIN: deliveryMethodUID, deliveryMethodName, leadTypeUID

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 0b — Client & Method Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, leadTypeUID={leadTypeUID}</current_state><next_instructions>Load and execute Phase 5 from mcp://resource/phase-5-create-delivery-account</next_instructions></summary>"

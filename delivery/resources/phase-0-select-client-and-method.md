# RESOURCE - PHASE 0b:

<phase_0_select_client_and_method>
 ANCHOR: anchor_client_and_method_selection_start
 IF clientUID is defined:
   USE clientUID from context
 ELSE:
   TOOL: get_clients → data as clientsList (clientUID, companyName, clientStatus)
   PROMPT: "Which client would you like to create a delivery account for?"
   ASK [adaptive_card]: companyName (clientUID) (Input.ChoiceSet with style=compact)
   WAIT for user choice
   RETAIN: clientUID

 TOOL: get_client(clientUID) → data.companyName as companyName, data.email as email, data.clientStatus as clientStatus, data.timeZoneName as timeZoneName, data.timeOffset as timeOffset
 RETAIN: companyName, email, clientStatus, timeZoneName, timeOffset

 TOOL: get_delivery_methods(clientUID) → data as deliveryMethodsList (deliveryMethodUID, data.name as deliveryMethodName, data.leadTypeUID as leadTypeUID)
 PROMPT: "Which delivery method would you like to use?"
 ASK [adaptive_card]: deliveryMethodName (deliveryMethodUID) (Input.ChoiceSet with style=compact)
 WAIT for user choice
 RETAIN: deliveryMethodUID, deliveryMethodName, leadTypeUID

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="anchor_client_and_method_selection_start", summarization_text="<summary><retain>clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, leadTypeUID={leadTypeUID}</retain><next_phase>mcp://resource/phase-5-create-delivery-account</next_phase></summary>"
</phase_0_select_client_and_method>
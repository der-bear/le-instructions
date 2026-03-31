# RESOURCE - PHASE 0a:

<phase_0_select_client>
 ANCHOR: anchor_client_selection_start
 IF clientUID is defined:
   USE clientUID from context
 ELSE:
   TOOL: get_clients → data as clientsList (clientUID, companyName, clientStatus)
   PROMPT: "Which client would you like to create a delivery method for?"
   ASK [adaptive_card]: companyName (clientUID) (Input.ChoiceSet with style=compact)
   WAIT for user choice
   RETAIN: clientUID

 TOOL: get_client(clientUID) → data.companyName as companyName, data.email as email, data.clientStatus as clientStatus, data.timeZoneName as timeZoneName, data.timeOffset as timeOffset
 RETAIN: companyName, email, clientStatus, timeZoneName, timeOffset

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="anchor_client_selection_start", summarization_text="<summary><retain>clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}</retain><next_phase>mcp://resource/phase-2-get-lead-types</next_phase></summary>"
</phase_0_select_client>
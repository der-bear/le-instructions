═══════════════════════════════════════
<current_phase>Phase 0a — Select Client</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

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
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 0a — Client Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}</current_state><next_instructions>Load and execute Phase 2 from mcp://resource/phase-2-get-lead-types</next_instructions></summary>"
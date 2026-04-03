═══════════════════════════════════════
<current_phase>Phase 2 — Lead Type Selection</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

 TOOL: get_lead_types → data as leadTypesList (leadTypeName, leadTypeUID)
 PROMPT: "Please select a Lead Type for this client."
 ASK [adaptive_card]: leadTypeName (leadTypeUID) (ActionSet if≤4, Input.ChoiceSet with style=compact if>4)
 WAIT for user choice
 RETAIN: leadTypeUID, leadTypeName

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 2 — Lead Type Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}</current_state><next_instructions>Load and execute Phase 3 from mcp://resource/phase-3-create-delivery-method</next_instructions></summary>"

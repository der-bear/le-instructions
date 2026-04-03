═══════════════════════════════════════
<current_phase>Phase 2 — Lead Type Selection</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "Please select a Lead Type for this client."
 TOOL: display_lead_types_choice
 WAIT for user choice
 RETAIN: leadTypeUID, leadTypeName

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 2 — Lead Type Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}</current_state><next_instructions>Load and execute Phase 3 from mcp://resource/phase-3-create-delivery-method</next_instructions></summary>"

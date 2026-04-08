═══════════════════════════════════════
<current_phase>Phase 2 — Lead Type Selection</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "Please select a Lead Type for this client."
 TOOL: display_lead_types_choice
 WAIT for user choice
 RETAIN: leadTypeUID, leadTypeName

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 2 — Lead Type Selected</completed><current_state>leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}</current_state><next_instructions>Load and execute Phase 3 from mcp://resource/split-phase-3-create-delivery-method</next_instructions></summary>"

═══════════════════════════════════════
<current_phase>Phase 1 — Create Client</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email"
 ASK [conversational]: companyName, email
 WAIT for user input
 
 TOOL: create_client → data as clientUID
 TOOL_DEFAULTS: createClientDto={companyName={companyName}, email={email}, clientStatus="New", clientAutomationType="Price", username={email}, password={generate-password}, timeZoneName="Pacific Standard Time", timeOffset=-8}
 CRITICAL: createClientDto must be passed as an object, NOT a JSON string
 RETAIN: clientUID, companyName, email, clientStatus="New", timeZoneName="Pacific Standard Time", timeOffset=-8
 
 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 1 — Client Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus=New, timeZoneName={timeZoneName}, timeOffset={timeOffset}</current_state><next_instructions>Load and execute Phase 2 from mcp://resource/split-phase-2-get-lead-types</next_instructions></summary>"

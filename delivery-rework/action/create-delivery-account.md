# INITIAL PROMPT - PHASE 0

<phase_0_flow_entry>
 ANCHOR: anchor_flow_entry_start
 IF clientUID set in user context:
   RETAIN: clientUID

 IF flowIntent NOT set:
   PROCESS (Silent - Intent Detection):
     - "create client" OR "setup delivery" → flowIntent="full-setup"
     - "create delivery method" OR "setup delivery method" → flowIntent="add-method"
     - "create delivery account" → flowIntent="add-account"
   IF unclear:
     PROMPT: "I can help you with:\n\n1. Setting up a new client (full setup)\n2. Adding a delivery method to an existing client\n3. Adding a delivery account using an existing method\n\nWhich would you like to do?"
     WAIT for clarification
   RETAIN: flowIntent

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="anchor_flow_entry_start", summarization_text="<summary><retain>flowIntent={flowIntent}, clientUID={clientUID}</retain><next_phase>IF flowIntent='full-setup': mcp://resource/phase-1-create-client; IF flowIntent='add-method': mcp://resource/phase-0-select-client; IF flowIntent='add-account': mcp://resource/phase-0-select-client-and-method</next_phase></summary>"
</phase_0_flow_entry>
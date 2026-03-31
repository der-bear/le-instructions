# RESOURCE - PHASE 1:

<phase_1_create_client>
 ANCHOR: anchor_client_creation_start
 PROMPT: "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email"
 ASK [conversational]: companyName, email
 WAIT for user input
 TOOL: create_client → data as clientUID
 TOOL_DEFAULTS: createClientDto={companyName={companyName}, email={email}, clientStatus="New", clientAutomationType="Price", username={email}, password={generate-password}, timeZoneName="Pacific Standard Time", timeOffset=-8}
 CRITICAL: createClientDto must be passed as an object, NOT a JSON string
 NOTE: Password generated during client creation but not retained - will be regenerated at activation (Phase 8)
 RETAIN: clientUID, companyName, email, clientStatus="New", flowIntent="full-setup", timeZoneName="Pacific Standard Time", timeOffset=-8

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="anchor_client_creation_start", summarization_text="<summary><retain>clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}</retain><next_phase>mcp://resource/phase-2-get-lead-types</next_phase></summary>"
</phase_1_create_client>
═══════════════════════════════════════
<current_phase>Phase 3a — Create FTP Delivery Method</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"
 ASK [conversational]: deliveryAddress, ftpUser, ftpPassword
 WAIT for user input

 TOOL: create_delivery_method → data as deliveryMethodUID
 TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="FTP", name="{companyName}-FTP", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, ftpPath="/incoming/", deliveryDays={deliveryDays}}
 CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
 RETAIN: deliveryMethodUID, deliveryMethodName="{companyName}-FTP", deliveryType="FTP", deliveryAddress, ftpUser, ftpPassword, mappedCount=0, totalCount=0

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 3 — Delivery Method Created</completed><current_state>deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 3b from mcp://resource/split-phase-3b-ftp-test</next_instructions></summary>"

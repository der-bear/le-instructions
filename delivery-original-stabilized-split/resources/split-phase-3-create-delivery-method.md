═══════════════════════════════════════
<current_phase>Phase 3 — Create Delivery Method</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "First, let's set the delivery schedule.\n\nWould you like leads delivered 24/7, or only during specific hours?"
 ASK [adaptive_card]: ActionSet (24/7 delivery | Specific hours only)
 WAIT for user choice

 IF "Specific hours only":
   PROMPT: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)"
   ASK [conversational]: scheduleInput
   WAIT for user input
   PROCESS: Parse scheduleInput per <data_normalization> rules

   BUILD deliveryDays array with EXACTLY 7 entries (weekDay 0-6, one for each day of week):
     - User-specified days: allow=true with their startTime/endTime
     - All other days: allow=false with startTime="YYYY-01-01T00:00:00±HH:mm", endTime="YYYY-01-01T23:59:59±HH:mm"
   RETAIN: deliveryDays, deliveryScheduleDisplay

 IF "24/7 delivery":
   BUILD deliveryDays array with EXACTLY 7 entries (weekDay 0-6):
     All days: allow=true, startTime="YYYY-01-01T00:00:00+00:00", endTime="YYYY-01-01T23:59:59+00:00"
   RETAIN: deliveryDays, deliveryScheduleDisplay="24/7"

 PROMPT: "How would you like your leads delivered?\n\n• Webhook – sends lead data via HTTP POST\n• Portal – client accesses leads via web portal\n• FTP – uploads lead files to a server\n• Email – delivers leads to an inbox"
 ASK [adaptive_card]: ActionSet (Portal | Webhook | Email | FTP)
 WAIT for user choice

 PROCESS (Route to method-specific phase):
   IF "Portal":  load mcp://resource/split-phase-3a-portal
   IF "Webhook": load mcp://resource/split-phase-3a-webhook
   IF "Email":   load mcp://resource/split-phase-3a-email
   IF "FTP":     load mcp://resource/split-phase-3a-ftp

 Do NOT call summarize_history in this router. Working memory (deliveryDays, deliveryScheduleDisplay) must carry into the routed phase.
 Do NOT call create_delivery_method in this router. The routed phase owns that tool call.
 Immediately load the selected resource and execute its instructions. Do not emit any assistant message between this routing step and the loaded resource's first instruction.

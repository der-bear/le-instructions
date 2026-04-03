═══════════════════════════════════════
<current_phase>Phase 3b — Test Connection</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 IF connectionTestMode = "none":
   Go directly to summarize_history below.

 IF connectionTestMode = "ftp" OR connectionTestMode = "webhook":
   PROMPT: "Would you like to test the connection to your endpoint before continuing?"
   ASK [adaptive_card]: ActionSet (Test Connection | Skip)
   WAIT for user choice

   IF "Skip":
     Go directly to summarize_history below.

   IF "Test Connection":
     IF connectionTestMode = "ftp":
       PROCESS (Detect protocol from deliveryAddress): if starts with "sftp://" → protocol="SFTP", else → protocol="FTP"
       TOOL: test_ftp_sftp_connection
       TOOL_DEFAULTS: protocol={protocol}, host={deliveryAddress}, username={ftpUser}, password={ftpPassword}, remotePath="/incoming/"

     IF connectionTestMode = "webhook":
       TOOL: test_webhook_connection
       IF mimeContentType AND requestBody:
         PROCESS (Generate test payload): Replace [SystemFieldName] placeholders in requestBody with sample values by field type (string→"Test", email→"test@example.com", phone→"5551234567", numeric→"50.00", bool→"true", date→"2025-01-01")
         TOOL_DEFAULTS: url={deliveryAddress}, method="POST", contentType={mimeContentType}, payload={testPayload}, timeoutSeconds=30
       ELSE:
         TOOL_DEFAULTS: url={deliveryAddress}, method="POST", payload="", timeoutSeconds=30

     IF test success:
       PROMPT: "✓ Connection test successful."
       Go directly to summarize_history below. Do NOT re-ask the test question.

     IF test failure:
       PROMPT: "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later."
       ASK [adaptive_card]: ActionSet (Retry | Skip)
       WAIT for user choice
       IF "Retry":
         Loop back to the test tool call above (test_ftp_sftp_connection or test_webhook_connection). Do NOT re-ask "Would you like to test the connection".
       IF "Skip":
         Go directly to summarize_history below.

 CRITICAL: After the test completes (success or skip), call summarize_history EXACTLY ONCE and do NOT re-display the test prompt.

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 3b — Connection Tested</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/phase-4-delivery-method-summary</next_instructions></summary>"

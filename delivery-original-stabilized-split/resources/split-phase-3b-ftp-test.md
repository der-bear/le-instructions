═══════════════════════════════════════
<current_phase>Phase 3b — Test FTP Connection</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "Would you like to test the connection to your endpoint before continuing?"
 ASK [adaptive_card]: ActionSet (Test Connection | Skip)
 WAIT for user choice

 IF "Test Connection":
   PROCESS (Detect protocol from deliveryAddress): if starts with "sftp://" → protocol="SFTP", else → protocol="FTP"
   TOOL: test_ftp_sftp_connection
   TOOL_DEFAULTS: protocol={protocol}, host={deliveryAddress}, username={ftpUser}, password={ftpPassword}, remotePath="/incoming/"
   PROCESS: connectionTestSucceeded = test_ftp_sftp_connection.IsSuccess
   PROCESS: connectionTestError = test_ftp_sftp_connection.Error

   IF connectionTestSucceeded = true:
     PROMPT: "✓ Connection test successful."
     ASK [adaptive_card]: ActionSet (Continue)
     WAIT for user to click Continue THEN Proceed to summarize_history
   ELSE:
     PROMPT: "✗ Connection test failed: {connectionTestError}. The delivery method is saved; you can update the configuration later."
     ASK [adaptive_card]: ActionSet (Retry | Skip)
     WAIT for user choice
     IF "Retry": Retry connection test tool call
     IF "Skip": Proceed to summarize_history (do not retry if skipped)

 IF "Skip": Proceed to summarize_history

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 3b — Connection Tested</completed><current_state>deliveryMethodUID={deliveryMethodUID}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/split-phase-4-delivery-method-summary</next_instructions></summary>"

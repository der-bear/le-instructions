# RESOURCE - PHASE 3b:

<phase_3b_test_connection>
 ANCHOR: anchor_test_connection
 IF deliveryType = "FTP" OR (deliveryType = "HttpPost" AND deliveryAddress starts with "http"):
   PROMPT: "Would you like to test the connection to your endpoint before continuing?"
   ASK [adaptive_card]: ActionSet (Test Connection | Skip)
   WAIT for user choice

   IF "Test Connection":
     IF deliveryType = "FTP":
       PROCESS (Detect protocol from deliveryAddress): if starts with "sftp://" → protocol="SFTP", else → protocol="FTP"
       TOOL: test_ftp_sftp_connection
       TOOL_DEFAULTS: protocol={detectedProtocol}, host={deliveryAddress}, username={ftpUser}, password={ftpPassword}, remotePath="/incoming/"

     IF deliveryType = "HttpPost":
       TOOL: test_webhook_connection
       IF mimeContentType AND requestBody:
         PROCESS (Generate test payload): Replace [SystemFieldName] placeholders in requestBody with sample values by field type (string→"Test", email→"test@example.com", phone→"5551234567", numeric→"50.00", bool→"true", date→"2025-01-01")
         TOOL_DEFAULTS: url={deliveryAddress}, method="POST", contentType={mimeContentType}, payload={testPayload}, timeoutSeconds=30
       ELSE:
         TOOL_DEFAULTS: url={deliveryAddress}, method="POST", payload="", timeoutSeconds=30

     IF tool result test_webhook_connection.IsSuccess = true OR test_ftp_sftp_connection.IsSuccess = true:
       PROMPT: "✓ Connection test successful."
       ASK [adaptive_card]: ActionSet (Continue)
       WAIT for user to click Continue THEN Proceed to summarize_history
     ELSE:
       PROMPT: "✗ Connection test failed: {test_webhook_connection.Error OR test_ftp_sftp_connection.Error}. The delivery method is saved; you can update the configuration later."
       ASK [adaptive_card]: ActionSet (Retry | Skip)
       WAIT for user choice
       IF "Retry": Retry connection test tool call
       IF "Skip": Proceed to summarize_history (do not retry if skipped)

   IF "Skip": Proceed to summarize_history

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="anchor_test_connection", summarization_text="<summary><next_phase>mcp://resource/phase-4-delivery-method-summary</next_phase></summary>"
</phase_3b_test_connection>
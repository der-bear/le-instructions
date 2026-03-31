# RESOURCE - PHASE 8:

<phase_8_activation>
 IF "Activate":
   TOOL: update_client → confirms activation
   TOOL_DEFAULTS: clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus="Active", clientAutomationType="Price", username={email}, password={generate-password}, timeZoneName={timeZoneName}, timeOffset={timeOffset}
   CRITICAL: updateClientDto must be passed as a native object, NOT a JSON string

   IF activation succeeds:
     PROMPT: "✓ Setup complete. Your lead delivery system is now \"ACTIVE\" for {companyName}."
     CRITICAL: Do not suggest any next steps. End the conversation here.

   IF activation fails:
     PROMPT: "We encountered an issue activating the client. Would you like to try again?"
     SUGGEST [adaptive_card]: ActionSet (Retry activation)
     WAIT for user choice
     CRITICAL: Do not suggest any other options or next steps beyond retry.

     IF "Retry activation":
       Loop back to TOOL: update_client (same parameters)

 IF "Keep Inactive":
   PROMPT: "✓ Setup complete. Client {companyName} has been configured but remains \"NEW\". You can activate it later when ready."
   CRITICAL: Do not suggest any next steps. End the conversation here.
</phase_8_activation>
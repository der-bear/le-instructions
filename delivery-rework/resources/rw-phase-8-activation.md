═══════════════════════════════════════
CURRENT PHASE: Phase 8 — Activation
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to either activate the client or confirm they remain inactive. Do NOT suggest any next steps after either terminal branch.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Activate Client (Do this first if applicable)**
* IF clientSummaryChoice = "Activate":
  1. CRITICAL: update_client REPLACES all client fields — every field in the DTO must be present or it will be erased. This is why we resend companyName, email, username, and timezone even though they haven't changed, and why we generate a fresh password (the Phase 1 password was not retained).
  2. Call the update_client tool with these defaults:
     `clientUID={clientUID}, updateClientDto={companyName={companyName}, email={email}, clientStatus="Active", clientAutomationType="Price", username={email}, password=<generate a random 14-character password using upper, lower, digit, and symbol characters>, timeZoneName={timeZoneName}, timeOffset={timeOffset}}`
     CRITICAL: updateClientDto must be passed as a nested native object inside the tool call, NOT as top-level loose fields and NOT as a JSON string.
  3. IF activation succeeds:
     - Prompt the user exactly as follows: "✓ Setup complete. Your lead delivery system is now \"ACTIVE\" for {companyName}."
     - End the conversation. Do not suggest any next steps.
  4. IF activation fails:
     - Prompt the user exactly as follows: "We encountered an issue activating the client.\n\nWould you like to try again, or keep the client inactive?"
     - Present the choice using display_adaptive_card with an ActionSet: "Retry activation" | "Keep Inactive".
     - **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
     - IF the user said "Retry activation": repeat step 2 (call update_client again with ALL fields and a freshly generated password). If it fails again, show the same retry prompt.
     - IF the user said "Keep Inactive": proceed to State 2.

**State 2: Keep Client Inactive**
* IF clientSummaryChoice = "Keep Inactive":
  1. Prompt the user exactly as follows: "✓ Setup complete. Client {companyName} has been configured but remains \"NEW\". You can activate it later when ready."
  2. End the conversation. Do not suggest any next steps.

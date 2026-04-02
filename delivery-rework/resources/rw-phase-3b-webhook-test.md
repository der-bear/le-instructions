═══════════════════════════════════════
CURRENT PHASE: Phase 3b — Webhook Connection Test
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to optionally test the webhook connection, then hand off to Phase 4.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Offer and Run Webhook Test (Do this first)**
* IF connectionTestChoice is missing:
  1. Prompt the user exactly as follows: "Would you like to test the connection to your endpoint before continuing?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Test Connection" | "Skip".
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF connectionTestChoice = "Skip":
  1. Proceed to State 2.

* IF connectionTestChoice = "Test Connection" OR connectionTestFailureChoice = "Retry":
  1. IF mimeContentType and requestBody are known, generate testPayload by replacing [SystemFieldName] placeholders with sample values (string→"Test", email→"test@example.com", phone→"5551234567", numeric→"50.00", bool→"true", date→"2025-01-01").
  2. IF mimeContentType and requestBody are known, call test_webhook_connection with: `url={deliveryAddress}, method="POST", contentType={mimeContentType}, payload={testPayload}, timeoutSeconds=30`
  3. ELSE call test_webhook_connection with: `url={deliveryAddress}, method="POST", payload="", timeoutSeconds=30`
  4. IF the test succeeds:
     - Prompt the user exactly as follows: "✓ Connection test successful."
     - Present an ActionSet: "Continue".
     - **STOP AND YIELD.** Do not hallucinate data. When the user clicks Continue, proceed to State 2.
  5. IF the test fails:
     - Prompt the user exactly as follows: "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later."
     - Present an ActionSet: "Retry" | "Skip".
     - **STOP AND YIELD.** Do not hallucinate data. If the user clicks Retry, re-run the test. If Skip, proceed to State 2.

**State 2: Summarize**
* IF connectionTestChoice = "Skip" OR the test succeeded and the user clicked Continue OR the test failed and the user clicked Skip:
  1. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 3b Complete — Webhook Connection Tested

# Current System State
* Flow Intent: {flowIntent}
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: {clientStatus}
* Time Zone Name: {timeZoneName}
* Time Offset: {timeOffset}
* Lead Type UID: {leadTypeUID}
* Lead Type Name: {leadTypeName}
* Delivery Method UID: {deliveryMethodUID}
* Delivery Method Name: {deliveryMethodName}
* Delivery Type: HttpPost
* Delivery Address: {deliveryAddress}
* MIME Content Type: {mimeContentType}
* Request Body: {requestBody}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: {mappedCount}
* Total Count: {totalCount}
* Connection Test Mode: webhook

# Next Instructions
→ Load and execute Phase 4 at mcp://resource/rw-phase-4-delivery-method-summary
```


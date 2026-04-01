# Phase 3b: Test Connection

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 3b resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to optionally test the connection to the delivery endpoint, then hand off to Phase 4.

Re-retain every value Phase 4 and Phase 5 still need. Do NOT drop any Phase 3 state in this summary.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Offer and Run Connection Test (Do this first)**
* This resource is only loaded for FTP and Webhook delivery types (Portal and Email skip directly to Phase 4).
* IF connectionTestMode = "ftp" OR connectionTestMode = "webhook":

  - IF connectionTestChoice is missing:
    1. Prompt the user exactly as follows: "Would you like to test the connection to your endpoint before continuing?"
    2. Present the choice using display_adaptive_card with an ActionSet: "Test Connection" | "Skip".
    3. **STOP AND YIELD.** Do not hallucinate data. Do not call the summarize_history tool. You must wait for the user to respond.

  - IF connectionTestChoice = "Skip":
    1. Proceed to State 2.

  - IF connectionTestChoice = "Test Connection" OR connectionTestFailureChoice = "Retry":
    * IF connectionTestMode = "ftp":
      1. Detect protocol from deliveryAddress: if it starts with "sftp://", protocol="SFTP"; otherwise protocol="FTP".
      2. Call the test_ftp_sftp_connection tool with: `protocol={protocol}, host={deliveryAddress}, username={ftpUser}, password={ftpPassword}, remotePath="/incoming/"`
    * IF connectionTestMode = "webhook":
      1. IF mimeContentType and requestBody are known, generate testPayload by replacing [SystemFieldName] placeholders with sample values (string→"Test", email→"test@example.com", phone→"5551234567", numeric→"50.00", bool→"true", date→"2025-01-01").
      2. IF mimeContentType and requestBody are known, call test_webhook_connection with: `url={deliveryAddress}, method="POST", contentType={mimeContentType}, payload={testPayload}, timeoutSeconds=30`
      3. ELSE call test_webhook_connection with: `url={deliveryAddress}, method="POST", payload="", timeoutSeconds=30`

    * IF the test succeeds:
      1. Prompt the user exactly as follows: "✓ Connection test successful."
      2. Present an ActionSet: "Continue".
      3. **STOP AND YIELD.** Do not hallucinate data. When the user clicks Continue, proceed to State 2.

    * IF the test fails:
      1. Prompt the user exactly as follows: "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later."
      2. Present an ActionSet: "Retry" | "Skip".
      3. **STOP AND YIELD.** Do not hallucinate data. If the user clicks Retry, re-run the test above. If Skip, proceed to State 2.

**State 2: Summarize**
* IF connectionTestChoice = "Skip" OR the test succeeded and the user clicked Continue OR the test failed and the user clicked Skip:
  1. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
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
* Delivery Type: {deliveryType}
* Delivery Address: {deliveryAddress}
* FTP User: {ftpUser}
* FTP Password: {ftpPassword}
* MIME Content Type: {mimeContentType}
* Request Body: {requestBody}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: {mappedCount}
* Total Count: {totalCount}
* Connection Test Mode: {connectionTestMode}

# Conversation Anchor
* DELIVERY_SETUP_START

# Next Instructions
Fetch and execute instructions from: mcp://resource/phase-4-delivery-method-summary
```

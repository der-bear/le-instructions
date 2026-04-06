═══════════════════════════════════════
CURRENT PHASE: Phase 3a — FTP Connection Test
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to optionally test the FTP connection, then hand off to Phase 4.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Offer and Run FTP Test (Do this first)**
* IF connectionTestChoice is missing:
  1. Prompt the user exactly as follows: "Would you like to test the connection to your endpoint before continuing?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Test Connection" | "Skip".
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF connectionTestChoice = "Skip":
  1. Proceed to State 2.

* IF connectionTestChoice = "Test Connection" OR connectionTestFailureChoice = "Retry":
  1. Detect protocol from deliveryAddress: if it starts with "sftp://", protocol="SFTP"; otherwise protocol="FTP".
  2. Call the test_ftp_sftp_connection tool with: `protocol={protocol}, host={deliveryAddress}, username={ftpUser}, password={ftpPassword}, remotePath="/incoming/"`
  3. IF the test succeeds:
     - Present using display_adaptive_card: TextBlock "✓ Connection test successful." + ActionSet: "Continue".
     - **STOP AND YIELD.** Do not hallucinate data. When the user says Continue, proceed to State 2.
  4. IF the test fails:
     - Present using display_adaptive_card: TextBlock "✗ Connection test failed: {error}. The delivery method is saved; you can update the configuration later." + ActionSet: "Retry" | "Skip".
     - **STOP AND YIELD.** Do not hallucinate data. If the user says Retry, re-run the test. If Skip, proceed to State 2.

**State 2: Summarize**
* IF connectionTestChoice = "Skip" OR the test succeeded and the user said Continue OR the test failed and the user said Skip:
  1. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 3a Complete — FTP Connection Tested

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
* Delivery Type: FTP
* Delivery Address: {deliveryAddress}
* FTP User: {ftpUser}
* FTP Password: {ftpPassword}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: 0
* Total Count: 0
* Connection Test Mode: ftp

# Next Instructions
→ Load and execute Phase 4 at mcp://resource/rw-phase-4-delivery-method-summary
```


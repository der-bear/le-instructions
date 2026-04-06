═══════════════════════════════════════
CURRENT PHASE: Phase 3 FTP — Create FTP Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to collect FTP credentials, create the FTP delivery method, and hand off to Phase 3a for connection testing.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Collect Credentials and Create Method**
Execute these steps in order:

Step 1: Collect FTP Credentials
  Prompt the user exactly as follows: "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"
  **STOP AND YIELD.** Do not hallucinate data. Do not call any tools. You must wait for the user to provide the FTP credentials.
  Verify all three fields (deliveryAddress, ftpUser, ftpPassword) were provided. If any are missing, re-prompt for the missing field(s). **STOP AND YIELD.**

Step 2: Create FTP Method
  Call the create_delivery_method tool with these defaults:
  `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="FTP", name="{companyName}-FTP", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, ftpPath="/incoming/", deliveryDays={deliveryDays}}`
  CRITICAL: createDeliveryMethodDto must be passed as a native object, NOT a JSON string.
  If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

Step 3: Retain and Summarize
  Retain: deliveryMethodUID, deliveryMethodName="{companyName}-FTP", deliveryType="FTP", deliveryAddress, ftpUser, ftpPassword, mappedCount=0, totalCount=0, connectionTestMode="ftp".
  Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 3 Complete — FTP Method Created

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
→ Load and execute Phase 3a at mcp://resource/rw-phase-3a-ftp-test
```

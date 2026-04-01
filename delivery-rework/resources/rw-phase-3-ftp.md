# Phase 3: FTP Delivery Method

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 3 FTP resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to collect FTP credentials, create the FTP delivery method, and hand off to Phase 3b for connection testing.

## Instructions

**State 1: Collect FTP Credentials (Do this first)**
* IF deliveryMethodUID is missing AND any of deliveryAddress, ftpUser, or ftpPassword is missing:
  1. Prompt the user exactly as follows: "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"
  2. **STOP AND YIELD.** Do not hallucinate data. Do not call any tools. You must wait for the user to provide the FTP credentials.

**State 2: Create FTP Method and Summarize**
* IF deliveryAddress, ftpUser, and ftpPassword are all known AND deliveryMethodUID is missing:
  1. Call the create_delivery_method tool with these defaults:
     `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="FTP", name="{companyName}-FTP", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, ftpPath="/incoming/", deliveryDays={deliveryDays}}`
  2. If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. If the tool succeeds, retain: deliveryMethodUID, deliveryMethodName="{companyName}-FTP", deliveryType="FTP", deliveryAddress, ftpUser, ftpPassword, mappedCount=0, totalCount=0, connectionTestMode="ftp".
  4. Immediately call the summarize_history tool.

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
* Delivery Type: FTP
* Delivery Address: {deliveryAddress}
* FTP User: {ftpUser}
* FTP Password: {ftpPassword}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: 0
* Total Count: 0
* Connection Test Mode: ftp

# Conversation Anchor
* DELIVERY_SETUP_START

# Next Instructions
Fetch and execute instructions from: mcp://resource/rw-phase-3b-test-connection
```

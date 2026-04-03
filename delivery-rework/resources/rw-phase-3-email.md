═══════════════════════════════════════
CURRENT PHASE: Phase 3 Email — Create Email Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to create the Email delivery method and hand off directly to Phase 4 (no connection test needed for Email).

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Create Email Method and Summarize (Do this first)**
* IF deliveryMethodUID is missing:
  1. Call the create_delivery_method tool with these defaults:
     `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="EMail", name="{companyName}-Email", enabled=true, leadTypeUID={leadTypeUID}, emailAddress={email}, toEmailAddress={email}, emailSubject="New Lead - {date}", emailOrSmsTemplate="Standard template", deliveryDays={deliveryDays}}`
     CRITICAL: createDeliveryMethodDto must be passed as a native object, NOT a JSON string.
  2. If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. If the tool succeeds, retain: deliveryMethodUID, deliveryMethodName="{companyName}-Email", deliveryType="EMail", deliveryAddress={email}, mappedCount=0, totalCount=0.
  4. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 3 Complete — Email Method Created

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
* Delivery Type: EMail
* Delivery Address: {email}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: 0
* Total Count: 0

# Next Instructions
→ Load and execute Phase 4 at mcp://resource/rw-phase-4-delivery-method-summary
```


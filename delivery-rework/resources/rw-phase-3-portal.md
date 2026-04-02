═══════════════════════════════════════
CURRENT PHASE: Phase 3 Portal — Create Portal Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to create the Portal delivery method and hand off directly to Phase 4 (no connection test needed for Portal).

## Instructions

**State 1: Create Portal Method and Summarize (Do this first)**
* IF deliveryMethodUID is missing:
  1. Call the create_delivery_method tool with these defaults:
     `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Portal", enabled=true, leadTypeUID={leadTypeUID}, deliveryDays={deliveryDays}}`
  2. If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. If the tool succeeds, retain: deliveryMethodUID, deliveryMethodName="{companyName}-Portal", deliveryType="HttpPost", deliveryAddress="Portal", mappedCount=0, totalCount=0.
  4. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 3 Complete — Portal Method Created

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
* Delivery Address: Portal
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: 0
* Total Count: 0

# Next Instructions
→ Load and execute Phase 4 at mcp://resource/rw-phase-4-delivery-method-summary
```


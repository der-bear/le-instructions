# Phase 3: Email Delivery Method

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 3 Email resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to create the Email delivery method and hand off directly to Phase 4 (no connection test needed for Email).

## Instructions

**State 1: Create Email Method and Summarize (Do this first)**
* IF deliveryMethodUID is missing:
  1. Call the create_delivery_method tool with these defaults:
     `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="EMail", name="{companyName}-Email", enabled=true, leadTypeUID={leadTypeUID}, emailAddress={email}, toEmailAddress={email}, emailSubject="New Lead - {date}", emailOrSmsTemplate="Standard template", deliveryDays={deliveryDays}}`
  2. If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.**
  3. If the tool succeeds, retain: deliveryMethodUID, deliveryMethodName="{companyName}-Email", deliveryType="EMail", deliveryAddress={email}, mappedCount=0, totalCount=0.
  4. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
## State
flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType=EMail, deliveryAddress={email}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount=0, totalCount=0

## Next
mcp://resource/phase-4-delivery-method-summary
```

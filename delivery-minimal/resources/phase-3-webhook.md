═══════════════════════════════════════
CURRENT PHASE: Phase 3 — Webhook Delivery Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Create a webhook delivery method using either default field names or one pasted posting specification, then summarize the result and move on.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.

1. `deliveryAddress`
2. `mappingMode`
   - Ask: `Provide instructions` or `Skip mapping`
3. `contentTypeChoice`
   - Ask only if `mappingMode="Provide instructions"`
   - Options: `URL Encoded`, `JSON`, `XML`, `I'm not sure`
4. `postingInstructions`
   - Ask only if `mappingMode="Provide instructions"`

Accept typed equivalents for every card choice.

## PROCESS
- Normalize `deliveryAddress` by prepending `https://` if the scheme is missing.
- Reject URL-only specs and ask the user to paste the actual field list or schema text.
- If the spec is too large, repetitive, or poorly structured, ask for a smaller excerpt or let the user switch to `Skip mapping`.
- If `contentTypeChoice="I'm not sure"`, detect the format once after the spec is accepted and confirm it once.
- Do not call `get_lead_type` until the user has supplied a usable pasted spec.

## TOOL
### Skip mapping path
- Call `create_delivery_method` with:
  - `clientUID={clientUID}`
  - `createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, responseSearch="success", useRegEx=false, settings=null, requestBody=null, deliveryDays={deliveryDays}}`
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName="{companyName}-Webhook"`
  - `deliveryType="HttpPost"`
  - `mappedCount=0`
  - `totalCount=0`
  - `mimeContentType=""`
  - `requestBody=""`

### Pasted spec path
- Call `get_lead_type(leadTypeUID)` and retain `leadFields`.
- Build `mappingSettings` and `requestBody` from the accepted spec:
  - URL Encoded → flat form body
  - JSON/XML → preserve the user's hierarchy
- MIME types:
  - `JSON -> application/json`
  - `XML -> application/xml`
  - `URL Encoded -> application/x-www-form-urlencoded`
- If mapping is ambiguous or unresolved, ask one blocking clarification step and wait.
- After ambiguity is resolved, show a short plain-text summary with:
  - content type
  - mapped count
  - unresolved or omitted fields
- Ask for one final create-or-revise confirmation.
- When confirmed, call `create_delivery_method` with:
  - `clientUID={clientUID}`
  - `createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, contentType={mimeContentType}, responseSearch="success", useRegEx=false, settings={mappingSettings}, requestBody={requestBody}, deliveryDays={deliveryDays}}`
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName="{companyName}-Webhook"`
  - `deliveryType="HttpPost"`
  - `mimeContentType`
  - `requestBody`
  - `mappedCount`
  - `totalCount`

## FAILURE
- If the create call can be safely repaired, repair once silently.
- If it still fails, ask whether to retry with the same webhook config or revise the URL/spec.

## SUMMARIZE
- If the webhook method was created successfully and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 3 Complete — Webhook Method Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/min-phase-4-create-delivery-account</next_instructions></summary>"`

## NEXT
- If `flowIntent="full-setup"` and the method was created successfully, load and execute mcp://resource/min-phase-4-create-delivery-account.
- If `flowIntent="add-method"` and the method was created successfully, reply in plain text with the new `deliveryMethodUID` and stop.

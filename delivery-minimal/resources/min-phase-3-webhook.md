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
- For `JSON` or `XML`, apply at most one lossless repair pass before asking the user to revise. Allowed repairs include wrapping missing outer braces, removing trailing commas, converting obvious single quotes to double quotes, and equivalent lossless syntax cleanup that does not change business meaning.

## TOOL
- `createDeliveryMethodDto` must be a nested native object, not a JSON string.
- `settings` must be either `null` or a native array of mapping objects.
- `requestBody` must be either `null` or the actual webhook template text to send, not a JSON-escaped wrapper object.
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
- After the pasted spec is accepted and any one-pass repair is complete, call `get_lead_type(leadTypeUID)` and retain `leadFields`.
- Build `mappingSettings` and `requestBody` from the accepted spec:
  - `mappingSettings` must be a native array of `{fieldType:"LeadField", fieldName:<delivery field>, leadFieldUID:<system field uid>}` objects.
  - Include only mapped fields in both `mappingSettings` and `requestBody`.
  - `requestBody` must use `[SystemFieldName]` placeholders.
  - URL Encoded → flat form body
  - JSON → preserve the user's hierarchy and use quoted placeholder strings
  - XML → preserve the user's hierarchy and use unquoted placeholder text inside elements
- MIME types:
  - `JSON -> application/json`
  - `XML -> application/xml`
  - `URL Encoded -> application/x-www-form-urlencoded`
- If mapping is ambiguous or unresolved, ask one blocking clarification step and wait.
- If the repaired spec is still unusable after one pass, offer only:
  - `Re-paste excerpt`
  - `Change content type`
  - `Skip mapping`
- Unusable-spec recovery:
  - `Re-paste excerpt` → clear `postingInstructions`, `mappingSettings`, `requestBody`, `mappedCount`, and `totalCount`, keep `deliveryAddress` and the current confirmed `contentTypeChoice`, and return to the posting-instructions step
  - `Change content type` → clear `contentTypeChoice`, `postingInstructions`, `mappingSettings`, `requestBody`, `mappedCount`, and `totalCount`, then return to the content-type step
  - `Skip mapping` → clear `mappingSettings`, `requestBody`, `mimeContentType`, `mappedCount`, and `totalCount`, treat the flow as `Skip mapping`, and use the no-mapping create path
- After ambiguity is resolved, show a short plain-text summary with:
  - content type
  - mapped count as `X of Y extracted fields`
  - unresolved or omitted fields by name, or `None`
- Then ask for exactly one explicit review choice:
  - `Create method`
  - `Revise URL`
  - `Revise spec`
  - `Change content type`
  - `Skip mapping`
- Accept typed equivalents for the five review choices.
- Reset behavior:
  - `Revise URL` → clear `deliveryAddress` and `webhookReviewChoice`, keep the accepted spec, and return to the URL step
  - `Revise spec` → clear `postingInstructions`, `mappingSettings`, `requestBody`, `mappedCount`, `totalCount`, and `webhookReviewChoice`, then return to the posting-instructions step for the current content type
  - `Change content type` → clear `contentTypeChoice`, `postingInstructions`, `mappingSettings`, `requestBody`, `mappedCount`, `totalCount`, and `webhookReviewChoice`, then return to the content-type step
  - `Skip mapping` → clear `mappingSettings`, `requestBody`, `mimeContentType`, `mappedCount`, `totalCount`, and `webhookReviewChoice`, treat the flow as `Skip mapping`, and use the no-mapping create path
- Do not call `create_delivery_method` on the pasted-spec path until the user explicitly chooses `Create method`.
- When the user chooses `Create method`, call `create_delivery_method` with:
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
- If it still fails, offer only:
  - `Retry with same config`
  - `Revise URL`
  - `Revise spec`
  - `Change content type`
  - `Skip mapping`
- Accept typed equivalents and apply the same reset behavior as the review step above.

## SUMMARIZE
- If the webhook method was created successfully and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 3 Complete — Webhook Method Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/min-phase-4-create-delivery-account</next_instructions></summary>"`

## NEXT
- If `flowIntent="full-setup"` and the method was created successfully, load and execute mcp://resource/min-phase-4-create-delivery-account.
- If `flowIntent="add-method"` and the method was created successfully, reply in plain text with the new `deliveryMethodUID` and stop.

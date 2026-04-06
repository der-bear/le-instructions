═══════════════════════════════════════
CURRENT PHASE: Phase 3 — Webhook Delivery Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect the delivery schedule, create a webhook delivery method using either default field names or one pasted posting specification, review risky mapping before create, then show the created method before moving on.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.
Any item without an explicit card instruction is asked in plain text.

1. `deliveryScheduleChoice`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `24/7 delivery`
     - `Specific hours only`
2. `scheduleInput`
   - Ask only if `deliveryScheduleChoice="Specific hours only"`
3. `deliveryAddress`
4. `mappingMode`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `Provide instructions`
     - `Skip mapping`
5. `contentTypeChoice`
   - Ask only if `mappingMode="Provide instructions"`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `URL Encoded`
     - `JSON`
     - `XML`
     - `I'm not sure`
6. `postingInstructions`
   - Ask only if `mappingMode="Provide instructions"`

Accept typed equivalents for every card choice.

## BUILD
- Build and retain `deliveryDays` and `deliveryScheduleDisplay` before method creation:
  - `deliveryDays` must remain a native array of exactly 7 objects with `weekDay`, `allow`, `startTime`, and `endTime`.
  - For `24/7 delivery`, build exactly 7 entries with `allow=true`.
  - For `Specific hours only`, parse the user's schedule into exactly 7 entries using `timeOffset` when known, otherwise `-08:00`.
- Normalize `deliveryAddress` by prepending `https://` if the scheme is missing.
- Reject URL-only specs and ask the user to paste the actual field list or schema text.
- If the spec is too large, repetitive, or poorly structured, ask for a smaller excerpt or let the user switch to `Skip mapping`.
- If `contentTypeChoice="I'm not sure"`, detect the format once after the spec is accepted, then ask the user to either accept the detected format or choose a different content type before continuing.
- Do not call `get_lead_type` until the user has supplied a usable pasted spec.
- For `JSON` or `XML`, apply at most one lossless repair pass before asking the user to revise. Allowed repairs include wrapping missing outer braces, removing trailing commas, converting obvious single quotes to double quotes, and equivalent lossless syntax cleanup that does not change business meaning.
- Auto-detect confirmation:
  - if the format was auto-detected, ask using `display_adaptive_card` with exactly one `ActionSet` containing:
    - `Use {detectedFormat}`
    - `Choose a different content type`
  - accept only typed equivalents that clearly map to those two responses
  - `Use {detectedFormat}` → set `contentTypeChoice` to the detected format and continue
  - `Choose a different content type` → clear `contentTypeChoice`, `mappingSettings`, `requestBody`, `mappedCount`, and `totalCount`, keep the accepted `postingInstructions`, and return to the content-type step

### Pasted spec preparation
- After the pasted spec is accepted and any one-pass repair is complete, call `get_lead_type(leadTypeUID)` and retain `leadFields`.
- Build `mappingSettings` and `requestBody` from the accepted spec:
  - treat each extracted leaf field or full nested path as one extracted field for `totalCount`
  - match extracted delivery fields to `leadFields` in this order:
    - exact name match
    - separator or case variants such as spaces, underscores, and camelCase
    - common abbreviations
    - semantic match only when confidence is clearly above 90%
  - if multiple system fields tie at the same priority or confidence is below that threshold, do not auto-map that field
  - `mappingSettings` must be a native array of `{fieldType:"LeadField", fieldName:<delivery field>, leadFieldUID:<system field uid>}` objects.
  - for nested JSON or XML, `fieldName` must use the full nested delivery path
  - Include only mapped fields in both `mappingSettings` and `requestBody`.
  - unmapped extracted fields must stay out of `mappingSettings` and `requestBody`, but they still count toward `totalCount` and the unresolved or omitted list
  - do not map the same delivery field path more than once
  - `requestBody` must use `[SystemFieldName]` placeholders.
  - preserve extracted sibling order from the accepted spec
  - keep only the ancestor containers needed to hold mapped descendants; omit empty objects, XML branches, and array items that end up with no mapped descendants
  - if the accepted JSON or XML includes arrays, preserve the array shell and keep one representative mapped element structure inside it
  - URL Encoded → flat form body in extracted field order using only mapped pairs
  - JSON → preserve the user's hierarchy and use quoted placeholder strings
  - XML → preserve the user's hierarchy and use unquoted placeholder text inside elements
  - MIME types:
  - `JSON -> application/json`
  - `XML -> application/xml`
  - `URL Encoded -> application/x-www-form-urlencoded`
- If mapping is ambiguous or unresolved, ask one blocking clarification step for exactly one delivery field at a time and wait.
- In that clarification step, accept only:
   - one chosen system field
   - `Skip this field`
   - `Revise spec`
- After each clarification, recompute `mappingSettings`, `requestBody`, `mappedCount`, `totalCount`, and the unresolved or omitted list before deciding whether another clarification step is required.
- If the repaired spec is still unusable after one pass, ask using `display_adaptive_card` with exactly one `ActionSet` containing:
  - `Re-paste excerpt`
  - `Change content type`
  - `Skip mapping`
- Unusable-spec recovery:
  - `Re-paste excerpt` → clear `postingInstructions`, `mappingSettings`, `requestBody`, `mappedCount`, and `totalCount`, keep `deliveryAddress` and the current confirmed `contentTypeChoice`, and return to the posting-instructions step
  - `Change content type` → clear `contentTypeChoice`, `postingInstructions`, `mappingSettings`, `requestBody`, `mappedCount`, and `totalCount`, then return to the content-type step
  - `Skip mapping` → clear `mappingSettings`, `requestBody`, `mimeContentType`, `mappedCount`, and `totalCount`, treat the flow as `Skip mapping`, and use the no-mapping create path
- Before rendering the review card, prepare a compact preview when needed:
  - if there are more than 20 mapped rows, render only the first 20 mapped rows in extracted field order and add a short note that the preview is truncated
  - if the unresolved or omitted list would exceed 400 characters, summarize it as `{totalCount - mappedCount} not mapped` instead of listing every field name
- After ambiguity is resolved, continue to `SHOW REVIEW (CARD)`.

## SHOW REVIEW (CARD)
- After ambiguity is resolved, call `display_adaptive_card` and render exactly one `WebhookMappingReviewCard` titled `Field Mapping Preview`.
- Render one 2-column summary table with rows in exactly this order:
  - `Content Type | {contentTypeChoice}`
  - `Mapped Fields | {mappedCount} of {totalCount} extracted fields`
  - `Not Mapped | {totalCount - mappedCount}`
- Render one 3-column mapping table with exact header text:
  - `Delivery Field | System Field | Status`
- Render mapped rows in extracted field order.
- If there are 20 or fewer mapped rows, render one row per mapped field.
- If there are more than 20 mapped rows, render only the first 20 rows and add one short note: `Showing first 20 of {mappedCount} mapped fields.`
- Status text is always `Mapped`.
- If there are zero mapped rows, render only the header row.
- Render exactly one root `Action.Submit` button:
  - `Continue` -> `data.action="continue-mapping-preview"`
- Accept only typed equivalents that clearly map to `continue-mapping-preview`.
- Do not call `create_delivery_method` on the pasted-spec path until the user explicitly chooses `Continue`.

## USE TOOL
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
  - `requestTemplateStatus="Not used"`
  - `previewCheckpoint="method-created"`
  - `previewNextAction="delivery-account"`
  - `previewEntityUID={deliveryMethodUID}`
- If `flowIntent="add-method"` and the webhook method is created successfully on the skip-mapping path, overwrite `previewNextAction="finish"` before showing the preview.
- Validate that `deliveryMethodUID` is present and positive before showing the preview. If it is missing or invalid, treat the create step as failed.

### Pasted spec path
- When the user chooses `Continue`, call `create_delivery_method` with:
  - `clientUID={clientUID}`
  - `createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, contentType={mimeContentType}, responseSearch="success", useRegEx=false, settings={mappingSettings}, requestBody={requestBody}, deliveryDays={deliveryDays}}`
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName="{companyName}-Webhook"`
  - `deliveryType="HttpPost"`
  - `mimeContentType`
  - `requestTemplateStatus="Saved"`
  - `mappedCount`
  - `totalCount`
  - `previewCheckpoint="method-created"`
  - `previewNextAction="delivery-account"`
  - `previewEntityUID={deliveryMethodUID}`
- If `flowIntent="add-method"` and the webhook method is created successfully on the pasted-spec path, overwrite `previewNextAction="finish"` before showing the preview.
- Validate that `deliveryMethodUID` is present and positive before showing the preview. If it is missing or invalid, treat the create step as failed.

## SHOW PREVIEW (CARD)
- After the webhook method is created successfully, call `display_adaptive_card` and render exactly one `KeyValuePreviewCard` titled `Method created`.
- Use exactly these rows in this order:
  - `Method UID | {deliveryMethodUID}`
  - `Method Name | {deliveryMethodName}`
  - `Webhook URL | {deliveryAddress}`
  - `Content Type | <normalized content type display>`
  - `Mapped Fields | {mappedCount} of {totalCount}`
  - `Delivery Schedule | {deliveryScheduleDisplay}`
  - `Request Template | {requestTemplateStatus}`
- Normalize display values:
  - when mapping was skipped, `Content Type | Default field names`
  - when mapping was used, `Content Type | {mimeContentType}`
- Do not add extra rows.
- Actions:
  - if `previewNextAction="delivery-account"`, render `Continue to Delivery Account` with `data.action="continue-delivery-account"`
  - if `previewNextAction="finish"`, render `Finish` with `data.action="finish-method"`
- Accept only typed equivalents that clearly map to the currently pending action token.
- If the user acknowledges:
  - clear `previewCheckpoint`, `previewNextAction`, and `previewEntityUID`
  - if `flowIntent="full-setup"`, continue to `SUMMARIZE` and `NEXT`
  - if `flowIntent="add-method"`, reply in plain text with the new `deliveryMethodUID` and stop
- If the user does not acknowledge, re-show the same preview and wait.
- Do not call `create_delivery_method` again while this preview is pending.

## FAILURE
- If the create call can be safely repaired, repair once silently.
- If it still fails, offer only:
  - `Retry with same config`
  - `Revise URL`
  - `Revise spec`
  - `Change content type`
  - `Skip mapping`
- Accept typed equivalents and apply the same reset behavior as the review step above.
- `Retry with same config` means retry the same confirmed `create_delivery_method` call once with the current retained values and without asking any new questions first.

## SUMMARIZE
- If the webhook method was created successfully, the preview was acknowledged, and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 3 Complete — Webhook Method Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, mimeContentType={mimeContentType}, requestTemplateStatus={requestTemplateStatus}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/min-phase-4-create-delivery-account</next_instructions></summary>"`

## NEXT
- After the preview is acknowledged:
  - if `flowIntent="full-setup"`, load and execute mcp://resource/min-phase-4-create-delivery-account
  - if `flowIntent="add-method"`, reply in plain text with the new `deliveryMethodUID` and stop

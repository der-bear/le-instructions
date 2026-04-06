═══════════════════════════════════════
CURRENT PHASE: Phase 3 — Portal Delivery Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect the delivery schedule, create the Portal delivery method, show the created method, then either continue to account creation or finish the add-method flow.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.

1. `deliveryScheduleChoice`
   - Ask: `24/7 delivery` or `Specific hours only`
2. `scheduleInput`
   - Ask only if `deliveryScheduleChoice="Specific hours only"`

## BUILD
- Build and retain `deliveryDays` and `deliveryScheduleDisplay` before creation:
  - `deliveryDays` must remain a native array of exactly 7 objects with `weekDay`, `allow`, `startTime`, and `endTime`.
  - For `24/7 delivery`, build exactly 7 entries with `allow=true`.
  - For `Specific hours only`, parse the user's schedule into exactly 7 entries using `timeOffset` when known, otherwise `-08:00`.

## USE TOOL
- If `methodPreviewPending=true`, do not call `create_delivery_method` again. Continue directly to `SHOW PREVIEW (CARD)`.
- `createDeliveryMethodDto` must be a nested native object, not a JSON string.
- Call `create_delivery_method` with:
  - `clientUID={clientUID}`
  - `createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Portal", enabled=true, leadTypeUID={leadTypeUID}, deliveryDays={deliveryDays}}`
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName="{companyName}-Portal"`
  - `deliveryType="HttpPost"`
  - `deliveryAddress="Portal"`
  - `mappedCount=0`
  - `totalCount=0`
  - `methodPreviewPending=true`
- Validate that `deliveryMethodUID` is present and positive before showing the preview. If it is missing or invalid, treat the create step as failed.

## FAILURE
- If the create call can be safely repaired, repair once silently.
- If method creation still fails, ask the user whether to retry with the same inputs.

## SHOW PREVIEW (CARD)
- Call `display_adaptive_card` and render exactly one `KeyValuePreviewCard` titled `Method created`.
- Use exactly these rows in this order:
  - `Method UID | {deliveryMethodUID}`
  - `Method Name | {deliveryMethodName}`
  - `Delivery Type | Portal`
  - `Destination | Portal`
  - `Delivery Schedule | {deliveryScheduleDisplay}`
- Do not add extra rows.
- Actions:
  - if `flowIntent="full-setup"`, render `Continue to Delivery Account` with `data.action="continue-delivery-account"`
  - if `flowIntent="add-method"`, render `Finish` with `data.action="finish-method"`
- Accept only typed equivalents that clearly map to the currently pending action token.
- If the user acknowledges:
  - clear `methodPreviewPending`
  - if `flowIntent="full-setup"`, continue to `SUMMARIZE` and `NEXT`
  - if `flowIntent="add-method"`, reply in plain text with the new `deliveryMethodUID` and stop
- If the user does not acknowledge, re-show the same preview and wait.
- Do not call `create_delivery_method` again while this preview is pending.

## SUMMARIZE
- If the Portal method was created successfully, the preview was acknowledged, and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 3 Complete — Portal Method Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/min-phase-4-create-delivery-account</next_instructions></summary>"`

## NEXT
- After the preview is acknowledged:
  - if `flowIntent="full-setup"`, load and execute mcp://resource/min-phase-4-create-delivery-account
  - if `flowIntent="add-method"`, reply in plain text with the new `deliveryMethodUID` and stop

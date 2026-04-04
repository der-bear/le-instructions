═══════════════════════════════════════
CURRENT PHASE: Phase 3 — Delivery Method Router
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect delivery schedule and delivery type, create the simple method types inline, and route only Webhook to the dedicated webhook resource.

## IMPORTANT
- Do not call `summarize_history` in this router before loading the webhook resource.
- This phase retains `deliveryDays` as a native array, and the webhook phase may need that structure directly.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.

1. `deliveryScheduleChoice`
   - Ask: `24/7 delivery` or `Specific hours only`
2. `scheduleInput`
   - Ask only if `deliveryScheduleChoice="Specific hours only"`
3. `deliveryTypeChoice`
   - Ask: `Portal`, `Webhook`, `Email`, or `FTP`
4. `deliveryAddress`, `ftpUser`, `ftpPassword`
   - Ask only if `deliveryTypeChoice="FTP"`

Accept typed equivalents for every card choice.

## PROCESS
- Build and retain `deliveryDays` and `deliveryScheduleDisplay` before creation:
  - For `24/7 delivery`, build exactly 7 entries with `allow=true`.
  - For `Specific hours only`, parse the user's schedule into exactly 7 entries using `timeOffset` when known, otherwise `-08:00`.
- If `deliveryTypeChoice="Webhook"`, route to the webhook phase instead of creating the method here.

## TOOL
### Portal
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

### Email
- Call `create_delivery_method` with:
  - `clientUID={clientUID}`
  - `createDeliveryMethodDto={deliveryType="EMail", name="{companyName}-Email", enabled=true, leadTypeUID={leadTypeUID}, emailAddress={email}, toEmailAddress={email}, emailSubject="New Lead - {date}", emailOrSmsTemplate="Standard template", deliveryDays={deliveryDays}}`
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName="{companyName}-Email"`
  - `deliveryType="EMail"`
  - `deliveryAddress={email}`
  - `mappedCount=0`
  - `totalCount=0`

### FTP
- Call `create_delivery_method` with:
  - `clientUID={clientUID}`
  - `createDeliveryMethodDto={deliveryType="FTP", name="{companyName}-FTP", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, ftpPath="/incoming/", deliveryDays={deliveryDays}}`
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName="{companyName}-FTP"`
  - `deliveryType="FTP"`
  - `deliveryAddress`
  - `ftpUser`
  - `ftpPassword`
  - `mappedCount=0`
  - `totalCount=0`

## FAILURE
- If the create call can be safely repaired, repair once silently.
- If method creation still fails, ask the user whether to retry with the same inputs or revise the last method-specific input.

## SUMMARIZE
- If `deliveryTypeChoice="Webhook"`, do not summarize in this router. Load the webhook phase directly.
- If a simple method was created successfully and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 3 Complete — Delivery Method Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/min-phase-4-create-delivery-account</next_instructions></summary>"`

## NEXT
- If `deliveryTypeChoice="Webhook"`, load and execute mcp://resource/min-phase-3-webhook.
- If `flowIntent="full-setup"` and a simple method was created successfully, load and execute mcp://resource/min-phase-4-create-delivery-account.
- If `flowIntent="add-method"` and a simple method was created successfully, reply in plain text with the new `deliveryMethodUID` and stop.

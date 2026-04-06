═══════════════════════════════════════
CURRENT PHASE: Phase 3 — Delivery Method Router
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Select the delivery type and route to the matching method-specific resource.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.

1. `deliveryTypeChoice`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing four `Action.Submit` buttons:
     - `Portal`
     - `Webhook`
     - `Email`
     - `FTP`

Accept typed equivalents for every card choice.

## RESOLVE
- If `deliveryTypeChoice` does not clearly resolve to one of `Portal`, `Webhook`, `Email`, or `FTP`, re-ask the delivery-type choice and wait.

## BUILD
- Before `NEXT`, clear stale Phase 3 method-attempt state from any earlier method run in this thread so the selected branch starts fresh.
- Clear:
  - `deliveryMethodUID`
  - `deliveryMethodName`
  - `deliveryType`
  - `deliveryAddress`
  - `deliveryScheduleChoice`
  - `scheduleInput`
  - `deliveryDays`
  - `deliveryScheduleDisplay`
  - `ftpUser`
  - `ftpPassword`
  - `mappingMode`
  - `contentTypeChoice`
  - `postingInstructions`
  - `mappingSettings`
  - `requestBody`
  - `mimeContentType`
  - `requestTemplateStatus`
  - `mappedCount`
  - `totalCount`
  - `methodPreviewPending`
  - `previewCheckpoint`
  - `previewNextAction`
  - `previewEntityUID`

## NEXT
- If `deliveryTypeChoice="Portal"`, load and execute mcp://resource/min-phase-3-portal.
- If `deliveryTypeChoice="Email"`, load and execute mcp://resource/min-phase-3-email.
- If `deliveryTypeChoice="FTP"`, load and execute mcp://resource/min-phase-3-ftp.
- If `deliveryTypeChoice="Webhook"`, load and execute mcp://resource/min-phase-3-webhook.

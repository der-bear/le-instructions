═══════════════════════════════════════
CURRENT PHASE: Phase 1 — Create Client
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect the client inputs, create the client, show the created client, then summarize and hand off to Phase 2.

## ASK
Collect missing inputs in this order:
1. `companyName`
2. `email`

Rules:
- If both are missing, ask for both in one message.
- If only one is missing, ask only for that field.
- After asking, wait for the user's reply.

## BUILD
- Normalize `email` by trimming whitespace and lowercasing the domain portion.

## USE TOOL
- Build `createClientDto` as a nested native object, not a JSON string or top-level loose fields.
- `password` must be an actual generated 14-character string value, not the literal placeholder text.
- Call `create_client` with:
  - `createClientDto={companyName={companyName}, email={email}, clientStatus="New", clientAutomationType="Price", username={email}, password=<fresh random 14-character password>, timeZoneName="Pacific Standard Time", timeOffset=-8}`
- Retain:
  - `clientUID`
  - `clientStatus="New"`
  - `timeZoneName="Pacific Standard Time"`
  - `timeOffset=-8`
  - `previewCheckpoint="client-created"`
  - `previewNextAction="lead-type"`
  - `previewEntityUID={clientUID}`
- Validate that `clientUID` is present and positive before showing the preview. If it is missing or invalid, treat the create step as failed.

## FAILURE
- If the payload can be safely repaired, repair once silently.
- If creation still fails, ask the user to confirm the company name and email before retrying.

## SHOW PREVIEW (CARD)
- After the client is created successfully, call `display_adaptive_card` and render exactly one `KeyValuePreviewCard` titled `Client created`.
- Use exactly these rows in this order:
  - `Client UID | {clientUID}`
  - `Company Name | {companyName}`
  - `Email | {email}`
  - `Status | {clientStatus}`
  - `Time Zone | {timeZoneName}`
- Do not add extra rows.
- Render exactly one root `Action.Submit`:
  - `Continue to Lead Type` with `data.action="continue-lead-type"`
- Accept only typed equivalents that clearly map to `continue-lead-type`.
- If the user acknowledges:
  - clear `previewCheckpoint`, `previewNextAction`, and `previewEntityUID`
  - continue to `SUMMARIZE` and `NEXT`
- If the user does not acknowledge, re-show the same preview and wait.
- Do not call `create_client` again while this preview is pending.

## SUMMARIZE
- After the preview is acknowledged, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 1 Complete — Client Created</completed><current_state>flowIntent=full-setup, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus=New, timeZoneName=Pacific Standard Time, timeOffset=-8</current_state><next_instructions>Load and execute Phase 2 from mcp://resource/min-phase-2-get-lead-types</next_instructions></summary>"`

## NEXT
- After the preview is acknowledged and the summary is complete, load and execute mcp://resource/min-phase-2-get-lead-types.

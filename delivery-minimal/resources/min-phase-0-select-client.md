═══════════════════════════════════════
CURRENT PHASE: Phase 0 — Select Client
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Select an existing client, load the client profile, summarize the result, and hand off to Phase 2.

## ASK
- If `clientUID` is already known and trusted, skip selection and load the profile.
- Otherwise:
  1. Call `get_clients` and retain `clientsList`.
  2. If `clientsList` is empty, tell the user no clients were found and stop.
  3. Ask: "Which client would you like to create a delivery method for?"
  4. If using `display_adaptive_card`, render a compact `Input.ChoiceSet` of clients plus `Action.Submit` with no extra submit `data`. The submitted value must be only `clientUID`.
  5. Accept typed company name or typed `clientUID` as fallback.
  6. If the typed match is ambiguous or low confidence, re-display the selector and wait again.

## TOOL
- After `clientUID` is known, call `get_client(clientUID)` and retain:
  - `clientUID`
  - `companyName`
  - `email`
  - `clientStatus`
  - `timeZoneName`
  - `timeOffset`

## FAILURE
- If `get_client` fails, ask the user to retry with the same client or choose another client.

## SUMMARIZE
- After the client profile is loaded successfully, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 0 Complete — Client Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}</current_state><next_instructions>Load and execute Phase 2 from mcp://resource/min-phase-2-get-lead-types</next_instructions></summary>"`

## NEXT
Load and execute mcp://resource/min-phase-2-get-lead-types.

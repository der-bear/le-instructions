═══════════════════════════════════════
CURRENT PHASE: Phase 2 — Lead Type Selection
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Select the lead type, summarize the result, and hand off to Phase 3.

## ASK
- Preferred path: call `display_lead_types_choice` with prompt: "Please select a Lead Type for this client."
- Wait for the user's selection.
- On a successful selector choice, immediately retain the returned `leadTypeUID` and `leadTypeName`.
- If the user types instead of selecting:
  - call `get_lead_types` if needed and retain `leadTypesList`
  - match the typed value against `leadTypeName`
  - if the match is clear, retain the matched `leadTypeUID` and `leadTypeName`
  - if the match is ambiguous or low confidence, re-display the selector and wait again

## FAILURE
- If no lead types are available, tell the user there are no lead types available and stop.

## SUMMARIZE
- After `leadTypeUID` and `leadTypeName` are confirmed, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 2 Complete — Lead Type Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}</current_state><next_instructions>Load and execute Phase 3 from mcp://resource/min-phase-3-create-delivery-method</next_instructions></summary>"`

## NEXT
Load and execute mcp://resource/min-phase-3-create-delivery-method.

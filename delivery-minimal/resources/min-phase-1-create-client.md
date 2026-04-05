═══════════════════════════════════════
CURRENT PHASE: Phase 1 — Create Client
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect the client inputs, create the client, summarize the result, and hand off to Phase 2.

## ASK
Collect missing inputs in this order:
1. `companyName`
2. `email`

Rules:
- If both are missing, ask for both in one message.
- If only one is missing, ask only for that field.
- After asking, wait for the user's reply.

## PROCESS
- Normalize `email` by trimming whitespace and lowercasing the domain portion.

## TOOL
- Build `createClientDto` as a nested native object, not a JSON string or top-level loose fields.
- `password` must be an actual generated 14-character string value, not the literal placeholder text.
- Call `create_client` with:
  - `createClientDto={companyName={companyName}, email={email}, clientStatus="New", clientAutomationType="Price", username={email}, password=<fresh random 14-character password>, timeZoneName="Pacific Standard Time", timeOffset=-8}`
- Retain:
  - `clientUID`
  - `companyName`
  - `email`
  - `clientStatus="New"`
  - `timeZoneName="Pacific Standard Time"`
  - `timeOffset=-8`
  - `flowIntent="full-setup"`

## FAILURE
- If the payload can be safely repaired, repair once silently.
- If creation still fails, ask the user to confirm the company name and email before retrying.

## SUMMARIZE
- After the client is created successfully, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 1 Complete — Client Created</completed><current_state>flowIntent=full-setup, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus=New, timeZoneName=Pacific Standard Time, timeOffset=-8</current_state><next_instructions>Load and execute Phase 2 from mcp://resource/min-phase-2-get-lead-types</next_instructions></summary>"`

## NEXT
Load and execute mcp://resource/min-phase-2-get-lead-types.

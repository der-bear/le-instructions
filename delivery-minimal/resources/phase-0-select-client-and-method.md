═══════════════════════════════════════
CURRENT PHASE: Phase 0 — Select Client And Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Select an existing client and delivery method, retain the account-creation context, summarize the result, and hand off to Phase 4.

## ASK
- If `clientUID` is already known and trusted, skip client selection.
- Otherwise:
  1. Call `get_clients` and retain `clientsList`.
  2. If `clientsList` is empty, tell the user no clients were found and stop.
  3. Ask: "Which client would you like to create a delivery account for?"
  4. Prefer a compact dropdown via `display_adaptive_card`.
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
- Then call `get_delivery_methods(clientUID)` and retain `deliveryMethodsList` with:
  - `deliveryMethodUID`
  - `deliveryMethodName` from `data.name`
  - `leadTypeUID` from `data.leadTypeUID`

## ASK
- If no delivery methods exist, tell the user this client has no delivery methods and stop.
- Ask: "Which delivery method would you like to use?"
- Prefer a compact dropdown via `display_adaptive_card`.
- Accept typed method name or typed `deliveryMethodUID` as fallback.
- If the typed match is ambiguous or low confidence, re-display the selector and wait again.

## TOOL
- Retain:
  - `deliveryMethodUID`
  - `deliveryMethodName`
  - `leadTypeUID`

## FAILURE
- If `get_client` fails, ask the user to retry or choose another client.
- If the selected method does not expose `leadTypeUID`, tell the user to choose a different method.

## SUMMARIZE
- After the client and method are both known, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 0 Complete — Client And Method Selected</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, leadTypeUID={leadTypeUID}</current_state><next_instructions>Load and execute Phase 4 from mcp://resource/min-phase-4-create-delivery-account</next_instructions></summary>"`

## NEXT
Load and execute mcp://resource/min-phase-4-create-delivery-account.

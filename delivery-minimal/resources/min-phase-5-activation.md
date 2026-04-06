═══════════════════════════════════════
CURRENT PHASE: Phase 5 — Activation
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Resolve the final activation choice with one explicit decision, then activate the client only if the user explicitly confirms.

## PROCESS
- Present a short plain-text summary that includes:
  - `clientUID`
  - `deliveryMethodUID`
  - `deliveryAccountUID`
  - delivery type
  - delivery schedule
  - price
  - geography choice
  - extra criteria summary
  - whether `useOrder` is enabled
- If `useOrder=true`, add one short note that order setup is still a separate follow-up before order-gated delivery can work.

## ASK (CARD)
- If `activationRetryPending=true`:
  - Call `display_adaptive_card` and render a small choice card with exactly two root `Action.Submit` buttons:
    - `Retry activation` with `data.action="retry-activation"`
    - `Keep inactive` with `data.action="keep-inactive"`
  - Accept only typed equivalents that clearly map to `retry-activation` or `keep-inactive`.
  - If the user chooses `Retry activation`, retain `activationChoice="retry-activation"`, clear `activationRetryPending`, and continue to `USE TOOL`.
  - If the user chooses `Keep inactive`, clear `activationChoice` and `activationRetryPending`, reply in plain text that setup is complete and the client remains inactive for now, and stop.
  - If the user does not choose one of the allowed actions, re-show the same retry card and wait.
- If `activationChoice` is missing and `activationRetryPending` is not true:
  - Call `display_adaptive_card` and render a small choice card with exactly two root `Action.Submit` buttons:
    - `Activate now` with `data.action="activate-now"`
    - `Keep inactive` with `data.action="keep-inactive"`
  - Accept only typed equivalents that clearly map to `activate-now` or `keep-inactive`.
  - If the user chooses `Activate now`, retain `activationChoice="activate-now"` and continue to `USE TOOL`.
  - If the user chooses `Keep inactive`, retain `activationChoice="keep-inactive"` and continue to `USE TOOL`.
  - If the user does not choose one of the allowed actions, re-show the same choice card and wait.

## USE TOOL
### Keep inactive
- If `activationChoice="keep-inactive"`:
  - clear `activationChoice` and `activationRetryPending`
  - reply in plain text that setup is complete and the client remains inactive for now.

### Activate now
- If `activationChoice="activate-now"` or `activationChoice="retry-activation"`:
  - Before calling `update_client`, verify `clientUID`, `companyName`, `email`, `timeZoneName`, and `timeOffset` are all present. If any are missing, stop and tell the user activation cannot continue until the client context is restored.
  - `update_client` is a replace-style mutation. Resend all client fields in `updateClientDto`, even unchanged ones.
  - `updateClientDto` must be a nested native object inside the tool call, not top-level loose fields and not a JSON string.
  - Use a fresh generated password because the Phase 1 password is not retained as durable context.
  - `password` must be an actual fresh generated 14-character string value on every activation attempt.
  - Call `update_client` with:
    - `clientUID={clientUID}`
    - `updateClientDto={companyName={companyName}, email={email}, clientStatus="Active", clientAutomationType="Price", username={email}, password=<fresh random 14-character password>, timeZoneName={timeZoneName}, timeOffset={timeOffset}}`
  - On success:
    - clear `activationChoice` and `activationRetryPending`
    - reply in plain text that setup is complete and active.
  - On failure:
    - clear `activationChoice`
    - retain `activationRetryPending=true`
    - return to `ASK (CARD)` and wait. If retry fails again, show the same retry card again.

## NEXT
End the conversation after the activation choice is resolved.

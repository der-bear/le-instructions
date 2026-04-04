═══════════════════════════════════════
CURRENT PHASE: Phase 5 — Activation
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Present a short plain-text summary of the completed setup, then activate the client only if the user explicitly confirms.

## PROCESS
- Present a plain-text summary that includes:
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

## ASK
- Ask: `Activate now` or `Keep inactive`.
- Prefer a simple choice card, but accept typed fallback.

## TOOL
### Activate now
- `update_client` is a replace-style mutation. Resend all client fields in `updateClientDto`, even unchanged ones.
- Use a fresh generated password because the Phase 1 password is not retained as durable context.
- Call `update_client` with:
  - `clientUID={clientUID}`
  - `updateClientDto={companyName={companyName}, email={email}, clientStatus="Active", clientAutomationType="Price", username={email}, password=<fresh random 14-character password>, timeZoneName={timeZoneName}, timeOffset={timeOffset}}`
- On success, reply in plain text that setup is complete and active.

### Keep inactive
- Reply in plain text that setup is complete and the client remains inactive for now.

## FAILURE
- If activation fails, ask whether to retry activation or keep the client inactive.
- If the user chooses retry, call `update_client` again with the full DTO and a fresh password.

## NEXT
End the conversation after the activation choice is resolved.

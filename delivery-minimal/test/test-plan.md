# Delivery Minimal — Test Plan

## Goal
Validate that the new pack keeps the current platform-facing architecture while removing the brittle summary/state-transfer behavior seen in the other variants.

## Required Scenarios
1. Full setup, Portal, 24/7, all states, keep inactive.
2. Full setup, Portal, specific states, no extra criteria.
3. Full setup, Email, activate at end.
4. Full setup, FTP, no extra criteria.
5. Full setup, Webhook, skip mappings.
6. Full setup, Webhook, JSON schema, auto-detect content type.
7. Full setup, Webhook, large or messy spec prompts for a smaller excerpt.
8. Method-only flow for an existing client.
9. Account-only flow for an existing client and method.
10. Typed-text fallback at every card step instead of button clicks.
11. State-field ambiguity fallback.
12. Extra criteria compile failure offers `fix` or `continue geography-only`.
13. Webhook malformed JSON repair: missing braces, trailing comma, single quotes.
14. Webhook review branch coverage: create, revise URL, revise spec, change content type, skip mapping after review.
15. Geography recovery rejects partial matches and offers only `Re-enter states` or `All states`.
16. Criteria builder enum flow works with ChoiceSet and typed `continue` exits cleanly.
17. Activation failure offers `Retry activation` or `Keep inactive`.

## Acceptance Criteria
- No repeated intro prompts.
- No stall after lead type selection.
- `summarize_history` is used only for scalar phase handoffs:
  - Phase 0 -> 2
  - Phase 1 -> 2
  - Phase 2 -> 3
  - Phase 3 -> 4 for created simple methods
  - Phase 3 Webhook -> 4
  - Phase 4 -> 5
- No summary-review phases or "Continue" summary gates.
- No summary-card gating.
- No table-card dependency.
- No hidden required tool steps between user prompts.
- `deliveryMethodUID` is available when creating the account.
- Geography is always collected before account creation.
- Existing-client entry actions do not load irrelevant phases.
- The criteria builder never mutates the account directly.
- All DTOs are passed as nested native objects, never JSON strings or flattened loose fields.
- `deliveryDays` is always a native 7-entry array of day objects inside method payloads.
- Manual selectors use compact ChoiceSet plus Submit with no extra submit data, and table cards are not required.
- Webhook methods are never created until the user explicitly confirms `Create method`.
- Webhook malformed JSON/XML is repaired once losslessly before the user is asked to revise.
- Webhook `settings` is either `null` or a native mapping array, and `requestBody` is the direct template text with placeholders.
- Partial state matches are rejected instead of silently creating a narrowed geography.
- Account `criteria` is always a native array of criterion objects.
- Enum criteria always use a compact ChoiceSet plus Submit with no extra submit data.
- Activation retries always resend the full DTO with a fresh password.

## Explicit Checks
### Core Flow Checks
- `min-create-single-client` goes:
  - create client
  - select lead type
  - create delivery method
  - create delivery account
  - activation
- Scalar phase context is transferred via summaries rather than relying only on raw conversation history.
- `min-create-delivery-method` ends after the method is created.
- `min-create-delivery-account` ends after the account is created.
- Selection phases retain downstream fields from the chosen row before handoff:
  - client selection resolves one `clientUID`
  - method selection resolves `deliveryMethodUID`, `deliveryMethodName`, and `leadTypeUID`
  - lead type selection resolves `leadTypeUID` and `leadTypeName`

### Webhook Checks
- `get_lead_type` is not called until after the user has pasted a usable spec.
- URL-only specs are rejected.
- Large or repetitive specs trigger `smaller excerpt` or `skip mapping`.
- No mapping preview table is required.
- No connection-test phase is loaded.
- The router does not summarize before loading the webhook phase because `deliveryDays` must stay native.
- The webhook review summary includes content type, mapped count, total extracted count, and unresolved or omitted fields by name.
- Review choices are explicit:
  - `Create method`
  - `Revise URL`
  - `Revise spec`
  - `Change content type`
  - `Skip mapping`
- Revision choices reset only the intended inputs and do not silently discard unrelated webhook state.
- `Re-paste excerpt` returns to the posting-instructions step without clearing the accepted URL or current content type.

### Account Checks
- The account flow asks in this exact order:
  1. price
  2. exclusivity
  3. use order
  4. geography choice
  5. states when needed
  6. extra criteria yes/no
- `All states` creates no state criterion.
- `Continue without state targeting` converts the path to no-state geography and does not create a partial state criterion.
- `Cancel` on state ambiguity returns to geography selection and does not create the account.
- Extra criteria are compiled before `create_delivery_account`.
- The account is created once in the normal path.
- The criteria builder does not summarize before returning to Phase 4 because `compiledCriteria` must stay native.
- If any entered state is unmatched, the full input is rejected and the flow does not silently keep the matched subset.
- Multiple criteria in one message are handled one at a time.
- On criteria failure inside Phase 4, `Fix criteria` reloads Phase 4b and `Continue with geography only` clears extra criteria and proceeds without them.

## Defaults To Verify
- `dayMax=50`
- `hourMax=-1`
- `weekMax=-1`
- `monthMax=-1`
- `automationEnabled=true`
- account `status="Open"`
- activation only on explicit confirmation

# Delivery Minimal — Test Plan

## Goal
Validate that the new pack keeps the current platform-facing architecture while removing the brittle summary/state-transfer behavior seen in the other variants, while still showing the user each created entity before the workflow moves forward.

## Required Scenarios
1. Full setup, Portal, 24/7, all states, keep inactive.
2. Full setup, Portal, specific states, no extra criteria.
3. Full setup, Email, activate at end.
4. Full setup, FTP, no extra criteria.
5. Full setup, Webhook, skip mappings.
6. Full setup, Webhook, explicit JSON content type and mapped review card.
7. Full setup, Webhook, `I'm not sure` auto-detects and confirms content type once.
8. Full setup, Webhook, large or messy spec prompts for a smaller excerpt.
9. Method-only flow for an existing client using Portal or Email.
10. Account-only flow for an existing client and method.
11. Typed-text fallback at every card step instead of button clicks.
12. State-field ambiguity fallback.
13. Extra criteria compile failure offers `fix` or `continue geography-only`.
14. Webhook malformed JSON repair: missing braces, trailing comma, single quotes.
15. Webhook review waits for a single `Continue` acknowledgement before pasted-spec create.
16. Geography recovery rejects partial matches and offers only `Re-enter states` or `All states`.
17. Criteria builder enum flow works with ChoiceSet and typed `continue`/`done` exits cleanly.
18. Activation failure offers `Retry activation` or `Keep inactive`.
19. Client-created preview appears before Phase 2.
20. Simple method-created preview appears before Phase 4 for Portal, Email, and FTP.
21. Webhook pre-create review is followed by a webhook post-create preview before Phase 4.
22. Account-created preview appears before Phase 5.
23. Add-method flow shows the method preview and ends only after `Finish` for Portal, Email, FTP, and Webhook.
24. Add-account flow shows the account preview and ends only after `Finish`.
25. Preview and review cards render with exact required row labels and no extra rows.
26. Normalized preview values are shown instead of raw DTO/internal values.
27. Large webhook mappings render a compact review preview instead of skipping the review.
28. Activation plain-text summary shows the `useOrder=true` follow-up note only when applicable.
29. Required card render failure retries once and then falls back cleanly.
30. Phase 4 loads Phase 4b and stops until the criteria builder returns.
31. Webhook ambiguity resolution asks one delivery field at a time and recomputes counts before review.
32. Webhook summary handoff carries `requestTemplateStatus`, not raw `requestBody`.
33. Activation can fail more than once and still re-offer `Retry activation` or `Keep inactive`.
34. Re-entering Phase 3 through the router clears stale method-attempt state so a new branch does not silently reuse an old schedule, URL, or FTP credentials.
35. Phase 3 typed-text asks stay plain text and do not render as cards with submit-only shells.

## Acceptance Criteria
- No repeated intro prompts.
- No stall after lead type selection.
- `summarize_history` is used only for scalar phase handoffs:
  - Phase 0 -> 2
  - Phase 1 -> 2
  - Phase 2 -> 3
  - Phase 3 Portal -> 4
  - Phase 3 Email -> 4
  - Phase 3 FTP -> 4
  - Phase 3 Webhook -> 4
  - Phase 4 -> 5
- No standalone summary-review phases or generic "Continue" summary gates.
- No standalone summary-card gating.
- Table cards are required only for mapping reviews and preview checkpoints, not as separate phases.
- No hidden required tool steps between user prompts.
- `deliveryMethodUID` is available when creating the account.
- Geography is always collected before account creation.
- Existing-client entry actions do not load irrelevant phases.
- The criteria builder never mutates the account directly.
- All DTOs are passed as nested native objects, never JSON strings or flattened loose fields.
- `deliveryDays` is always a native 7-entry array of day objects inside method payloads.
- Manual selectors use compact ChoiceSet plus Submit with no extra submit data.
- Phase 3 short enumerations use `ActionSet` buttons when the phase explicitly requires a card:
  - delivery type
  - delivery schedule
  - webhook mapping mode
  - webhook content type
- Phase 3 typed-text asks remain plain text:
  - schedule hours
  - webhook URL
  - webhook posting instructions
  - FTP host, username, and password
- No typed-text ask renders as an adaptive card or as a submit-only shell.
- Webhook pasted-spec methods are never created until the user explicitly confirms `Continue` on the mapping preview.
- Webhook skip-mapping creates immediately without an extra confirmation card.
- Webhook malformed JSON/XML is repaired once losslessly before the user is asked to revise.
- Webhook `settings` is either `null` or a native mapping array, and `requestBody` is the direct template text with placeholders.
- Raw webhook `requestBody` is not serialized through summaries; later phases carry only `requestTemplateStatus`.
- Partial state matches are rejected instead of silently creating a narrowed geography.
- Account `criteria` is always a native array of criterion objects.
- Enum criteria use a compact ChoiceSet plus Submit with no extra submit data when the user has not already provided a concrete enum value.
- Activation retries always resend the full DTO with a fresh password.
- Successful create phases show an adaptive-card table created-state preview before handoff.
- Post-create preview acknowledgements do not trigger a duplicate create mutation.
- The next phase is not loaded until the preview is acknowledged.
- Preview state is transient and is not serialized through `summarize_history`.
- Preview values come from the validated created entity, not inferred state.
- Typed acknowledgement responses work the same as card clicks.
- Simple-method preview replay is guarded by transient preview state, not by reusing a previously retained `deliveryMethodUID`.
- FTP previews redact the password as `set`.
- Webhook mapping review renders as an adaptive-card table.
- Activation presents a short plain-text summary plus a simple choice card.
- Preview and review cards are tested at the contract level, not by full raw-card snapshots.

## Explicit Checks
### Core Flow Checks
- `min-create-single-client` goes:
  - create client
  - client-created preview
  - select lead type
  - create delivery method
  - method-created preview
  - create delivery account
  - account-created preview
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
- The explicit JSON path and the `I'm not sure` auto-detect path are both covered separately.
- URL-only specs are rejected.
- Large or repetitive specs trigger `smaller excerpt` or `skip mapping`.
- Webhook mapping review uses an adaptive-card table rather than plain text.
- No connection-test phase is loaded.
- The router does not summarize before loading Portal, Email, FTP, or Webhook.
- The router never calls `create_delivery_method`; creation happens only in the method-specific resource.
- The router clears stale Phase 3 method-attempt state before loading the selected branch.
- The Phase 3 delivery-type selector renders as one `ActionSet` with exactly four buttons: `Portal`, `Webhook`, `Email`, and `FTP`.
- Each method-specific resource collects and builds its own `deliveryDays` payload locally instead of receiving a native schedule handoff from the router.
- Webhook and FTP typed-text asks stay plain text and do not render as cards with a standalone `Submit` button.
- Webhook short-enumeration choices render as `ActionSet` cards for:
  - schedule
  - `Provide instructions` / `Skip mapping`
  - `URL Encoded` / `JSON` / `XML` / `I'm not sure`
  - auto-detect confirmation
  - unusable-spec recovery
- The webhook review summary includes content type, mapped count, total extracted count, and not-mapped count.
- The webhook review summary includes mapped field pairs only.
- The webhook review card contains:
  - one summary table with exact rows for content type, mapped fields, and not-mapped count
  - one mapping table with exact header text `Delivery Field | System Field | Status`
  - mapped rows in extracted field order
  - `Mapped` as the status text for every mapped row
- The webhook review card uses one explicit `Continue` action.
- `Re-paste excerpt` returns to the posting-instructions step without clearing the accepted URL or current content type.
- After webhook creation succeeds, a second post-create method preview is shown before handoff or final completion.
- If the webhook review would exceed the comfortable size cap, the flow still shows a compact review preview instead of skipping directly to create.
- Ambiguous mappings are resolved one delivery field at a time before the review card is shown.
- `totalCount` still includes unmapped extracted fields, and nested JSON/XML paths stay explicit in the mapping contract.

### Account Checks
- The account flow asks in this exact order:
  1. price
  2. exclusivity
  3. use order
  4. geography choice
  5. states when needed
  6. extra criteria yes/no
- Phase 4 short choices render as `ActionSet` cards for:
  - `Exclusive` / `Shared`
  - `Yes` / `No` for use order
  - `All states` / `Specific states`
  - `Yes` / `No` for extra criteria
- `All states` creates no state criterion.
- `Continue without state targeting` converts the path to no-state geography and does not create a partial state criterion.
- `Cancel` on state ambiguity returns to geography selection and does not create the account.
- Extra criteria are compiled before `create_delivery_account`.
- The account is created once in the normal path.
- The criteria builder does not summarize before returning to Phase 4 because `compiledCriteria` must stay native.
- Phase 4 stops immediately when it loads Phase 4b and does not fall through into account creation on the same turn.
- If any entered state is unmatched, the full input is rejected and the flow does not silently keep the matched subset.
- Multiple criteria in one message are handled one at a time.
- On criteria failure inside Phase 4, `Fix criteria` reloads Phase 4b and `Continue with geography only` clears extra criteria and proceeds without them.
- Criteria parsing handles deterministic comparator phrases like `at least`, `more than`, `less than`, and `between`.
- Typed `done` exits the criteria loop the same way as typed `continue`.
- The account is not summarized or finalized until the post-create account preview is acknowledged.

### Preview Checks
- Phase 1 waits for `Continue to Lead Type` before summarizing and loading Phase 2.
- Simple method phases wait for `Continue to Delivery Account` or `Finish` before summarizing or completing.
- Webhook phases keep the pre-create mapping review and then wait for the post-create method preview acknowledgement.
- Phase 4 waits for `Continue to Activation` or `Finish` before summarizing or completing.
- If a preview is pending, the phase re-shows that preview instead of re-running the create mutation.
- Client, method, and account previews render as adaptive-card tables.
- Preview cards contain the exact required row labels for their phase and no extra rows.
- Preview action labels and action counts are exact for the pending phase.
- Typed equivalents map only to the pending card's allowed action set.
- Preview cards show normalized display values:
  - no raw `HttpPost` or `EMail`
  - no raw boolean `true/false`
  - price shows as `$NN.NN`
  - geography shows a human-readable state list when targeted
- Activation plain-text summary shows the order follow-up note only when `useOrder=true`.
- Activation choice handling is stateful: once the user chooses `Activate now` or `Keep inactive`, the phase handles that choice instead of re-asking from the start.
- Activation failure can repeat and continues to offer the same retry-or-keep-inactive choice.
- If a required card fails to render, the system retries once and then falls back without sending both card and plain text in the same turn.

## Defaults To Verify
- `dayMax=50`
- `hourMax=-1`
- `weekMax=-1`
- `monthMax=-1`
- `automationEnabled=true`
- account `status="Open"`
- activation only on explicit confirmation

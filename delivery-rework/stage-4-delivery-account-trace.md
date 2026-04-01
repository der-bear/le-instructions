# Stage 4 Delivery Account Trace

## Purpose

This trace records the reachable branch graph for the rewritten delivery-account and activation unit in `delivery-rework/`.

## Resource Graph

Entry:
- `mcp://resource/phase-5-create-delivery-account`

Account setup:
- `phase-5-create-delivery-account` collects account basics, state targeting, and criteria context
- `phase-5-create-delivery-account` routes to `mcp://resource/phase-5-criteria-builder`
- `phase-5-criteria-builder` owns the initial criteria prompt, the full criteria loop, the delivery-account creation, and the summarize_history handoff from `anchor_delivery_account_start`

Post-create:
- `phase-6-delivery-account-summary`
  - `flowIntent = full-setup` -> `mcp://resource/phase-7-client-summary`
  - otherwise terminal done message
- `phase-7-client-summary` routes to `mcp://resource/phase-8-activation`
- `phase-8-activation`
  - `Activate` -> update client active
  - `Keep Inactive` -> terminal inactive message

## Phase 5 Router Trace

Resource:
- `delivery-rework/resources/phase-5-create-delivery-account.md`

States:
1. Missing `price`
   - prompt for price
   - STOP AND YIELD
2. Missing `isExclusive`
   - prompt `Exclusive | Shared`
   - STOP AND YIELD
3. Missing `useOrder`
   - prompt `Yes | No`
   - STOP AND YIELD
4. Missing `leadFields`
   - call `get_lead_type`
5. Missing `stateFieldUID`
   - detect from `leadFields`
   - if ambiguous, prompt for state-field choice and STOP AND YIELD
6. Missing `targetStates`
   - prompt for target states
   - STOP AND YIELD
7. Missing criteria context
   - build `suggestedFields`, `extraFields`, `extraFieldCount`, `leadFieldsMap`
   - initialize `parsedCriteriaList`, `criteriaSummaryList`, and `criteriaPromptStage`
8. Criteria context ready
   - route to `mcp://resource/phase-5-criteria-builder`

## Phase 5 Criteria Trace

Resource:
- `delivery-rework/resources/phase-5-criteria-builder.md`

States:
1. Initial criteria prompt not yet shown
   - display recommended fields
   - prompt `Show more fields | Skip` when extra fields exist
   - STOP AND YIELD
2. Initial response processing
   - `Skip` -> `additionalCriteria="None"` -> build payload
   - `Show more fields` -> display extra fields and STOP AND YIELD
   - typed criterion -> parse criterion
3. Criteria loop prompt not yet shown
   - prompt `Show more fields | Continue` or `Continue`
   - STOP AND YIELD
4. Criteria loop response processing
   - `Continue` -> flatten `criteriaSummaryList` into `additionalCriteria`
   - `Show more fields` -> display extra fields and STOP AND YIELD
   - typed criterion -> parse criterion
5. Typed criterion parse
   - field not matched -> re-prompt for field name and STOP AND YIELD
   - enum value unresolved -> prompt compact choice set and STOP AND YIELD
   - valid scalar or resolved criterion -> append to `parsedCriteriaList` and `criteriaSummaryList`
6. Enum selection resolution
   - append resolved enum criterion to both arrays
7. Payload build and account creation
   - call `get_usa_states`
   - build state criterion
   - prepend the state criterion to the final payload and preserve every item from `parsedCriteriaList`
   - call `create_delivery_account`
   - summarize from `anchor_delivery_account_start`

## Phase 6 Trace

Resource:
- `delivery-rework/resources/phase-6-delivery-account-summary.md`

States:
1. Summary card not yet acknowledged
   - display account summary card
   - `Continue` for `full-setup`
   - `Done` otherwise
   - STOP AND YIELD
2. `flowIntent = full-setup` and `Continue`
   - route to `mcp://resource/phase-7-client-summary`
3. non-`full-setup` and `Done`
   - terminal `delivery account is ready to use` message

## Phase 7 Trace

Resource:
- `delivery-rework/resources/phase-7-client-summary.md`

States:
1. Final summary card not yet acknowledged
   - display final client summary card
   - actions: `Activate`, `Keep Inactive`
   - STOP AND YIELD
2. Choice captured
   - route to `mcp://resource/phase-8-activation`

## Phase 8 Trace

Resource:
- `delivery-rework/resources/phase-8-activation.md`

States:
1. `Activate`
   - call `update_client` with `updateClientDto={...}`
   - success -> terminal active message
   - failure -> prompt retry selector and STOP AND YIELD
2. `Retry activation`
   - re-run the same activation call
3. `Keep Inactive`
   - terminal inactive message

## Open Review Targets

- Phase 5 criteria resource still needs reviewer confirmation that its loop conditions are explicit enough for mechanical re-entry.
- Phase 5 state-field ambiguity handling needs a concrete card shape in implementation review.
- Phase 8 should be checked once more against the live tool schema to confirm the DTO envelope matches expected tool semantics.

═══════════════════════════════════════
CURRENT PHASE: Phase 4 — Create Delivery Account
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect the account inputs in a fixed visible order, compile geography and optional extra criteria before creation, create the account once, show the created account, then summarize the result.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.
Any item without an explicit card instruction is asked in plain text.

1. `price`
2. `isExclusive`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `Exclusive`
     - `Shared`
3. `useOrder`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `Yes`
     - `No`
4. `geographyChoice`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `All states`
     - `Specific states`
5. `targetStates`
   - Ask only if `geographyChoice="Specific states"`
6. `addExtraCriteriaChoice`
   - Ask using `display_adaptive_card` with exactly one `ActionSet` containing:
     - `Yes`
     - `No`

Accept typed equivalents for every card choice.

Rules:
- Do not call `get_lead_type` before geography is collected.
- If `useOrder=true`, give one short note that order setup is separate and still required later.

## ROUTE
- If `addExtraCriteriaChoice="Yes"` and `extraCriteriaStatus!="done"`, load and execute mcp://resource/min-phase-4b-criteria-builder, then stop this phase immediately. Do not continue `BUILD` or `USE TOOL` on the same turn.

## BUILD
- Normalize `price` to a positive number with two decimals.
- Normalize `targetStates` to uppercase USPS abbreviations when possible.
- `All states` means no state criterion.
- If `addExtraCriteriaChoice="No"`:
  - retain `compiledCriteria=[]`
  - retain `additionalCriteriaSummary="None"`
  - retain `extraCriteriaStatus="done"`

## USE TOOL
After all visible inputs are known and any optional criteria work is complete, run one contiguous tool block.
- `createDeliveryAccountDto` must be a nested native object, not a JSON string.
- `criteriaPayload` must be a native array of criterion objects, not a JSON string.

1. Verify that `deliveryMethodUID` and `leadTypeUID` are both present before any create call. If either value is missing, stop, tell the user the delivery-method context is missing, and do not guess or create the account.
2. If `geographyChoice="Specific states"` or lead-field metadata is otherwise needed, call `get_lead_type(leadTypeUID)` and retain `leadTypeName` and `leadFields`.
3. If `geographyChoice="Specific states"`, detect the state field in this priority order:
   - `leadFieldSpecialBit` in `State` or `StandardState`
   - exact `leadFieldName="state"` case-insensitive
   - `leadFieldName` contains `state`
4. If the match resolves cleanly, retain `stateFieldUID` immediately.
5. If the match is ambiguous, retain `stateFieldCandidates` and `stateFieldDisambiguationPending=true`, ask one focused fallback question naming the candidate fields, and wait.
6. After the fallback reply, resolve and retain exactly one `stateFieldUID`, then clear `stateFieldCandidates` and `stateFieldDisambiguationPending`.
7. If ambiguity still remains, offer only:
   - `Continue without state targeting`
   - `Cancel`
8. If no viable state field is found at all, offer the same two choices and apply the same outcomes.
9. Ambiguity outcomes:
   - `Continue without state targeting` → set `geographyChoice="All states"`, clear `targetStates`, `stateFieldUID`, `stateFieldCandidates`, and `stateFieldDisambiguationPending`, and continue with no state criterion
   - `Cancel` → clear `geographyChoice`, `targetStates`, `stateFieldUID`, `stateFieldCandidates`, and `stateFieldDisambiguationPending`, return to the geography-choice step, and do not create the account
10. If `geographyChoice="Specific states"`, call `get_usa_states()` and match the normalized input.
11. Reject partial state matches. If any entered state is unmatched, treat the full input as invalid rather than silently using only the matched subset.
12. If no states match or the input is only partially valid, offer only:
   - `Re-enter states`
   - `All states`
13. Invalid-state outcomes:
   - `Re-enter states` → clear `targetStates`, return to the states step, and do not create the account yet
   - `All states` → set `geographyChoice="All states"`, clear `targetStates` and `stateFieldUID`, and continue with no state criterion
14. Build `criteriaPayload`:
   - when `geographyChoice="Specific states"`, dedupe matched states while preserving the user's normalized order
   - prepend the state criterion when needed as `{leadFieldUID: stateFieldUID, type:"FieldValue", operator:"In", value:"<pipe-delimited stateUID string>"}`
   - append `compiledCriteria` in the order they were collected
   - use `[]` when there are no criteria
15. Call `create_delivery_account` with:
   - `clientUID={clientUID}`
   - `createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
16. Retain:
   - `deliveryAccountUID`
   - `previewCheckpoint="account-created"`
   - `previewNextAction="activation"`
   - `previewEntityUID={deliveryAccountUID}`
17. If `flowIntent="add-account"` and the account is created successfully, overwrite `previewNextAction="finish"` before showing the preview.
18. Validate that `deliveryAccountUID` is a positive value. If it is missing or invalid, treat the create step as failed and retry or ask the user how to proceed.

## FAILURE
- If criteria compilation is ambiguous or invalid, offer only:
  - `Fix criteria`
  - `Continue with geography only`
- Criteria-failure outcomes:
  - `Fix criteria` → load and execute mcp://resource/min-phase-4b-criteria-builder again. Do not create the account in the same turn.
  - `Continue with geography only` → clear `compiledCriteria`, retain `additionalCriteriaSummary="None"`, retain `extraCriteriaStatus="done"`, retain `addExtraCriteriaChoice="No"`, and continue Phase 4 with geography-only criteria.
- If the create call can be safely repaired, repair once silently.
- If it still fails, ask whether to retry with the same inputs or revise the last account setting.

## SHOW PREVIEW (CARD)
- After the account is created successfully, call `display_adaptive_card` and render exactly one `KeyValuePreviewCard` titled `Account created`.
- Use exactly these rows in this order:
  - `Account UID | {deliveryAccountUID}`
  - `Delivery Method | {deliveryMethodName}`
  - `Price | <normalized price display>`
  - `Exclusivity | <normalized exclusivity label>`
  - `Use Order | <normalized use-order label>`
  - `Geography | <normalized geography display>`
  - `Additional Criteria | {additionalCriteriaSummary}`
- Normalize display values:
  - `Price | $NN.NN`
  - `Exclusivity | Exclusive` or `Shared`
  - `Use Order | Yes` or `No`
  - `Geography | All states` or `Specific states: <comma-delimited normalized state codes>`
- Do not add extra rows.
- Actions:
  - if `previewNextAction="activation"`, render `Continue to Activation` with `data.action="continue-activation"`
  - if `previewNextAction="finish"`, render `Finish` with `data.action="finish-account"`
- Accept only typed equivalents that clearly map to the currently pending action token.
- If the user acknowledges:
  - clear `previewCheckpoint`, `previewNextAction`, and `previewEntityUID`
  - if `flowIntent="full-setup"`, continue to `SUMMARIZE` and `NEXT`
  - if `flowIntent="add-account"`, reply in plain text with the new `deliveryAccountUID` and stop
- If the user does not acknowledge, re-show the same preview and wait.
- Do not call `create_delivery_account` again while this preview is pending.

## SUMMARIZE
- If the account was created successfully, the preview was acknowledged, and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 4 Complete — Delivery Account Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, mimeContentType={mimeContentType}, requestTemplateStatus={requestTemplateStatus}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}, deliveryAccountUID={deliveryAccountUID}, price={price}, geographyChoice={geographyChoice}, targetStates={targetStates}, additionalCriteriaSummary={additionalCriteriaSummary}, isExclusive={isExclusive}, useOrder={useOrder}</current_state><next_instructions>Load and execute Phase 5 from mcp://resource/min-phase-5-activation</next_instructions></summary>"`

## NEXT
- After the preview is acknowledged:
  - if `flowIntent="full-setup"`, load and execute mcp://resource/min-phase-5-activation
  - if `flowIntent="add-account"`, reply in plain text with the new `deliveryAccountUID` and stop

═══════════════════════════════════════
CURRENT PHASE: Phase 4 — Create Delivery Account
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect the account inputs in a fixed visible order, compile geography and optional extra criteria before creation, create the account once, then summarize the result.

## ASK
Collect missing inputs in this exact order. Ask only the first missing item, then wait.

1. `price`
2. `isExclusive`
   - Ask: `Exclusive` or `Shared`
3. `useOrder`
   - Ask: `Yes` or `No`
4. `geographyChoice`
   - Ask: `All states` or `Specific states`
5. `targetStates`
   - Ask only if `geographyChoice="Specific states"`
6. `addExtraCriteriaChoice`
   - Ask: `Yes` or `No`

Rules:
- Do not call `get_lead_type` before geography is collected.
- If `useOrder=true`, give one short note that order setup is separate and still required later.

## NEXT
- If `addExtraCriteriaChoice="Yes"` and `extraCriteriaStatus` is not complete, load and execute mcp://resource/min-phase-4b-criteria-builder.

## PROCESS
- Normalize `price` to a positive number with two decimals.
- Normalize `targetStates` to uppercase USPS abbreviations when possible.
- `All states` means no state criterion.
- If extra criteria were skipped, treat `additionalCriteriaSummary="None"` unless a later phase already set it.

## TOOL
After all visible inputs are known and any optional criteria work is complete, run one contiguous tool block.
- `createDeliveryAccountDto` must be a nested native object, not a JSON string.
- `criteriaPayload` must be a native array of criterion objects, not a JSON string.

1. Verify that `deliveryMethodUID` and `leadTypeUID` are both present before any create call. If either value is missing, stop, tell the user the delivery-method context is missing, and do not guess or create the account.
2. If `geographyChoice="Specific states"` or lead-field metadata is otherwise needed, call `get_lead_type(leadTypeUID)` and retain `leadTypeName` and `leadFields`.
3. If `geographyChoice="Specific states"`, detect the state field in this priority order:
   - `leadFieldSpecialBit` in `State` or `StandardState`
   - exact `leadFieldName="state"` case-insensitive
   - `leadFieldName` contains `state`
4. If the match is ambiguous, ask one focused fallback question.
5. If ambiguity still remains, offer only:
   - `Continue without state targeting`
   - `Cancel`
6. Ambiguity outcomes:
   - `Continue without state targeting` → set `geographyChoice="All states"`, clear `targetStates`, and continue with no state criterion
   - `Cancel` → clear `geographyChoice` and `targetStates`, return to the geography-choice step, and do not create the account
7. If `geographyChoice="Specific states"`, call `get_usa_states()` and match the normalized input.
8. Reject partial state matches. If any entered state is unmatched, treat the full input as invalid rather than silently using only the matched subset.
9. If no states match or the input is only partially valid, offer only:
   - `Re-enter states`
   - `All states`
10. Invalid-state outcomes:
   - `Re-enter states` → clear `targetStates`, return to the states step, and do not create the account yet
   - `All states` → set `geographyChoice="All states"`, clear `targetStates`, and continue with no state criterion
11. Build `criteriaPayload`:
   - prepend the state criterion when needed as `{leadFieldUID: stateFieldUID, type:"FieldValue", operator:"In", value:"<pipe-delimited stateUID string>"}`
   - append `compiledCriteria` if any
   - use `[]` when there are no criteria
12. Call `create_delivery_account` with:
   - `clientUID={clientUID}`
   - `createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
13. Retain:
   - `deliveryAccountUID`
   - `price`
   - `geographyChoice`
   - `targetStates`
   - `additionalCriteriaSummary`
   - `isExclusive`
   - `useOrder`
14. Validate that `deliveryAccountUID` is a positive value. If it is missing or invalid, treat the create step as failed and retry or ask the user how to proceed.

## FAILURE
- If criteria compilation is ambiguous or invalid, offer only:
  - `Fix criteria`
  - `Continue with geography only`
- Criteria-failure outcomes:
  - `Fix criteria` → load and execute mcp://resource/min-phase-4b-criteria-builder again. Do not create the account in the same turn.
  - `Continue with geography only` → clear `compiledCriteria`, retain `additionalCriteriaSummary="None"`, retain `extraCriteriaStatus="done"`, retain `addExtraCriteriaChoice="No"`, and continue Phase 4 with geography-only criteria.
- If the create call can be safely repaired, repair once silently.
- If it still fails, ask whether to retry with the same inputs or revise the last account setting.

## SUMMARIZE
- If the account was created successfully and `flowIntent="full-setup"`, call `summarize_history`.
- Use:
  - `start_anchor_substring="DELIVERY_SETUP_START"`
  - `summarization_text="<summary><completed>Phase 4 Complete — Delivery Account Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}, deliveryAccountUID={deliveryAccountUID}, price={price}, geographyChoice={geographyChoice}, targetStates={targetStates}, additionalCriteriaSummary={additionalCriteriaSummary}, isExclusive={isExclusive}, useOrder={useOrder}</current_state><next_instructions>Load and execute Phase 5 from mcp://resource/min-phase-5-activation</next_instructions></summary>"`

## NEXT
- If `flowIntent="full-setup"` and the account was created successfully, load and execute mcp://resource/min-phase-5-activation.
- If `flowIntent="add-account"` and the account was created successfully, reply in plain text with the new `deliveryAccountUID` and stop.

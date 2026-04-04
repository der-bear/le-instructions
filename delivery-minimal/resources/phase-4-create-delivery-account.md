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

1. If `geographyChoice="Specific states"` or lead-field metadata is otherwise needed, call `get_lead_type(leadTypeUID)` and retain `leadTypeName` and `leadFields`.
2. If `geographyChoice="Specific states"`, detect the state field in this priority order:
   - `leadFieldSpecialBit` in `State` or `StandardState`
   - exact `leadFieldName="state"` case-insensitive
   - `leadFieldName` contains `state`
3. If the match is ambiguous, ask one focused fallback question.
4. If ambiguity still remains, ask whether to continue without state targeting or cancel.
5. If `geographyChoice="Specific states"`, call `get_usa_states()` and match the normalized input.
6. If no states match, ask the user to re-enter the states or switch to `All states`.
7. Build `criteriaPayload`:
   - prepend the state criterion when needed
   - append `compiledCriteria` if any
   - use `[]` when there are no criteria
8. Call `create_delivery_account` with:
   - `clientUID={clientUID}`
   - `createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
9. Retain:
   - `deliveryAccountUID`
   - `price`
   - `geographyChoice`
   - `targetStates`
   - `additionalCriteriaSummary`
   - `isExclusive`
   - `useOrder`
10. Validate that `deliveryAccountUID` is a positive value. If it is missing or invalid, treat the create step as failed and retry or ask the user how to proceed.

## FAILURE
- If criteria compilation is ambiguous or invalid, offer only:
  - `Fix criteria`
  - `Continue with geography only`
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

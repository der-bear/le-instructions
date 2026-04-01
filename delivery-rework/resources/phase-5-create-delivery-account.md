# Phase 5: Create Delivery Account

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 5 resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to collect the delivery-account basics (price, exclusivity, order system), detect the state field, collect target states, and either skip to account creation or route to the criteria builder.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Collect Price (Do this first)**
* IF price is missing:
  1. Prompt the user exactly as follows: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead."
  2. **STOP AND YIELD.** Do not proceed. You must wait for the user to respond.
  - Normalize the price: extract numbers from user input (e.g., "$25" → 25.00), format to 2 decimals, positive only.

**State 2: Collect Exclusivity and Order System**
* IF isExclusive is not yet known:
  1. Prompt the user exactly as follows: "Will this client receive exclusive or shared leads?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Exclusive" | "Shared".
  3. **STOP AND YIELD.** You must wait for the user to respond.
  - Map: "Exclusive" → isExclusive=true, "Shared" → isExclusive=false.

* IF useOrder is not yet known:
  1. Prompt the user exactly as follows: "Would you like to enable the Order System for this client?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Yes" | "No".
  3. **STOP AND YIELD.** You must wait for the user to respond.
  - Map: "Yes" → useOrder=true, "No" → useOrder=false.

**State 3: Load Lead Fields, Detect State Field, Collect Target States**
* IF leadFields is missing:
  1. Call the get_lead_type(leadTypeUID) tool and retain: leadTypeName, leadFields.
  2. If the tool fails, prompt: "I ran into an issue loading the lead type fields.\n\nPlease try again." **STOP AND YIELD.**

* IF stateFieldUID is missing:
  1. Detect the state field from leadFields using this priority:
     - Priority 1: leadFieldSpecialBit in {'State', 'StandardState'}
     - Priority 2: leadFieldName = "state" (case-insensitive)
     - Priority 3: leadFieldName contains "state" (substring)
     Do not process lower tiers once matched. If confidence is low (<5%), confirm the correct field with the user.
  2. Retain stateFieldUID.
  3. If ambiguous, prompt the user to confirm which field represents the state. Present a compact selector. **STOP AND YIELD.**

* IF targetStates is missing:
  1. Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
  2. **STOP AND YIELD.** You must wait for the user to respond.
  - Normalize target states to uppercase USPS codes (e.g., California → CA). Accept any separator.

**State 4: Ask About Additional Criteria**
* IF additionalCriteriaChoice is not yet known:
  1. Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Add criteria" | "Skip".
  3. **STOP AND YIELD.** Do not proceed. You must wait for the user to respond.

**State 5: Skip Criteria — Create Account and Summarize**
* IF the user selected "Skip" OR said "none", "skip", or "no" AND deliveryAccountUID is missing:
  1. Retain additionalCriteria = "None".
  2. Call the get_usa_states tool. Match the normalized targetStates to the returned USA-state list (match abbreviation against targetStates, exact match, case-insensitive). Collect the corresponding stateUID values.
  3. Create stateUIDArray by collecting all matched stateUID values. Serialize as pipe-delimited string: stateUIDArray.join('|').
  4. Build criteriaPayload with only the state criterion:
     `[{leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: "<pipe-delimited stateUID string>"}]`
  5. Call the create_delivery_account tool with these defaults:
     `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
  6. If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.**
  7. Retain: deliveryAccountUID, price, targetStates, additionalCriteria, isExclusive, useOrder.
  8. Immediately call the summarize_history tool.

* IF the user selected "Add criteria" OR said "yes", "add", or "criteria":
  - Load mcp://resource/phase-5-criteria-builder (no summarize — leadFields stays in working memory)

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
## State
flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}, connectionTestMode={connectionTestMode}, deliveryAccountUID={deliveryAccountUID}, price={price}, targetStates={targetStates}, additionalCriteria={additionalCriteria}, isExclusive={isExclusive}, useOrder={useOrder}

## Next
mcp://resource/phase-6-delivery-account-summary
```

═══════════════════════════════════════
CURRENT PHASE: Phase 5 — Delivery Account Setup
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to collect account basics, create the delivery account with state-targeting criteria, then ask about additional criteria.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Collect Price (Do this first)**
* IF price is missing:
  1. Prompt the user exactly as follows: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead."
  2. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - Normalize the price: extract numbers (e.g., "$25" → 25.00), 2 decimals, positive only.

**State 2: Collect Exclusivity and Order System**
* IF isExclusive is not yet known:
  1. Prompt the user exactly as follows: "Will this client receive exclusive or shared leads?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Exclusive" | "Shared".
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - Map: "Exclusive" → isExclusive=true, "Shared" → isExclusive=false.

* IF useOrder is not yet known:
  1. Prompt the user exactly as follows: "Would you like to enable the Order System for this client?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Yes" | "No".
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - Map: "Yes" → useOrder=true, "No" → useOrder=false.

**State 3: Load Lead Fields, Detect State Field, Collect Target States**
* IF leadFields is missing:
  1. Call the get_lead_type(leadTypeUID) tool and retain: leadTypeName, leadFields.
  2. If the tool fails, prompt: "I ran into an issue loading the lead type fields.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

* IF stateFieldUID is missing:
  1. Detect the state field from leadFields:
     - Priority 1: leadFieldSpecialBit in {'State', 'StandardState'}
     - Priority 2: leadFieldName = "state" (case-insensitive)
     - Priority 3: leadFieldName contains "state" (substring)
     Do not process lower tiers once matched. If confidence is low, confirm with the user.
  2. Retain stateFieldUID.
  3. If ambiguous, prompt the user to confirm which field represents the state. **STOP AND YIELD.** Do not hallucinate data.

* IF targetStates is missing:
  1. Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
  2. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - Normalize to uppercase USPS codes (e.g., California → CA).

**State 4: Create Account with State-Only Criteria**
* IF deliveryAccountUID is missing:
  1. Call the get_usa_states tool. Match normalized targetStates to the returned list (match abbreviation, exact, case-insensitive). Collect stateUID values.
  2. Serialize as pipe-delimited string: stateUIDArray.join('|').
  3. Build criteriaPayload with the state criterion:
     `[{leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: "<pipe-delimited stateUID string>"}]`
  4. Call the create_delivery_account tool with these defaults:
     `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
  5. If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  6. Retain: deliveryAccountUID, price, targetStates, isExclusive, useOrder.

**State 5: Ask About Additional Criteria**
* IF additionalCriteriaChoice is not yet known:
  1. Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  2. Present the choice using display_adaptive_card with an ActionSet: "Add criteria" | "Skip".
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF the user selected "Skip" OR said "none", "skip", or "no":
  1. Retain additionalCriteria = "None".
  2. Immediately call the summarize_history tool.

* IF the user selected "Add criteria" OR said "yes", "add", or "criteria":
  - Load mcp://resource/rw-phase-5c-criteria-builder (no summarize — leadFields stays in working memory)

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 5 Complete — Delivery Account Created

# Current System State
* Flow Intent: {flowIntent}
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: {clientStatus}
* Time Zone Name: {timeZoneName}
* Time Offset: {timeOffset}
* Lead Type UID: {leadTypeUID}
* Lead Type Name: {leadTypeName}
* Delivery Method UID: {deliveryMethodUID}
* Delivery Method Name: {deliveryMethodName}
* Delivery Type: {deliveryType}
* Delivery Address: {deliveryAddress}
* FTP User: {ftpUser}
* FTP Password: {ftpPassword}
* MIME Content Type: {mimeContentType}
* Request Body: {requestBody}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: {mappedCount}
* Total Count: {totalCount}
* Connection Test Mode: {connectionTestMode}
* Delivery Account UID: {deliveryAccountUID}
* Price: {price}
* Target States: {targetStates}
* Additional Criteria: {additionalCriteria}
* Is Exclusive: {isExclusive}
* Use Order: {useOrder}

# Next Instructions
→ Load and execute Phase 6 at mcp://resource/rw-phase-6-delivery-account-summary
```


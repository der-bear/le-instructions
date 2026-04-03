═══════════════════════════════════════
CURRENT PHASE: Phase 5 — Delivery Account Setup
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to collect account basics, create the delivery account with state-targeting criteria, then ask about additional criteria.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Collect Account Basics**
Execute these steps in order, one per turn:

Step 1: Collect Price
  Prompt the user exactly as follows: "Finally, let's set up your Delivery Account.\n\nPlease provide the price per lead."
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Normalize the price: extract numbers (e.g., "$25" → 25.00), 2 decimals, positive only.

Step 2: Collect Exclusivity
  Prompt the user exactly as follows: "Will this client receive exclusive or shared leads?"
  Present using display_adaptive_card with an ActionSet: "Exclusive" | "Shared". MUST use ActionSet buttons, NOT radio buttons.
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Map: "Exclusive" → isExclusive=true, "Shared" → isExclusive=false. If the user types "exclusive" or "shared" as text, accept it directly.

Step 3: Collect Order System
  Prompt the user exactly as follows: "Would you like to enable the Order System for this client?"
  Present using display_adaptive_card with an ActionSet: "Yes" | "No".
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Map: "Yes" → useOrder=true, "No" → useOrder=false. If the user types "yes" or "no" as text, accept it directly.

**State 2: Load Fields and Collect Target States**
Execute these steps in order:

Step 1: Load Lead Fields (silent — no user prompt)
  Call the get_lead_type(leadTypeUID) tool and retain: leadTypeName, leadFields.
  If the tool fails, prompt: "I ran into an issue loading the lead type fields.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

Step 2: Detect State Field (silent — no user prompt)
  Detect the state field from leadFields:
  - Priority 1: leadFieldSpecialBit in {'State', 'StandardState'}
  - Priority 2: leadFieldName = "state" (case-insensitive)
  - Priority 3: leadFieldName contains "state" (substring)
  Do not process lower tiers once matched. If confidence is below 5%, confirm with the user which field represents the US state. **STOP AND YIELD.** If confirming.
  Retain stateFieldUID.

Step 3: Collect Target States
  Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Normalize to uppercase USPS codes (e.g., California → CA).

**State 3: Create Account with State Criteria**
Execute these steps in order (silent — no user prompt unless failure):

Step 1: Match Target States to UIDs
  Call the get_usa_states tool. Match normalized targetStates to the returned list (match abbreviation, exact, case-insensitive). Collect ALL matched stateUID values into stateUIDArray.
  If no states matched, prompt: "None of those matched valid US states. Please re-enter your target states (e.g., CA, AZ, TX)." **STOP AND YIELD.** Re-normalize and re-match.

Step 2: Build Criteria Payload
  Serialize stateUIDArray as pipe-delimited string: stateUIDArray.join('|').
  Build criteriaPayload with the state criterion as the FIRST element:
  `[{leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: "<pipe-delimited stateUID string>"}]`

Step 3: Create Delivery Account
  Call the create_delivery_account tool with these defaults:
  `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
  CRITICAL: createDeliveryAccountDto must be passed as a native object, NOT a JSON string. criteria must be an array of criterion objects, NOT a JSON string.
  If the tool fails, repair the payload and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

Step 4: Validate and Retain
  Verify deliveryAccountUID is a positive integer > 0. If 0 or null, treat as failure and retry.
  Retain: deliveryAccountUID, price, targetStates, isExclusive, useOrder.

**State 4: Criteria Gate**

Step 1: Check if Already Decided
  IF additionalCriteriaChoice is already set to "Skip" (returned from Phase 5c):
    Retain additionalCriteria = "None".
    Immediately call the summarize_history tool.
    (Do not re-prompt — the user already chose to skip in Phase 5c.)

Step 2: Ask About Additional Criteria
  Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  Present using display_adaptive_card with an ActionSet: "Add criteria" | "Skip". MUST use display_adaptive_card.
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

Step 3: Route
  IF the user selected "Skip" OR said "none", "skip", or "no":
    Retain additionalCriteria = "None".
    Immediately call the summarize_history tool.

  IF the user selected "Add criteria" OR said "yes", "add", or "criteria":
    Load mcp://resource/rw-phase-5c-criteria-builder (no summarize — leadFields stays in working memory)

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

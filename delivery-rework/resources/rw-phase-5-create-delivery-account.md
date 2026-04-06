═══════════════════════════════════════
CURRENT PHASE: Phase 5 — Delivery Account Setup
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to collect account basics, build state-targeting criteria, ask about additional criteria, then create the delivery account.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Create Delivery Account**
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

Step 4: Collect Target States
  Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Normalize the user's response to uppercase USPS codes (e.g., California → CA). Store the normalized USPS abbreviations in targetStates — use abbreviated form in all downstream references and summarization.

Step 5: Load Lead Fields and Detect State Field
  Call the get_lead_type(leadTypeUID) tool and retain: leadTypeName, leadFields.
  If the tool fails, prompt: "I ran into an issue loading the lead type fields.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

  Detect the state field from leadFields:
  - Priority 1: leadFieldSpecialBit in {'State', 'StandardState'}
  - Priority 2: leadFieldName = "state" (case-insensitive)
  - Priority 3: leadFieldName contains "state" (substring)
  Do not process lower tiers once matched. If confidence is below 5%, confirm with the user which field represents the US state. **STOP AND YIELD.** If confirming.
  Retain stateFieldUID.

Step 6: Match States and Build Initial Criteria
  GUARD: If targetStates is empty, undefined, or was never explicitly provided by the user in this conversation, do NOT build a state criterion. Do NOT use example values (CA, AZ, TX) as defaults. Set criteriaPayload = [] and proceed to Step 7.
  Initialize criteriaPayload = [] (empty array). This array accumulates criteria across Phase 5 and Phase 5c. Do NOT reset it at any point.
  Call the get_usa_states tool. Match normalized targetStates to the returned list (match abbreviation, exact, case-insensitive). Collect ALL matched stateUID values into stateUIDArray.
  If no states matched, prompt: "None of those matched valid US states. Please re-enter your target states (e.g., CA, AZ, TX)." **STOP AND YIELD.** Re-normalize and re-match.
  Serialize stateUIDArray as pipe-delimited string: stateUIDArray.join('|').
  Build state criterion: {leadFieldUID: stateFieldUID, type: "FieldValue", operator: "In", value: "<pipe-delimited stateUID string>"}
  Set criteriaPayload = [state criterion].

Step 7: Criteria Gate — This step REQUIRES user input. Do not skip it.
  Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  Present using display_adaptive_card with an ActionSet: "Add criteria" | "Skip". MUST use display_adaptive_card.
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

  IF the user selected "Skip" OR said "none", "skip", or "no":
    Retain additionalCriteria = "None".
    Proceed to Step 8.

  IF the user selected "Add criteria" OR said "yes", "add", or "criteria":
    Call get_resource for mcp://resource/rw-phase-5c-criteria-builder. Do not generate any response until the resource has loaded.
    (No summarize — leadFields, criteriaPayload, and all account creation parameters stay in working memory.)
    IF the resource fails to load: prompt "I ran into an issue loading the criteria builder. Would you like to try again, or skip criteria?" Present ActionSet: "Try again" | "Skip". **STOP AND YIELD.** If Skip, retain additionalCriteria = "None", proceed to Step 8.
    (Phase 5c will handle account creation and summarization — do NOT execute Steps 8-9.)

Step 8: Create Delivery Account — MANDATORY TOOL CALL
  This step runs ONLY when the user skipped criteria in Step 7 (or Phase 5c failed to load). IF deliveryAccountUID already exists (set by Phase 5c), skip this step.
  You MUST call create_delivery_account. deliveryAccountUID does not exist yet — it is returned by this call. Do NOT use deliveryMethodUID as deliveryAccountUID.
  Call the create_delivery_account tool with these defaults:
  `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
  CRITICAL: createDeliveryAccountDto must be passed as a native object, NOT a JSON string. criteria must be an array of criterion objects, NOT a JSON string.
  If the tool fails, repair the payload and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

Step 9: Validate, Retain, and Summarize
  Verify deliveryAccountUID is a positive integer > 0. If 0 or null, treat as failure and retry Step 8.
  Retain: deliveryAccountUID, price, targetStates, isExclusive, useOrder, additionalCriteria.
  Immediately call the summarize_history tool.

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
* Target States: {targetStates} (USPS abbreviations)
* Additional Criteria: {additionalCriteria}
* Is Exclusive: {isExclusive}
* Use Order: {useOrder}

# Next Instructions
→ Load and execute Phase 6 at mcp://resource/rw-phase-6-delivery-account-summary
```

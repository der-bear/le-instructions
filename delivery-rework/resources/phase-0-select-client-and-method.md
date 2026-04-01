# Phase 0b: Select Client and Delivery Method

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 0b resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to collect a valid client and delivery-method selection, then hand off directly to Phase 5 (bypassing Phases 1-3).

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Missing Client Selection (Do this first)**
* IF clientUID is already known from context, skip to State 2.
* IF clientUID is missing:
  1. Call the get_clients tool to retrieve the clientsList.
  2. If clientsList is empty, prompt: "I couldn't find any clients.\n\nPlease create a client before continuing." **STOP AND YIELD.**
  3. Prompt the user exactly as follows: "Which client would you like to create a delivery account for?"
  4. Present the client list using the display_adaptive_card tool with an Input.ChoiceSet (style=compact) showing companyName as the display value and clientUID as the value.
  5. **STOP AND YIELD.** Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to select a client.
  - If the user types a value instead of clicking the card, match it against clientsList. If the result is ambiguous or low confidence, re-display the selector with the prompt: "I couldn't match that selection.\n\nWhich client would you like to create a delivery account for?" and STOP AND YIELD again.

**State 2: Missing Delivery Method Selection**
* IF clientUID is known AND deliveryMethodUID is missing:
  1. Call the get_client(clientUID) tool to retrieve: companyName, email, clientStatus, timeZoneName, timeOffset.
  2. If the tool fails, prompt: "I ran into an issue loading that client.\n\nPlease try again." **STOP AND YIELD.**
  3. Call the get_delivery_methods(clientUID) tool to retrieve the deliveryMethodsList. From the response, extract: deliveryMethodUID, deliveryMethodName (from data.name), and leadTypeUID (from data.leadTypeUID).
  4. If deliveryMethodsList is empty, prompt: "I couldn't find any delivery methods for this client.\n\nPlease create a delivery method before adding a delivery account." **STOP AND YIELD.**
  5. Prompt the user exactly as follows: "Which delivery method would you like to use?"
  6. Present the delivery methods using the display_adaptive_card tool with an Input.ChoiceSet (style=compact) showing deliveryMethodName as the display value and deliveryMethodUID as the value.
  7. **STOP AND YIELD.** Do not proceed to State 3. Do not call the summarize_history tool. You must wait for the user to select a delivery method.
  - If the user types a value instead of clicking the card, match it against deliveryMethodsList. If the result is ambiguous or low confidence, re-display the selector with the prompt: "I couldn't match that selection.\n\nWhich delivery method would you like to use?" and STOP AND YIELD again.

**State 3: Ready for Summarization**
* IF clientUID, companyName, email, clientStatus, timeZoneName, timeOffset, deliveryMethodUID, deliveryMethodName, and leadTypeUID are all known:
  1. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
## State
flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, leadTypeUID={leadTypeUID}

## Next
mcp://resource/phase-5-create-delivery-account
```

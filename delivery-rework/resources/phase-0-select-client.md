# Phase 0a: Select Client

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 0a resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to collect a valid client selection, load the client profile, and hand off to Phase 2.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Missing Client Selection (Do this first)**
* IF clientUID is already known from context, skip to State 2.
* IF clientUID is missing:
  1. Call the get_clients tool to retrieve the clientsList.
  2. If clientsList is empty, prompt: "I couldn't find any clients.\n\nPlease create a client before continuing." **STOP AND YIELD.** Do not hallucinate data.
  3. Prompt the user exactly as follows: "Which client would you like to create a delivery method for?"
  4. Present the client list using the display_adaptive_card tool with an Input.ChoiceSet (style=compact) showing companyName as the display value and clientUID as the value.
  5. **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to select a client.
  - If the user types a value instead of clicking the card, match it against clientsList. If the result is ambiguous or low confidence, re-display the selector with the prompt: "I couldn't match that selection.\n\nWhich client would you like to create a delivery method for?" and **STOP AND YIELD.** Do not hallucinate data.

**State 2: Load Client Profile and Summarize**
* IF clientUID is known:
  1. Call the get_client(clientUID) tool to retrieve: companyName, email, clientStatus, timeZoneName, timeOffset.
  2. If the tool fails, prompt: "I ran into an issue loading that client.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
# Current System State
* Flow Intent: {flowIntent}
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: {clientStatus}
* Time Zone Name: {timeZoneName}
* Time Offset: {timeOffset}

# Conversation Anchor
* DELIVERY_SETUP_START

# Next Instructions
Fetch and execute instructions from: mcp://resource/phase-2-get-lead-types
```

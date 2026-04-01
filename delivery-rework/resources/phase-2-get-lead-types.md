# Phase 2: Lead Type Selection

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 2 resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to have the user select a Lead Type for their client.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Missing Selection (Do this first)**
* IF the user has NOT yet selected a leadTypeUID:
  1. Call the get_lead_types tool (this is a data tool, NOT a resource-fetching tool) to retrieve the available leadTypesList.
  2. Prompt the user exactly as follows: "Please select a Lead Type for this client."
  3. Present the options to the user using the display_adaptive_card tool.
     - If there are 4 or fewer options, use an ActionSet.
     - If there are more than 4 options, use an Input.ChoiceSet (with style=compact).
  4. **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to click or reply with their selection.
  - If the user types a value instead of clicking the card, match it against leadTypesList. If the result is ambiguous or low confidence, re-display the selector with the prompt: "I couldn't match that selection.\n\nPlease select a Lead Type for this client." and **STOP AND YIELD.** Do not hallucinate data.

**State 2: Ready for Summarization**
* IF the user has successfully selected a leadTypeUID and leadTypeName:
  1. Immediately call the summarize_history tool.

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
* Lead Type UID: {leadTypeUID}
* Lead Type Name: {leadTypeName}

# Conversation Anchor
* DELIVERY_SETUP_START

# Next Instructions
Fetch and execute instructions from: mcp://resource/phase-3-create-delivery-method
```

═══════════════════════════════════════
CURRENT PHASE: Phase 2 — Lead Type Selection
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to have the user select a Lead Type for their client.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Missing Selection (Do this first)**
* IF the user has NOT yet selected a leadTypeUID:
  1. Call the get_lead_types tool (this is a data tool, NOT a resource-fetching tool) to retrieve the available leadTypesList.
     If leadTypesList is empty, prompt: "No lead types are available. Please contact support." **STOP AND YIELD.** Do not hallucinate data.
  2. You MUST call display_adaptive_card for the lead type selector. Do NOT render lead types as a plain text list. Present the lead type list using display_adaptive_card:
     - TextBlock: "Please select a Lead Type for this client."
     - If 4 or fewer options: ActionSet with leadTypeName as button titles.
     - If more than 4 options: Input.ChoiceSet (style=compact, placeholder="Select a Lead Type", choices from leadTypesList with title=leadTypeName, value=leadTypeUID) + Action.Submit.
  3. **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to respond.
  - If the user types a value instead of selecting from the card, match it against leadTypesList. When matching typed text, normalize spaces and compare case-insensitively before fuzzy matching. If the result is ambiguous or low confidence, re-display the selector with the prompt: "I couldn't match that selection.\n\nPlease select a Lead Type for this client." and **STOP AND YIELD.** Do not hallucinate data.

**State 2: Ready for Summarization**
* IF the user has successfully selected a leadTypeUID and leadTypeName:
  1. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 2 Complete — Lead Type Selected

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

# Next Instructions
→ Load and execute Phase 3 at mcp://resource/rw-phase-3-create-delivery-method
```


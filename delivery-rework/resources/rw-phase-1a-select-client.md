═══════════════════════════════════════
CURRENT PHASE: Phase 1a — Select Client
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to collect a valid client selection, load the client profile, and hand off to Phase 2.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Missing Client Selection (Do this first)**
* IF clientUID is already known from context, skip to State 2.
* IF clientUID is missing:
  1. Call the get_clients tool to retrieve the clientsList.
  2. If clientsList is empty, prompt: "I couldn't find any clients.\n\nPlease create a client before continuing." **STOP AND YIELD.** Do not hallucinate data.
  3. Present the client list using display_adaptive_card:
     - TextBlock: "Which client would you like to create a delivery method for?"
     - Input.ChoiceSet (style=compact, placeholder="Select a client", choices from clientsList with title=companyName, value=clientUID) + Action.Submit.
  4. **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to select a client.
  - If the user types a value instead of selecting from the card, match it against clientsList. If the result is ambiguous or low confidence, re-display the selector with the prompt: "I couldn't match that selection.\n\nWhich client would you like to create a delivery method for?" and **STOP AND YIELD.** Do not hallucinate data.

**State 2: Load Client Profile and Summarize**
* IF clientUID is known:
  1. Call the get_client(clientUID) tool to retrieve: companyName, email, clientStatus, timeZoneName, timeOffset.
  2. If the tool fails, prompt: "I ran into an issue loading that client.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 1a Complete — Client Selected

# Current System State
* Flow Intent: {flowIntent}
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: {clientStatus}
* Time Zone Name: {timeZoneName}
* Time Offset: {timeOffset}

# Next Instructions
→ Load and execute Phase 2 at mcp://resource/rw-phase-2-get-lead-types
```


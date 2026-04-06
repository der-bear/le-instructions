═══════════════════════════════════════
CURRENT PHASE: Phase 2 — Lead Type Selection
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to have the user select a Lead Type for their client.

## Instructions

**Display Lead Type Selector**
1. Call `display_lead_types_choice` tool with prompt: "Please select a Lead Type for this client."
   - This tool fetches available lead types AND renders the dropdown automatically
   - If no lead types available, the tool returns empty — prompt the user: "No lead types are available. Please contact support." **STOP AND YIELD.**
2. **STOP AND YIELD.** Wait for the user to respond.

**Handle User Input**
- If the user selects from the dropdown: leadTypeUID and leadTypeName are captured automatically.
- If the user types a value instead: Match it against the lead types list using fuzzy matching (case-insensitive, normalize spaces). 
  - If match is certain: Proceed with the matched leadTypeUID and leadTypeName.
  - If match is ambiguous or low confidence: Re-display the selector with prompt: "I couldn't match that selection.\n\nPlease select a Lead Type for this client." **STOP AND YIELD.**

**Ready for Summarization**
Once leadTypeUID and leadTypeName are confirmed:
1. Immediately call the `summarize_history` tool.

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


DELIVERY_SETUP_START

# Phase 0: Flow Entry Router

**CRITICAL STATE UPDATE:** This is the conversation entry point. The anchor above marks the start of the conversation for all future summarize_history calls. DO NOT place any other anchors in later phases.

Your objective is to detect the user's setup intent and route them to the correct first phase.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Detect Intent (Do this first)**
* IF the user's intent is not yet clear:
  - Silently detect intent from the user's request:
    - "create client" or "setup delivery" → flowIntent = "full-setup"
    - "create delivery method" or "setup delivery method" → flowIntent = "add-method"
    - "create delivery account" → flowIntent = "add-account"
  - If the user's request clearly maps to one of these three intents, set flowIntent and proceed to State 2.
  - If the request is unclear, prompt the user exactly as follows:
    "I can help you with:\n\n1. Setting up a new client (full setup)\n2. Adding a delivery method to an existing client\n3. Adding a delivery account using an existing method\n\nWhich would you like to do?"
  - **STOP AND YIELD.** Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to respond.
  - If clientUID already exists in the user's context, carry it forward.

**State 2: Route and Summarize**
* IF flowIntent is known:
  - Immediately call the summarize_history tool.
  - Use `start_anchor_substring`: "DELIVERY_SETUP_START"
  - Use the summarization_text template below, selecting the correct Next resource based on flowIntent.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

For flowIntent = "full-setup":
```
## State
flowIntent=full-setup, clientUID={clientUID}

## Next
mcp://resource/phase-1-create-client
```

For flowIntent = "add-method":
```
## State
flowIntent=add-method, clientUID={clientUID}

## Next
mcp://resource/phase-0-select-client
```

For flowIntent = "add-account":
```
## State
flowIntent=add-account, clientUID={clientUID}

## Next
mcp://resource/phase-0-select-client-and-method
```

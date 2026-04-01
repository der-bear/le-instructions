# Phase 1: Create Client

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 1 resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to collect the company name and contact email, create the client, and hand off to Phase 2.

Normalize the email before calling the tool: strip whitespace, lowercase the domain portion, validate RFC-5322 format.

The password is generated during client creation but is NOT retained. Phase 8 (activation) will generate a new password later.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Missing Client Inputs (Do this first)**
* IF companyName is missing OR email is missing:
  - IF both fields are missing, prompt the user exactly as follows:
    "Great - first we'll set up your Client, Delivery Method, and Delivery Account.\n\nTo create a new client, please provide:\n\n1. Company Name\n2. Contact Email"
  - ELSE IF only companyName is missing, prompt: "Please provide the Company Name."
  - ELSE IF only email is missing, prompt: "Please provide the Contact Email."
  - **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to respond.

**State 2: Create Client and Summarize**
* IF companyName and email are both known AND clientUID is missing:
  1. Call the create_client tool with these defaults:
     `clientUID={clientUID}, createClientDto={companyName={companyName}, email={email}, clientStatus="New", clientAutomationType="Price", username={email}, password=<generate a random 14-character password using upper, lower, digit, and symbol characters>, timeZoneName="Pacific Standard Time", timeOffset=-8}`
  2. If the tool fails because of a payload-shape or repairable request issue, repair the payload and retry once silently.
  3. If the tool still fails, prompt: "I ran into an issue creating the client.\n\nPlease confirm the company name and contact email and I'll try again."
     **STOP AND YIELD.** Do not hallucinate data. Do not call the summarize_history tool.
  4. If the tool succeeds, retain: clientUID, clientStatus="New", timeZoneName="Pacific Standard Time", timeOffset=-8, flowIntent="full-setup".
  5. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
# Current System State
* Flow Intent: full-setup
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: New
* Time Zone Name: Pacific Standard Time
* Time Offset: -8

# Conversation Anchor
* DELIVERY_SETUP_START

# Next Instructions
Fetch and execute instructions from: mcp://resource/rw-phase-2-get-lead-types
```

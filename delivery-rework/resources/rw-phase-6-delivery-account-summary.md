═══════════════════════════════════════
CURRENT PHASE: Phase 6 — Delivery Account Summary
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to display the delivery-account summary card and route based on the user's flow intent.

Display boolean values with context-appropriate labels:
- isExclusive: true → "Exclusive", false → "Shared"
- useOrder: true → "Yes", false → "No"

## Instructions

Display the delivery account summary using display_adaptive_card with this template:

```json
{"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Delivery Account Created", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Price per Lead"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{price}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Exclusivity"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{isExclusiveDisplay}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Target States"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{targetStates}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Additional Criteria"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{additionalCriteria}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Order System"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{useOrderDisplay}"}]}]}]}], "actions": [{"type": "Action.Submit", "title": "{buttonTitle}", "data": {"action": "{buttonAction}"}}]}
```

Replace all {variable} placeholders with current retained values. Set isExclusiveDisplay and useOrderDisplay per the boolean display rules above.
IF flowIntent = "full-setup": set buttonTitle="Continue" and buttonAction="Continue".
ELSE: set buttonTitle="Done" and buttonAction="Done".

**STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

IF flowIntent = "full-setup" AND the user said "Continue" or "done":
  Immediately call the summarize_history tool, then load Phase 7.

IF flowIntent is NOT "full-setup" AND the user said "Done" or "continue":
  Prompt the user exactly as follows: "✓ Your delivery account is ready to use for {companyName}."
  End the conversation for this branch. Do not suggest any next steps.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 6 Complete — Account Summary Reviewed

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
→ Load and execute Phase 7 at mcp://resource/rw-phase-7-client-summary
```

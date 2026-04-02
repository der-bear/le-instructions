═══════════════════════════════════════
CURRENT PHASE: Phase 6 — Delivery Account Summary
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to display the delivery-account summary card and route based on the user's flow intent. Do NOT call summarize_history in this phase — it adds no new state.

Display boolean values with context-appropriate labels:
- isExclusive: true → "Exclusive", false → "Shared"
- useOrder: true → "Yes", false → "No"

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Show Summary Card (Do this first)**
* IF accountSummaryChoice is missing:
  1. Display the following adaptive card using the display_adaptive_card tool:

```json
{"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Delivery Account Created", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Price per Lead"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{price}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Exclusivity"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{isExclusiveDisplay}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Target States"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{targetStates}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Additional Criteria"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{additionalCriteria}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Order System"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{useOrderDisplay}"}]}]}]}], "actions": [{"type": "Action.Submit", "title": "{buttonTitle}", "data": {"action": "{buttonAction}"}}]}
```

  2. Replace all {variable} placeholders with current retained values. Set isExclusiveDisplay and useOrderDisplay per the boolean display rules above.
  3. IF flowIntent = "full-setup": set buttonTitle="Continue" and buttonAction="Continue".
  4. ELSE: set buttonTitle="Done" and buttonAction="Done".
  5. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

**State 2: Route or Complete**
* IF flowIntent = "full-setup" AND the user said "Continue" or "done":
  - Load mcp://resource/rw-phase-7-client-summary

* IF flowIntent is NOT "full-setup" AND the user said "Done" or "continue":
  1. Prompt the user exactly as follows: "✓ Your delivery account is ready to use for {companyName}."
  2. End the conversation for this branch. Do not suggest any next steps.

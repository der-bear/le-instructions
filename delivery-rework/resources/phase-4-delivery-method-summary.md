# Phase 4: Delivery Method Summary

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 4 resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to display the delivery-method summary card and route based on the user's flow intent. Do NOT call summarize_history in this phase — it adds no new state.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Show Summary Card (Do this first)**
* IF the user has not yet acknowledged the summary card:
  1. Display the following adaptive card using the display_adaptive_card tool:

```json
{"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Delivery Method Created", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryType}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Hours"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryScheduleDisplay}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Field Mappings"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{mappedCount} of {totalCount} fields mapped"}]}]}]}], "actions": [{"type": "Action.Submit", "title": "{buttonTitle}", "data": {"action": "{buttonAction}"}}]}
```

  2. Replace all {variable} placeholders with current retained values.
  3. IF flowIntent = "full-setup": set buttonTitle="Continue" and buttonAction="Continue".
  4. ELSE: set buttonTitle="Done" and buttonAction="Done".
  5. **STOP AND YIELD.** You must wait for the user to click the button.

**State 2: Route or Complete**
* IF flowIntent = "full-setup" AND the user clicked "Continue":
  - Load mcp://resource/phase-5-create-delivery-account

* IF flowIntent is NOT "full-setup" AND the user clicked "Done":
  1. Prompt the user exactly as follows: "✓ Your delivery method is ready to use for {companyName}."
  2. End the conversation for this branch. Do not suggest any next steps.

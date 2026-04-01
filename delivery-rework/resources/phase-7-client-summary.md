# Phase 7: Client Summary

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 7 resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to display the final setup summary card and collect the user's activation choice. Do NOT call summarize_history in this phase — it adds no new state.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Show Summary Card (Do this first)**
* IF the user has not yet made an activation choice:
  1. Display the following adaptive card using the display_adaptive_card tool:

```json
{"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Client Setup Summary", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Contact Email"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{email}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Client Status"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{clientStatus}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName} (ID: {deliveryMethodUID})"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Account"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}-Account (ID: {deliveryAccountUID})"}]}]}]}, {"type": "TextBlock", "text": "Review your configuration above. Would you like to activate the client now?"}], "actions": [{"type": "Action.Submit", "title": "Activate", "data": {"action": "Activate"}}, {"type": "Action.Submit", "title": "Keep Inactive", "data": {"action": "Keep Inactive"}}]}
```

  2. Replace all {variable} placeholders with current retained values.
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to click Activate or Keep Inactive.

**State 2: Route to Activation**
* IF the user clicked "Activate" OR "Keep Inactive":
  - Retain clientSummaryChoice with the user's selection.
  - Load mcp://resource/phase-8-activation

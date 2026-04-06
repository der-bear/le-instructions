═══════════════════════════════════════
CURRENT PHASE: Phase 7 — Client Summary
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to display the final setup summary card and collect the user's activation choice. Do NOT call summarize_history in this phase — it adds no new state.

## Instructions


Display the client setup summary using display_adaptive_card tool and this template as base:

```json
{"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Client Setup Summary", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Contact Email"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{email}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Client Status"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{clientStatus}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Account"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}-Account"}]}]}]}, {"type": "TextBlock", "text": "Review your configuration above. Would you like to activate the client now?"}], "actions": [{"type": "Action.Submit", "title": "Activate", "data": {"action": "Activate"}}, {"type": "Action.Submit", "title": "Keep Inactive", "data": {"action": "Keep Inactive"}}]}
```

Replace all {variable} placeholders with current retained values.

**STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

IF the user said "Activate" or "Keep Inactive":
  Retain clientSummaryChoice with the user's selection.
  Load mcp://resource/rw-phase-8-activation

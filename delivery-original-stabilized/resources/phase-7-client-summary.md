═══════════════════════════════════════
<current_phase>Phase 7 — Client Summary</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 DISPLAY [adaptive_card] using this template:
 {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Client Setup Summary", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Contact Email"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{email}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Client Status"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{clientStatus}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName} (ID: {deliveryMethodUID})"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Account"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}-Account (ID: {deliveryAccountUID})"}]}]}]}, {"type": "TextBlock", "text": "Review your configuration above. Would you like to activate the client now?"}], "actions": [{"type": "Action.Submit", "title": "Activate", "data": {"action": "Activate"}}, {"type": "Action.Submit", "title": "Keep Inactive", "data": {"action": "Keep Inactive"}}]}

 Use the template above, replacing {variable} placeholders with actual retained values.
 WAIT for user choice
 NEXT_PHASE: mcp://resource/phase-8-activation - mandatory

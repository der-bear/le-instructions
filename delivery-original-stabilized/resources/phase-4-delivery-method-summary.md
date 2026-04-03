═══════════════════════════════════════
<current_phase>Phase 4 — Delivery Method Summary</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 DISPLAY [adaptive_card] using this template as base:
 {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Delivery Method Created", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryType}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Hours"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryScheduleDisplay}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Field Mappings"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{mappedCount} of {totalCount} fields mapped"}]}]}]}], "actions": [{"type": "Action.Submit", "title": "{methodSummaryButtonTitle}", "data": {"action": "{methodSummaryButtonAction}"}}]}

 Replace {variable} placeholders with actual retained values.
 IF flowIntent = "full-setup": methodSummaryButtonTitle = "Continue", methodSummaryButtonAction = "Continue"
 ELSE: methodSummaryButtonTitle = "Done", methodSummaryButtonAction = "Done"

 IF flowIntent = "full-setup":
   WAIT for user to click Continue
   NEXT_PHASE: mcp://resource/phase-5-create-delivery-account - mandatory
 ELSE:
   WAIT for user to click Done
   PROMPT: "✓ Your delivery method is ready to use for {companyName}."

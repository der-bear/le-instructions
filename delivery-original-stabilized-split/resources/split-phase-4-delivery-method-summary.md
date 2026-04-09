═══════════════════════════════════════
<current_phase>Phase 4 — Delivery Method Summary</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 DISPLAY [adaptive_card] using the template below. Replace {variable} placeholders with actual retained values.
 {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Delivery Method Created", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryTypeDisplay}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Hours"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryScheduleDisplay}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Field Mappings"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{mappedCount} of {totalCount} fields mapped"}]}]}]}], "actions": [{"type": "Action.Submit", "title": "{methodSummaryButtonTitle}", "data": {"action": "{methodSummaryButtonAction}"}}]}

 IF flowIntent = "full-setup": methodSummaryButtonTitle = "Continue", methodSummaryButtonAction = "Continue"
 ELSE: methodSummaryButtonTitle = "Done", methodSummaryButtonAction = "Done"

 IF flowIntent = "full-setup":
   WAIT for user to click Continue
   TOOL: summarize_history - mandatory
   TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 4 — Delivery Method Summary Reviewed</completed><current_state>deliveryMethodUID={deliveryMethodUID}</current_state><next_instructions>Load and execute Phase 5 from mcp://resource/split-phase-5-create-delivery-account</next_instructions></summary>"
 ELSE:
   WAIT for user to click Done
   PROMPT: "✓ Your delivery method is ready to use for {companyName}."

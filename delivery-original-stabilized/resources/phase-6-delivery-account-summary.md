═══════════════════════════════════════
<current_phase>Phase 6 — Delivery Account Summary</current_phase>
All prior phase summaries are completed history.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 Before inserting values, format booleans: isExclusive → "Exclusive"/"Shared", useOrder → "Enabled"/"Disabled"

 DISPLAY [adaptive_card] using this template as base:
 {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Delivery Account Created", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": false, "showGridLines": true, "columns": [{"width": 1}, {"width": 2}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Company Name"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{companyName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Type"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadTypeName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Method"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{deliveryMethodName}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Price per Lead"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{price}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Lead Exclusivity"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{isExclusive}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Target States"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{targetStates}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Additional Criteria"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{additionalCriteria}"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "Order System"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{useOrder}"}]}]}]}], "actions": [{"type": "Action.Submit", "title": "{accountSummaryButtonTitle}", "data": {"action": "{accountSummaryButtonAction}"}}]}

 Replace {variable} placeholders with actual retained values.
 IF flowIntent = "full-setup": accountSummaryButtonTitle = "Continue", accountSummaryButtonAction = "Continue" ELSE: accountSummaryButtonTitle = "Done", accountSummaryButtonAction = "Done"

 IF flowIntent = "full-setup":
   WAIT for user to click Continue
   TOOL: summarize_history - mandatory
   TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 6 — Delivery Account Summary Reviewed</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryAccountUID={deliveryAccountUID}, price={price}, targetStates={targetStates}, additionalCriteria={additionalCriteria}, isExclusive={isExclusive}, useOrder={useOrder}</current_state><next_instructions>Load and execute Phase 7 from mcp://resource/phase-7-client-summary</next_instructions></summary>"
 ELSE:
   WAIT for user to click Done
   PROMPT: "✓ Your delivery account is ready to use for {companyName}."

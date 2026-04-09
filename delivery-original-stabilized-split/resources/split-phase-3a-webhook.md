═══════════════════════════════════════
<current_phase>Phase 3a — Create Webhook Delivery Method</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
═══════════════════════════════════════

 PROMPT: "What's your webhook URL where we should send the leads?"
 ASK [conversational]: deliveryAddress
 WAIT for user input
 RETAIN: deliveryAddress (prepend https:// if the user omitted a scheme)

 PROMPT: "Would you like to configure field mappings?\n\nIf you have posting instructions or API documentation, I can automatically extract the field mappings."
 SUGGEST [adaptive_card]: ActionSet (I'll provide instructions | Skip for now)
 WAIT for user choice

 IF "Skip for now":
   skipFieldMapping = true

 IF "I'll provide instructions":
   TOOL: get_lead_type(leadTypeUID) → data.leadFields as leadFields
   RETAIN: leadFields

   PROMPT: "What content type should this delivery use?"
   ASK [adaptive_card]: ActionSet (URL Encoded | JSON | XML | I'm not sure)
   WAIT for user choice

   IF contentType = "I'm not sure":
     PROMPT: "Please paste your posting instructions or API schema, and I'll detect the format automatically."
   ELSE IF contentType = "URL Encoded":
     PROMPT: "Please provide field names (comma or line-separated) or a request body example."
   ELSE:
     PROMPT: "Please paste the {contentType} schema that your client's API expects."

   ASK [conversational]: postingInstructions
   WAIT for user input

   PROCESS (Input Validation):
     - If user provided only a raw URL, explain we can't fetch links and ask for the actual instructions.
     - Calculate input characteristics:
         inputLength = length of postingInstructions
         estimatedTokens = inputLength / 3
         hasLargeList = check for >200 repetitive items (regex: (\d{5}[,\s]+){200,} for ZIPs or similar patterns)
     - IF inputLength > 5000 OR estimatedTokens > 1500 OR hasLargeList:
         PROMPT: "This looks too large or unformatted. Would you like to simplify or skip mapping for now?"
         SUGGEST [adaptive_card]: ActionSet (Simplify input | Skip mapping)
         WAIT for user choice

         IF "Simplify input":
           PROMPT: "Please paste a smaller, well-formatted excerpt (e.g., the request body and a short field list)."
           ASK [conversational]: postingInstructions
           WAIT for user input
           Loop back to PROCESS (Input Validation) for re-validation

         IF "Skip mapping":
           skipFieldMapping = true

 IF skipFieldMapping:
   TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, responseSearch="success", useRegEx=false, settings=null, requestBody=null, deliveryDays={deliveryDays}}
   CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
   RETAIN: deliveryMethodName="{companyName}-Webhook", deliveryType="HttpPost", deliveryTypeDisplay="Webhook", deliveryAddress, mappedCount=0, totalCount=0

 IF NOT skipFieldMapping:
     PROCESS (Auto-detect Content Type):
       IF contentType = "I'm not sure":
         - Analyze postingInstructions format:
             IF appears to be JSON object/array syntax: detectedFormat = "JSON"
             ELSE IF appears to be XML markup with tags: detectedFormat = "XML"
             ELSE (plain text fields, comma-separated, key=value, etc.): detectedFormat = "URL Encoded"
         - PROMPT: "I've detected this as {detectedFormat} format. Is this correct?"
         - SUGGEST [adaptive_card]: ActionSet (Continue with {detectedFormat} | Switch content type)
         - WAIT for user choice
         - Do NOT proceed to Schema Validation until the user explicitly selects one of these actions.
         - IF "Continue with {detectedFormat}": contentType = detectedFormat
         - IF "Switch content type": Loop back to ask for contentType

     PROCESS (Schema Validation - JSON/XML only):
       IF contentType = "JSON" OR contentType = "XML":
         - Attempt to parse postingInstructions as {contentType}
         - Auto-fix common issues first, then retry parsing:
             • If missing outer braces for a JSON object property (e.g., starts with "key": {), wrap in { }
             • Replace single quotes with double quotes where safe
             • Remove trailing commas
             • Fix common formatting issues
         - ONLY IF parse still fails after auto-fix:
             PROMPT: "I couldn't parse this as valid {contentType}. Would you like to:\n• Fix and re-paste the {contentType} schema\n• Switch to a different content type"
             SUGGEST [adaptive_card]: ActionSet (Re-paste {contentType} schema | Switch content type)
             WAIT for user choice
             IF "Re-paste {contentType} schema": Loop back to ask for postingInstructions
             IF "Switch content type": Loop back to ask for contentType
         - Store valid schema structure

     PROCESS (Field Mapping & RequestBody Generation):
       - Extract field names from postingInstructions
       - Fuzzy match to leadFields (of selected lead type) in the following priority: exact match → underscore/CamelCase variations → abbreviations → semantic mapping (>90% confidence only).
       - If multiple fields match at the same priority level, prompt the user to select the correct field from the candidates.
       - If no confident match is found, ask the user for clarification before proceeding.
       - Build mappingSettings: [{fieldType:"LeadField", fieldName:<delivery field>, leadFieldUID:<system field uid>}]
       - Compute mappedCount, totalCount

       - Generate requestBody following this rule: delivery_field=[SystemFieldName]
           IF contentType = "URL Encoded":
             Format: field1=[SystemField1]&field2=[SystemField2]&
           IF contentType = "JSON" OR contentType = "XML":
             1. Extract structure from user's schema (hierarchy, nesting, arrays)
             2. Auto-correct any syntax errors
             3. Generate requestBody template:
                - Include only mapped fields, preserving hierarchy from user's schema
                - Replace values with [SystemFieldName] placeholders
                - JSON: placeholders are quoted strings "[SystemFieldName]"
                - XML: placeholders are unquoted content <field>[SystemFieldName]</field>
                - Output format: 2-space indentation, one element per line

       - RETAIN: mappingSettings, requestBody, mappedCount, totalCount, deliveryAddress
       - CRITICAL: MUST execute the DISPLAY [adaptive_card] below IMMEDIATELY after processing (in same message, no progress updates, show even if 0 mapped)

       DISPLAY [adaptive_card] field mapping preview using this template:
       {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Field Mapping Preview", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": true, "showGridLines": true, "columns": [{"width": 1}, {"width": 1}, {"width": 1}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "System Field"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Field"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "Status"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadFieldName}"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{fieldName}"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "✓ Mapped"}]}]}]}, {"type": "TextBlock", "text": "Successfully mapped {mappedCount} out of {totalCount} fields."}], "actions": [{"type": "Action.Submit", "title": "Continue", "data": {"action": "Continue"}}]}

       FOR EACH item in mappingSettings:
         - Get leadFieldName from leadFields where leadFieldUID matches item.leadFieldUID
         - Duplicate the data row template (line with {leadFieldName} and {fieldName})
         - Replace {leadFieldName} with actual leadFieldName, {fieldName} with item.fieldName

       WAIT for user to click Continue

     PROCESS (Map contentType to MIME): mimeContentType = JSON→"application/json", XML→"application/xml", "URL Encoded"→"application/x-www-form-urlencoded"
     TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, contentType={mimeContentType}, responseSearch="success", useRegEx=false, settings={mappingSettings}, requestBody={requestBody}, deliveryDays={deliveryDays}}
     CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
     NOTE: settings=mappingSettings, requestBody is generated template with [SystemFieldName] placeholders, deliveryDays=null for 24/7 or array for specific hours
     RETAIN: deliveryMethodName="{companyName}-Webhook", deliveryType="HttpPost", deliveryTypeDisplay="Webhook", deliveryAddress, mimeContentType, requestBody, mappedCount, totalCount

 TOOL: create_delivery_method → data as deliveryMethodUID
 RETAIN: deliveryMethodUID, deliveryType, deliveryTypeDisplay

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 3 — Delivery Method Created</completed><current_state>deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryTypeDisplay={deliveryTypeDisplay}, deliveryAddress={deliveryAddress}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}</current_state><next_instructions>Load and execute Phase 3b from mcp://resource/split-phase-3b-webhook-test</next_instructions></summary>"

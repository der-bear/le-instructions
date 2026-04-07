═══════════════════════════════════════
<current_phase>Phase 3 — Create Delivery Method</current_phase>
All prior phase summaries are completed history.
CRITICAL: Any <next_instructions> tags in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
Execute ONLY the instructions below.
Follow steps in order from top to bottom. Do NOT skip ahead.
CRITICAL: After each user response (including card button clicks), resume from where you left off — do not regress to earlier steps or re-load this phase, unless an explicit retry/loop-back is specified in the instructions.
═══════════════════════════════════════

 PROMPT: "First, let's set the delivery schedule.\n\nWould you like leads delivered 24/7, or only during specific hours?"
 ASK [adaptive_card]: ActionSet (24/7 delivery | Specific hours only)
 WAIT for user choice — STOP here. Do NOT proceed to delivery method type until the schedule is fully resolved (hours collected if "Specific hours only", or deliveryDays built if "24/7").

 IF "Specific hours only":
   PROMPT: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)"
   ASK [conversational]: scheduleInput
   WAIT for user input
   PROCESS (Normalize schedule): Parse natural language to ISO format with timezone. Use current year's date (YYYY-01-01) + time + timezone offset (use retained timeOffset if available, otherwise default to -8 for PST). Example: "9am-5pm" with timeOffset=-8 → startTime: "YYYY-01-01T09:00:00-08:00", endTime: "YYYY-01-01T17:00:00-08:00"

   BUILD deliveryDays array with EXACTLY 7 entries (weekDay 0-6, one for each day of week):
     - User-specified days: allow=true with their startTime/endTime
     - All other days: allow=false with startTime="YYYY-01-01T00:00:00±HH:mm", endTime="YYYY-01-01T23:59:59±HH:mm"
   RETAIN: deliveryDays, deliveryScheduleDisplay

 IF "24/7 delivery":
   BUILD deliveryDays array with EXACTLY 7 entries (weekDay 0-6):
     All days: allow=true, startTime="YYYY-01-01T00:00:00+00:00", endTime="YYYY-01-01T23:59:59+00:00"
   RETAIN: deliveryDays, deliveryScheduleDisplay="24/7"

 PROMPT: "How would you like your leads delivered?\n\n• Webhook – sends lead data via HTTP POST\n• Portal – client accesses leads via web portal\n• FTP – uploads lead files to a server\n• Email – delivers leads to an inbox"
 ASK [adaptive_card]: ActionSet (Portal | Webhook | Email | FTP)
 WAIT for user choice — STOP here. Then enter the matching IF branch below and follow its steps sequentially.

 IF "Portal":
   TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Portal", enabled=true, leadTypeUID={leadTypeUID}, deliveryDays={deliveryDays}}
   CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
   RETAIN: deliveryMethodName="{companyName}-Portal", deliveryType="HttpPost", deliveryAddress="Portal", mappedCount=0, totalCount=0, connectionTestMode="none"

 IF "Webhook":
   PROMPT: "What's your webhook URL where we should send the leads?"
   ASK [conversational]: deliveryAddress
   WAIT for user input — STOP here. Do NOT show the field mapping question until the webhook URL is received.
   PROCESS (Normalize URL): prepend https:// if missing

   PROMPT: "Would you like to configure field mappings?\n\nIf you have posting instructions or API documentation, I can automatically extract the field mappings."
   SUGGEST [adaptive_card]: ActionSet (I'll provide instructions | Skip for now)
   WAIT for user choice

   IF "Skip for now":
     skipFieldMapping = true

   IF "I'll provide instructions":
     TOOL: get_lead_type(leadTypeUID) → data.leadFields as leadFields
     RETAIN: leadFields
     After retaining leadFields, immediately proceed to the content type question below.

     PROMPT: "What content type should this delivery use?"
     ASK [adaptive_card] using display_adaptive_card: ActionSet (URL Encoded | JSON | XML | I'm not sure)
     WAIT for user choice

     IF contentType = "I'm not sure":
       PROMPT: "Please paste your posting instructions or API schema, and I'll detect the format automatically."
     ELSE IF contentType = "URL Encoded":
       PROMPT: "Please provide field names (comma or line-separated) or a request body example."
     ELSE:
       PROMPT: "Please paste the {contentType} schema that your client's API expects."

     CRITICAL: You MUST display the PROMPT above and wait for user input before proceeding. Do NOT skip to field mapping or method creation.
     ASK [conversational]: postingInstructions
     WAIT for user input

     PROCESS (Input Validation):
       - Check if postingInstructions contains "http://" OR "https://"
       - IF URL detected:
           PROMPT: "I can't access external links. Please open the page and paste the posting instructions or schema text here."
           ASK [conversational]: postingInstructions
           WAIT for user input
           Loop back to PROCESS (Input Validation) for re-validation

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
     RETAIN: deliveryMethodName="{companyName}-Webhook", deliveryType, deliveryAddress, mappedCount=0, totalCount=0, connectionTestMode="webhook"

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
           - IF "Continue with {detectedFormat}": contentType = detectedFormat
           - IF "Switch content type": Loop back to ask for contentType

       PROCESS (Schema Validation - JSON/XML only):
         IF contentType = "JSON" OR contentType = "XML":
           - Interpret the postingInstructions as {contentType}. Use your best judgment to fix common issues silently: missing outer braces, trailing commas, unescaped quotes, single quotes, missing colons. The user is providing field structure, not production code — be lenient and extract the field names even if the format is imperfect.
           - ONLY if the input is truly unintelligible (no recognizable field structure at all):
               PROMPT: "I couldn't parse this as valid {contentType}. Would you like to:\n• Fix and re-paste the {contentType} schema\n• Switch to a different content type\n• Skip field mapping for now"
               SUGGEST [adaptive_card]: ActionSet (Re-paste {contentType} schema | Switch content type | Skip mapping)
               WAIT for user choice
               IF "Re-paste {contentType} schema": Loop back to ask for postingInstructions
               IF "Switch content type": Loop back to ask for contentType
               IF "Skip mapping": skipFieldMapping = true
           - Store valid schema structure

       PROCESS (Field Mapping & RequestBody Generation):
         - Extract field names from postingInstructions. Match field NAMES only — do NOT analyze field values, data types, or enumeration values.
         - Fuzzy match to leadFields (of selected lead type) in the following priority: exact match → underscore/CamelCase variations → abbreviations → semantic mapping (>85% confidence only).
         - Leave unmapped: any field with multiple candidates at the same priority level, and any field with no confident match.
         - Build mappingSettings: [{fieldType:"LeadField", fieldName:<delivery field>, leadFieldUID:<system field uid>}]
         - Compute mappedCount, totalCount

         - Generate requestBody following this rule: delivery_field=[SystemFieldName]
             IF contentType = "URL Encoded":
               Format: field1=[SystemField1]&field2=[SystemField2]&
             IF contentType = "JSON" OR contentType = "XML":
               1. Extract structure from user's schema (hierarchy, nesting, arrays)
               2. Auto-correct any syntax errors
               3. Generate requestBody template:
                  - Include ONLY fields present in mappingSettings. Do NOT include any field from the user's schema that was not mapped. Do NOT invent placeholders to fill structural slots.
                  - Preserve hierarchy from user's schema for the included fields only.
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

         WAIT for user to click Continue — Do NOT proceed to method creation until user confirms the mapping preview.

       PROCESS (Map contentType to MIME): mimeContentType = JSON→"application/json", XML→"application/xml", "URL Encoded"→"application/x-www-form-urlencoded"
       TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, contentType={mimeContentType}, responseSearch="success", useRegEx=false, settings={mappingSettings}, requestBody={requestBody}, deliveryDays={deliveryDays}}
       CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
       NOTE: settings=mappingSettings, requestBody is generated template with [SystemFieldName] placeholders, deliveryDays is always the 7-entry array built earlier
       RETAIN: deliveryMethodName="{companyName}-Webhook", deliveryType, deliveryAddress, mimeContentType, requestBody, mappedCount, totalCount, connectionTestMode="webhook"

 IF "Email":
   PROMPT: "Leads will be delivered to {email}. Would you like to use a different email address?"
   ASK [adaptive_card]: ActionSet (Use {email} | Different address)
   WAIT for user choice
   IF "Different address":
     PROMPT: "Please provide the delivery email address."
     ASK [conversational]: deliveryEmailAddress
     WAIT for user input
   ELSE:
     deliveryEmailAddress = {email}
   TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="EMail", name="{companyName}-Email", enabled=true, leadTypeUID={leadTypeUID}, emailAddress={deliveryEmailAddress}, toEmailAddress={deliveryEmailAddress}, emailSubject="New Lead - {date}", emailOrSmsTemplate="Standard template", deliveryDays={deliveryDays}}
   CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
   RETAIN: deliveryMethodName="{companyName}-Email", deliveryType="EMail", deliveryAddress={deliveryEmailAddress}, mappedCount=0, totalCount=0, connectionTestMode="none"

 IF "FTP":
   PROMPT: "Please provide FTP details:\n\n1. Server address\n2. Username\n3. Password"
   ASK [conversational]: deliveryAddress, ftpUser, ftpPassword
   WAIT for user input
   TOOL_DEFAULTS: clientUID={clientUID}, createDeliveryMethodDto={deliveryType="FTP", name="{companyName}-FTP", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, ftpPath="/incoming/", deliveryDays={deliveryDays}}
   CRITICAL: createDeliveryMethodDto must be passed as an object, NOT a JSON string
   RETAIN: deliveryMethodName="{companyName}-FTP", deliveryType="FTP", deliveryAddress, ftpUser, ftpPassword, mappedCount=0, totalCount=0, connectionTestMode="ftp"

 TOOL: create_delivery_method → data as deliveryMethodUID
 RETAIN: deliveryMethodUID, deliveryType

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="DELIVERY_SETUP_START", summarization_text="<summary><completed>Phase 3 — Delivery Method Created</completed><current_state>flowIntent={flowIntent}, clientUID={clientUID}, companyName={companyName}, email={email}, clientStatus={clientStatus}, timeZoneName={timeZoneName}, timeOffset={timeOffset}, leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}, deliveryMethodUID={deliveryMethodUID}, deliveryMethodName={deliveryMethodName}, deliveryType={deliveryType}, deliveryAddress={deliveryAddress}, ftpUser={ftpUser}, ftpPassword={ftpPassword}, mimeContentType={mimeContentType}, requestBody={requestBody}, deliveryScheduleDisplay={deliveryScheduleDisplay}, mappedCount={mappedCount}, totalCount={totalCount}, connectionTestMode={connectionTestMode}</current_state><next_instructions>Load and execute Phase 3b from mcp://resource/phase-3b-test-connection</next_instructions></summary>"

═══════════════════════════════════════
CURRENT PHASE: Phase 3 Webhook — Webhook Delivery Method
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

Your objective is to collect webhook configuration, optionally build field mappings, create the delivery method, and hand off to Phase 3b.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Collect URL and Mapping Choice**
Execute these steps in order:

Step 1: Collect Webhook URL
  Prompt the user exactly as follows: "What's your webhook URL where we should send the leads?"
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to provide the URL.
  Prepend `https://` to the URL if the user omitted a scheme.

Step 2: Ask About Field Mapping
  Prompt the user exactly as follows: "Would you like to configure field mappings?\n\nIf you have posting instructions or API documentation, I can automatically extract the field mappings."
  Present the choice using display_adaptive_card with an ActionSet: "I'll provide instructions" | "Skip for now".
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  - IF user selects "Skip for now": set skipFieldMapping=true, go to State 4 (create without mapping).
  - IF user selects "I'll provide instructions": set skipFieldMapping=false, proceed to State 2.

**State 2: Collect Content Type and Posting Instructions**
Execute these steps in order:

Step 1: Load Lead Fields (silent)
  Call the get_lead_type(leadTypeUID) tool and retain leadFields.
  If the tool fails, prompt: "I ran into an issue loading the lead fields for mapping.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

Step 2: Ask Content Type
  Prompt the user exactly as follows: "What content type should this delivery use?"
  Present the choice using display_adaptive_card with an ActionSet: "URL Encoded" | "JSON" | "XML" | "I'm not sure".
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

Step 3: Collect Posting Instructions
  IF contentTypeChoice = "I'm not sure": prompt exactly: "Please paste your posting instructions or API schema, and I'll detect the format automatically."
  ELSE IF contentTypeChoice = "URL Encoded": prompt exactly: "Please provide field names (comma or line-separated) or a request body example."
  ELSE: prompt exactly: "Please paste the {contentTypeChoice} schema that your client's API expects."
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to paste the content.

Step 4: Input Validation (inline — check before proceeding)
  IF postingInstructions contains "http://" or "https://": prompt exactly: "I can't access external links. Please open the page and paste the posting instructions or schema text here." **STOP AND YIELD.** On re-entry, re-validate.
  IF input length > 5000 chars OR estimated tokens (length/3) > 1500 OR contains >200 repetitive items (e.g., ZIP code lists): prompt exactly: "This looks too large or unformatted. Would you like to simplify or skip mapping for now?" Present using display_adaptive_card an ActionSet: "Simplify input" | "Skip mapping". **STOP AND YIELD.**
  - IF "Simplify input": prompt exactly: "Please paste a smaller, well-formatted excerpt (e.g., the request body and a short field list)." **STOP AND YIELD.** On re-entry, re-validate.
  - IF "Skip mapping": set skipFieldMapping=true, go to State 4 (create without mapping).

Step 5: Auto-Detect (only if "I'm not sure")
  IF contentTypeChoice = "I'm not sure":
    Detect format from postingInstructions: JSON object/array syntax → "JSON", XML markup with tags → "XML", otherwise → "URL Encoded".
    Prompt the user exactly as follows: "I've detected this as {detectedFormat} format. Is this correct?"
    Present the choice using display_adaptive_card with an ActionSet: "Continue with {detectedFormat}" | "Switch content type".
    **STOP AND YIELD.** Do not hallucinate data.
    - IF "Continue with {detectedFormat}": set contentTypeChoice = detectedFormat, proceed to State 3.
    - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to Step 2 of this state.
  ELSE: proceed directly to State 3.

**State 3: Parse, Map, and Preview**
Execute these steps in order:

Step 1: Parse Schema
  IF contentTypeChoice = "JSON" or "XML": attempt to parse postingInstructions. Auto-fix before failing (missing braces, trailing commas, unescaped quotes, single→double quotes).
  XML/JSON auto-fix MUST complete in a single attempt. If still invalid after one auto-fix pass, immediately show the Re-paste/Switch prompt. Do NOT attempt multiple rounds of auto-fixing.
  IF parsing still fails: prompt exactly: "I couldn't parse this as valid {contentTypeChoice}. Would you like to:\n• Fix and re-paste the {contentTypeChoice} schema\n• Switch to a different content type"
  Present using display_adaptive_card an ActionSet: "Re-paste {contentTypeChoice} schema" | "Switch content type". **STOP AND YIELD.**
  - IF "Re-paste": clear postingInstructions, prompt exactly: "Please paste the {contentTypeChoice} schema that your client's API expects." **STOP AND YIELD.**
  - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to State 2 Step 2.

Step 2: Match Fields (silent — do NOT display results)
  Extract field names from postingInstructions.
  Match extracted fields to leadFields by priority: exact match → underscore/CamelCase variations → abbreviations → semantic (>90% only). Auto-map confident matches silently — do NOT display mapping results, the preview comes in Step 5.

Step 3: Resolve Unclear Mappings
  If any ambiguous fields (multiple candidates at the same priority level) or unmatched fields (no confident match) remain after Step 2, you MUST ask the user for clarification before proceeding.
  For each ambiguous field, prompt the user to select the correct system field from the candidate matches.
  For each unmatched field, ask the user which system field it should map to.
  Do NOT auto-resolve ambiguous or unmatched fields.
  Do NOT present an adaptive card for each ambiguous field. Ask in plain text. You may batch multiple unresolved fields into one clarification message.
  **STOP AND YIELD.** Do not hallucinate data.
  On re-entry, apply the user's answers to the unresolved fields.
  If unresolved fields still remain, ask again and **STOP AND YIELD** until they are resolved.

Step 4: Build Mapping Payload
  Build mappingSettings using the confidently matched fields plus any user-resolved fields: [{fieldType:"LeadField", fieldName:<delivery field>, leadFieldUID:<system field uid>}, ...]
  Build requestBody with [SystemFieldName] placeholders:
  - URL Encoded: `field1=[SystemField1]&field2=[SystemField2]&`
  - JSON/XML: preserve the user's posted structure, replace mapped values with [SystemFieldName] placeholders. JSON placeholders are quoted strings "[SystemFieldName]". XML placeholders are unquoted content `<field>[SystemFieldName]</field>`. Use 2-space indentation, one element per line.
  Include only mapped fields in mappingSettings and in requestBody.
  Compute mappedCount (number of successfully mapped fields) and totalCount (total number of fields extracted from postingInstructions, including unmapped). Both MUST be shown in the preview.
  Map contentTypeChoice to mimeContentType: JSON → "application/json", XML → "application/xml", URL Encoded → "application/x-www-form-urlencoded".
  Retain: mappingSettings, requestBody, mappedCount, totalCount, mimeContentType, connectionTestMode="webhook".

Step 5: Display Mapping Preview
  CRITICAL: You MUST call display_adaptive_card with the EXACT JSON template below IMMEDIATELY — in the same message as field matching completes, no progress text, no intermediate messages. Do NOT render mappings as plain text or arrows. The preview MUST be a Table card. MUST show preview for ALL content types (JSON, XML, URL Encoded).
  Show only the mapped rows in the table preview.

  Display the field mapping preview using display_adaptive_card tool and this template as base:

```json
  {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Field Mapping Preview", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": true, "showGridLines": true, "columns": [{"width": 1}, {"width": 1}, {"width": 1}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "System Field"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Field"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "Status"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadFieldName}"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{fieldName}"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "✓ Mapped"}]}]}]}, {"type": "TextBlock", "text": "Successfully mapped {mappedCount} out of {totalCount} fields."}], "actions": [{"type": "Action.Submit", "title": "Continue", "data": {"action": "Continue"}}]}
```

  FOR EACH item in mappingSettings: look up the leadFieldName from leadFields where leadFieldUID matches the item's leadFieldUID. Duplicate the data row template, replacing {leadFieldName} with the actual leadFieldName and {fieldName} with item.fieldName. Set Status to "✓ Mapped".
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to say Continue.
  When the user says Continue, proceed to State 4.

**State 4: Create Method and Summarize**

Step 1: Create Delivery Method
  IF skipFieldMapping = true (no mapping path):
    Call the create_delivery_method tool with these defaults:
    `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, responseSearch="success", useRegEx=false, settings=null, requestBody=null, deliveryDays={deliveryDays}}`
    Retain: deliveryMethodUID, deliveryMethodName="{companyName}-Webhook", deliveryType="HttpPost", mappedCount=0, totalCount=0, connectionTestMode="webhook".

  IF skipFieldMapping = false (with mapping path):
    Call the create_delivery_method tool with these defaults:
    `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, contentType={mimeContentType}, responseSearch="success", useRegEx=false, settings={mappingSettings}, requestBody={requestBody}, deliveryDays={deliveryDays}}`
    Retain: deliveryMethodUID, deliveryMethodName="{companyName}-Webhook", deliveryType="HttpPost".

  CRITICAL: createDeliveryMethodDto must be passed as a native object, NOT a JSON string.
  If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

Step 2: Summarize
  Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```text
# Phase 3 Complete — Webhook Method Created

# Current System State
* Flow Intent: {flowIntent}
* Client UID: {clientUID}
* Company Name: {companyName}
* Email: {email}
* Client Status: {clientStatus}
* Time Zone Name: {timeZoneName}
* Time Offset: {timeOffset}
* Lead Type UID: {leadTypeUID}
* Lead Type Name: {leadTypeName}
* Delivery Method UID: {deliveryMethodUID}
* Delivery Method Name: {deliveryMethodName}
* Delivery Type: HttpPost
* Delivery Address: {deliveryAddress}
* MIME Content Type: {mimeContentType}
* Request Body: {requestBody}
* Delivery Schedule Display: {deliveryScheduleDisplay}
* Mapped Count: {mappedCount}
* Total Count: {totalCount}
* Connection Test Mode: webhook

# Next Instructions
→ Load and execute Phase 3b at mcp://resource/rw-phase-3b-webhook-test
```

# Phase 3: Webhook Delivery Method

**CRITICAL STATE UPDATE:** You have successfully fetched the Phase 3 webhook resource. DO NOT call the get_resource tool again for this phase. You must now read the instructions below and execute State 1.

Your objective is to collect webhook configuration, optionally build field mappings, create the delivery method, and hand off to Phase 3b.

## Instructions

Evaluate what information you currently have and take the appropriate action:

**State 1: Collect Webhook URL (Do this first)**
* IF deliveryAddress is missing:
  1. Prompt the user exactly as follows: "What's your webhook URL where we should send the leads?"
  2. **STOP AND YIELD.** Do not hallucinate data. Do not proceed. You must wait for the user to provide the URL.
  - Prepend `https://` to the URL if the user omitted a scheme.

**State 2: Field Mapping Choice**
* IF deliveryAddress is known AND skipFieldMapping is not yet decided:
  1. Prompt the user exactly as follows: "Would you like to configure field mappings?\n\nIf you have posting instructions or API documentation, I can automatically extract the field mappings."
  2. Present the choice using display_adaptive_card with an ActionSet: "I'll provide instructions" | "Skip for now".
  3. **STOP AND YIELD.** Do not hallucinate data. Do not proceed. You must wait for the user to respond.
  - IF user selects "Skip for now": set skipFieldMapping=true, proceed to State 3.
  - IF user selects "I'll provide instructions": set skipFieldMapping=false, proceed to State 4.

**State 3: Create Method Without Mapping**
* IF skipFieldMapping = true AND deliveryMethodUID is missing:
  1. Call the create_delivery_method tool with these defaults:
     `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, responseSearch="success", useRegEx=false, settings=null, requestBody=null, deliveryDays={deliveryDays}}`
  2. If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. If the tool succeeds, retain: deliveryMethodUID, deliveryMethodName="{companyName}-Webhook", deliveryType="HttpPost", mappedCount=0, totalCount=0, connectionTestMode="webhook".
  4. Proceed to State 8.

**State 4: Collect Content Type and Posting Instructions**
* IF skipFieldMapping = false AND leadFields is missing:
  - Call the get_lead_type(leadTypeUID) tool and retain leadFields.
  - If the tool fails, prompt: "I ran into an issue loading the lead fields for mapping.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.

* IF skipFieldMapping = false AND contentTypeChoice is missing:
  1. Prompt the user exactly as follows: "What content type should this delivery use?"
  2. Present the choice using display_adaptive_card with an ActionSet: "URL Encoded" | "JSON" | "XML" | "I'm not sure".
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF skipFieldMapping = false AND contentTypeChoice is known AND postingInstructions is missing:
  1. IF contentTypeChoice = "I'm not sure": prompt exactly: "Please paste your posting instructions or API schema, and I'll detect the format automatically."
  2. ELSE IF contentTypeChoice = "URL Encoded": prompt exactly: "Please provide field names (comma or line-separated) or a request body example."
  3. ELSE: prompt exactly: "Please paste the {contentTypeChoice} schema that your client's API expects."
  4. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to paste the content.

* IF postingInstructions is known AND mappingSettings is missing:
  - **Input validation (run inline, do not create separate states):**
    - IF postingInstructions contains "http://" or "https://": prompt exactly: "I can't access external links. Please open the page and paste the posting instructions or schema text here." **STOP AND YIELD.** Do not hallucinate data. On re-entry, re-validate.
    - IF input is too large (>5000 chars) or too unformatted: prompt exactly: "This looks too large or unformatted. Would you like to simplify or skip mapping for now?" Present an ActionSet: "Simplify input" | "Skip mapping". **STOP AND YIELD.** Do not hallucinate data.
      - IF "Simplify input": prompt exactly: "Please paste a smaller, well-formatted excerpt (e.g., the request body and a short field list)." **STOP AND YIELD.** Do not hallucinate data. On re-entry, re-validate.
      - IF "Skip mapping": set skipFieldMapping=true, proceed to State 3.
  - **After input passes validation**, proceed to State 5.

**State 5: Auto-Detect Confirmation (only if content type was "I'm not sure")**
* IF contentTypeChoice = "I'm not sure" AND postingInstructions is known AND mappingSettings is missing:
  1. Detect format from postingInstructions: JSON object/array syntax → "JSON", XML markup with tags → "XML", otherwise → "URL Encoded".
  2. Prompt the user exactly as follows: "I've detected this as {detectedFormat} format. Is this correct?"
  3. Present the choice using display_adaptive_card with an ActionSet: "Continue with {detectedFormat}" | "Switch content type".
  4. **STOP AND YIELD.** Do not hallucinate data.
  - IF "Continue with {detectedFormat}": set contentTypeChoice = detectedFormat, proceed to State 6.
  - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to the content-type prompt in State 4.

**State 6: Parse Schema and Build Mapping**
* IF contentTypeChoice is known (not "I'm not sure") AND postingInstructions is known AND mappingSettings is missing:
  1. IF contentTypeChoice = "JSON" or "XML": attempt to parse postingInstructions. Auto-fix before failing (missing braces, trailing commas, unescaped quotes, single→double quotes).
     - IF parsing still fails: prompt exactly: "I couldn't parse this as valid {contentTypeChoice}. Would you like to:\n• Fix and re-paste the {contentTypeChoice} schema\n• Switch to a different content type"
       Present an ActionSet: "Re-paste {contentTypeChoice} schema" | "Switch content type". **STOP AND YIELD.** Do not hallucinate data.
       - IF "Re-paste": clear postingInstructions, prompt exactly: "Please paste the {contentTypeChoice} schema that your client's API expects." **STOP AND YIELD.** Do not hallucinate data.
       - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to the content-type prompt in State 4.
  2. Extract field names from postingInstructions.
  3. Match fields to leadFields by priority: exact match → underscore/CamelCase variations → abbreviations → semantic (>90% only). If ambiguous, prompt the user to select. If no match, ask for clarification.
  4. If ambiguity or no match for a field, prompt the user for clarification. **STOP AND YIELD.** Do not hallucinate data.
  5. Build mappingSettings: [{fieldType:"LeadField", fieldName:<delivery field>, leadFieldUID:<system field uid>}, ...]
  6. Build requestBody with [SystemFieldName] placeholders:
     - URL Encoded: `field1=[SystemField1]&field2=[SystemField2]&...`
     - JSON/XML: preserve the user's posted structure, replace mapped values with [SystemFieldName] placeholders. JSON placeholders are quoted strings "[SystemFieldName]". XML placeholders are unquoted content `<field>[SystemFieldName]</field>`. Use 2-space indentation, one element per line.
  7. Compute mappedCount and totalCount.
  8. Map contentTypeChoice to mimeContentType: JSON → "application/json", XML → "application/xml", URL Encoded → "application/x-www-form-urlencoded".
  9. Retain: mappingSettings, requestBody, mappedCount, totalCount, mimeContentType, connectionTestMode="webhook".
  10. Proceed to State 7.

**State 7: Mapping Preview**
* IF mappingSettings is known AND deliveryMethodUID is missing AND the user has not yet clicked Continue on the preview:
  1. Display the field mapping preview using display_adaptive_card with this template:

```json
{"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Field Mapping Preview", "weight": "bolder"}, {"type": "Table", "firstRowAsHeader": true, "showGridLines": true, "columns": [{"width": 1}, {"width": 1}, {"width": 1}], "rows": [{"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "System Field"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "Delivery Field"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "Status"}]}]}, {"type": "TableRow", "cells": [{"type": "TableCell", "items": [{"type": "TextBlock", "text": "{leadFieldName}"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "{fieldName}"}]}, {"type": "TableCell", "items": [{"type": "TextBlock", "text": "✓ Mapped"}]}]}]}, {"type": "TextBlock", "text": "Successfully mapped {mappedCount} out of {totalCount} fields."}], "actions": [{"type": "Action.Submit", "title": "Continue", "data": {"action": "Continue"}}]}
```

  2. FOR EACH item in mappingSettings: look up the leadFieldName from leadFields where leadFieldUID matches the item's leadFieldUID. Duplicate the data row template, replacing {leadFieldName} with the actual leadFieldName and {fieldName} with item.fieldName.
  3. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to click Continue.

**State 8: Create Method With Mapping and Summarize**
* IF deliveryMethodUID is missing AND the user clicked Continue on the mapping preview:
  1. Call the create_delivery_method tool with these defaults:
     `clientUID={clientUID}, createDeliveryMethodDto={deliveryType="HttpPost", name="{companyName}-Webhook", enabled=true, leadTypeUID={leadTypeUID}, deliveryAddress={deliveryAddress}, contentType={mimeContentType}, responseSearch="success", useRegEx=false, settings={mappingSettings}, requestBody={requestBody}, deliveryDays={deliveryDays}}`
  2. If the tool fails, repair the payload and retry once silently. If it still fails, prompt: "I ran into an issue creating the delivery method.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
  3. If the tool succeeds, retain: deliveryMethodUID, deliveryMethodName="{companyName}-Webhook", deliveryType="HttpPost".
  4. Immediately call the summarize_history tool.

* IF deliveryMethodUID is known (from State 3 skip path):
  1. Immediately call the summarize_history tool.

## Summarization Requirements

When calling summarize_history:
- **start_anchor_substring:** "DELIVERY_SETUP_START"
- **summarization_text:** Format exactly as follows:

```
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
Load mcp://resource/rw-phase-3b-webhook-test
```

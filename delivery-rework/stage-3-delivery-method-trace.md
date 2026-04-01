# Stage 3 Delivery Method Trace

## Purpose

This trace records the reachable branch graph for the rewritten Stage 3 delivery-method unit in `delivery-rework/`.

## Resource Graph

Entry:
- `mcp://resource/phase-3-create-delivery-method`

Method routing:
- `Webhook -> mcp://resource/phase-3-webhook-delivery-method`
- `Portal | Email | FTP -> mcp://resource/phase-3-simple-delivery-methods`

Post-create:
- every method-specific resource summarizes from `anchor_delivery_method_start`
- every method-specific resource hands off to `mcp://resource/phase-3b-test-connection`
- `phase-3b-test-connection` summarizes from `anchor_delivery_method_start`
- `phase-3b-test-connection` hands off to `mcp://resource/phase-4-delivery-method-summary`

## Router Trace

Resource:
- `delivery-rework/resources/phase-3-create-delivery-method.md`

States:
1. Missing `deliveryScheduleChoice`
   - prompt `24/7 delivery | Specific hours only`
   - STOP AND YIELD
2. `Specific hours only` but missing `scheduleInput`
   - prompt for schedule text
   - STOP AND YIELD
3. Missing `deliveryDays`
   - build `deliveryDays`
   - retain `deliveryScheduleDisplay`
   - prompt for delivery type
   - STOP AND YIELD
4. `deliveryDays` known but missing `deliveryTypeChoice`
   - re-display delivery-type selector
   - STOP AND YIELD
5. `deliveryTypeChoice = Portal`
   - route to simple-methods resource
6. `deliveryTypeChoice = Webhook`
   - route to webhook resource
7. `deliveryTypeChoice = Email`
   - route to simple-methods resource
8. `deliveryTypeChoice = FTP`
   - route to simple-methods resource

## Simple-Methods Trace

Resource:
- `delivery-rework/resources/phase-3-simple-delivery-methods.md`

States:
1. Wrong branch loaded
   - missing `deliveryTypeChoice` routes back to the router
   - `Webhook` routes to the webhook resource
2. `deliveryTypeChoice = Portal` and missing `deliveryMethodUID`
   - create portal delivery method
   - on repairable DTO issue: retry once silently
   - on failure: prompt retry later message, STOP AND YIELD
3. `deliveryTypeChoice = Email` and missing `deliveryMethodUID`
   - create email delivery method
   - on repairable DTO issue: retry once silently
   - on failure: prompt retry later message, STOP AND YIELD
4. `deliveryTypeChoice = FTP` and missing `deliveryMethodUID`
   - if any FTP input is missing: prompt for all FTP details, STOP AND YIELD
   - otherwise create FTP delivery method
   - on repairable DTO issue: retry once silently
   - on failure: prompt retry later message, STOP AND YIELD
5. `deliveryMethodUID` known
   - summarize full delivery-method unit
   - hand off to Phase 3b

Retained output by branch:
- Portal:
  - `deliveryMethodName`
  - `deliveryType`
  - `deliveryAddress`
  - `mappedCount`
  - `totalCount`
  - `connectionTestMode="none"`
- Email:
  - `deliveryMethodName`
  - `deliveryType`
  - `deliveryAddress`
  - `mappedCount`
  - `totalCount`
  - `connectionTestMode="none"`
- FTP:
  - `deliveryMethodName`
  - `deliveryType`
  - `deliveryAddress`
  - `ftpUser`
  - `ftpPassword`
  - `mappedCount`
  - `totalCount`
  - `connectionTestMode="ftp"`

## Webhook Trace

Resource:
- `delivery-rework/resources/phase-3-webhook-delivery-method.md`

States:
1. Missing `deliveryAddress`
   - prompt for webhook URL
   - STOP AND YIELD
2. Missing `skipFieldMapping`
   - prompt `I'll provide instructions | Skip for now`
   - STOP AND YIELD
3. `skipFieldMapping = true` and missing `deliveryMethodUID`
   - create webhook without mappings
   - on failure: prompt retry later message, STOP AND YIELD
4. `skipFieldMapping = false` and missing `leadFields`
   - call `get_lead_type`
   - on failure: prompt retry later message, STOP AND YIELD
5. Missing `contentTypeChoice`
   - prompt for content type
   - STOP AND YIELD
6. Missing `postingInstructions`
   - prompt based on `contentTypeChoice`
   - STOP AND YIELD
7. `postingInstructions` contains external URL
   - prompt for pasted text instead
   - STOP AND YIELD
8. `postingInstructions` too large or unformatted
   - prompt `Simplify input | Skip mapping`
   - STOP AND YIELD
   - `Simplify input` -> request smaller excerpt, STOP AND YIELD
   - `Skip mapping` -> set `skipFieldMapping=true`, fall through to State 3 on re-entry
9. `contentTypeChoice = "I'm not sure"`
   - detect format
   - prompt `Continue with {detectedFormat} | Switch content type`
   - STOP AND YIELD
   - continue path -> retain detected type, fall through to mapping build
   - switch path -> clear content type and instructions, fall through to State 5 on re-entry
10. `contentTypeChoice in {JSON, XML}` and parse still fails
   - prompt `Re-paste {contentTypeChoice} schema | Switch content type`
   - STOP AND YIELD
   - re-paste path -> clear instructions, prompt schema again, STOP AND YIELD
   - switch path -> clear content type and instructions, fall through to State 5 on re-entry
11. Build mappings
   - extract delivery fields
   - match against `leadFields`
   - if ambiguous: prompt clarification, STOP AND YIELD
   - if no confident match: prompt clarification, STOP AND YIELD
   - build `mappingSettings`, `requestBody`, `mappedCount`, `totalCount`, `mimeContentType`
12. Mapping preview
   - display preview card
   - STOP AND YIELD
13. Preview acknowledged
   - create webhook with mappings
   - on failure: prompt retry later message, STOP AND YIELD
14. `deliveryMethodUID` known
   - summarize full delivery-method unit
   - hand off to Phase 3b

Retained output:
- `deliveryMethodName`
- `deliveryType`
- `deliveryAddress`
- `mimeContentType`
- `requestBody`
- `mappedCount`
- `totalCount`
- `connectionTestMode="webhook"`

## Connection-Test Trace

Resource:
- `delivery-rework/resources/phase-3b-test-connection.md`

States:
1. `connectionTestMode` missing or `none`
   - summarize full delivery-method unit
   - hand off to Phase 4
2. `connectionTestMode in {ftp, webhook}` and prompt not yet answered
   - prompt `Test Connection | Skip`
   - STOP AND YIELD
3. FTP test path
   - detect `protocol`
   - run FTP/SFTP test
   - success -> State 5
   - failure -> State 6
4. Webhook test path
   - run webhook test
   - use generated sample payload when mappings exist
   - otherwise use empty payload
   - success -> State 5
   - failure -> State 6
5. Success acknowledgement
   - prompt `Continue`
   - STOP AND YIELD
6. Failure acknowledgement
   - prompt `Retry | Skip`
   - STOP AND YIELD
7. Summarization exit
   - user chose `Skip`
   - or user chose `Continue` after success
   - summarize and hand off to Phase 4

## Validation Notes

Confirmed statically:
- one shared anchor: `anchor_delivery_method_start`
- router never summarizes
- every method-specific resource summarizes before Phase 3b
- Phase 3b re-summarizes and re-retains the full delivery-method state for Phase 4 and Phase 5
- delivery-method eligibility for connection test now uses explicit `connectionTestMode`

Open review focus for agent cross-check:
- whether webhook repair states need even more explicit retained markers for “prompt already shown”
- whether the extra router -> method-resource transition is justified by the branch-clarity gain
- whether Phase 4 should stay summary-only or also be split by `flowIntent`

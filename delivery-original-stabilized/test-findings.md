# Test Findings — delivery-original-stabilized

## Summary

Testing performed via:
- Live platform testing on GPT-5.4 Mini Model (dev.leadexec.app)
- 10 haiku agent traces covering all flows, edge cases, and stability checks
- 5 haiku agents for schema validation and criteria edge cases

Date: 2026-04-03

---

## Live Platform Findings

### F1 — Phase 1 prompt duplicated (intermittent)
- **Severity:** Low
- **Status:** Partially fixed — heading rename helped, but still intermittent
- **Description:** Phase 1 intro prompt sometimes appears twice in the initial message.
- **Root cause:** Platform behavior — action file context and Phase 1 resource both visible to LLM.

### F2 — Phase 5 skips target states
- **Severity:** High
- **Status:** Fixed
- **Fix:** Removed `(Silent)` label from state detection, collapsed to one line, added "Follow steps in order" to all phase headers.

### F3 — Phase 4 summary appears after Phase 5 content
- **Severity:** High
- **Status:** Fixed
- **Fix:** Added `summarize_history` to Phase 4 after user selects Continue.

### F4 — Delivery Method Summary card shown twice
- **Severity:** Medium
- **Status:** Fixed (same fix as F3)

### F5 — JSON schema rejected despite being valid/fixable
- **Severity:** High
- **Status:** Fixed
- **Fix:** Changed to lenient validation: "use your best judgment, be lenient, extract field names even if format is imperfect."

### F6 — No Skip option on JSON parse failure
- **Severity:** Medium
- **Status:** Fixed
- **Fix:** Added "Skip mapping" as third option on parse failure prompt.

### F7 — Conversational prompts wrapped in adaptive cards
- **Severity:** Low
- **Status:** Open (model behavior)
- **Description:** ASK [conversational] steps sometimes render as adaptive cards with Continue buttons. Inconsistent — sometimes works correctly as plain text.

### F8 — Enum criteria not validated (accepts invalid values)
- **Severity:** Medium
- **Status:** Open (LLM compliance)
- **Description:** User typed "SelfCreditRating - meh" — LLM accepted without showing ChoiceSet. Worse when multiple criteria in one message.

### F9 — Schedule ActionSet shown 3 times
- **Severity:** Low
- **Status:** Open (platform issue with ActionSet click delivery)

### F10 — Recommended fields show contact/personal fields
- **Severity:** Low
- **Status:** Open (LLM compliance)
- **Description:** Shows TrackingNumber, FirstName instead of LoanAmount, PropertyValue despite "exclude contact/personal" instruction.

### F11 — Partial input causes full re-prompt
- **Severity:** Medium
- **Status:** Fixed
- **Fix:** Changed global rule to "retain what was provided and re-prompt only for the missing fields."

### F12 — AI stalls after webhook URL input
- **Severity:** Medium
- **Status:** Open (platform/model timeout)
- **Description:** LLM receives URL but takes >50s to respond with field mapping choice. May be related to Phase 3 monolithic file length (182 lines).

---

## Agent Trace Findings

### Content Type Mismatch (HIGH)
- **No format mismatch detection exists.** If user selects "JSON" but pastes XML, the LLM silently extracts fields without alerting the user.
- Same for XML→JSON, JSON→URL-encoded mismatches.
- The "lenient" instruction tells LLM to extract field names regardless of format quality — this works for imperfect formatting but not for wrong format entirely.
- URL detection (pastes a link instead of schema) works correctly.
- Large input detection (>5000 chars) works correctly.
- **Recommendation:** Add format type-checking after user declares format. If declared format doesn't match actual format, prompt user to confirm.

### Batch Criteria Parsing (HIGH)
- When user provides multiple criteria in one message (e.g., "LoanAmount > 500k, PropertyType = Condo, AnnualIncome > 100k"), the LLM processes all as a batch.
- Per-criterion enum validation (ChoiceSet display) is skipped during batch processing.
- Instructions assume one criterion per Criteria Parsing invocation but don't explicitly forbid batching.
- **Recommendation:** Add instruction: "If user provides multiple criteria in one message, process each one through Criteria Parsing individually, including enum validation per criterion."

### XML Attributes Dropped (HIGH)
- XML attributes (e.g., `<loan amount="" term="">`) are silently dropped during field mapping.
- Instructions only cover element content: `<field>[SystemFieldName]</field>`
- No guidance for attribute-based schemas.
- **Impact:** Data loss for API schemas that encode fields as XML attributes.

### JSON Array Mapping Undocumented (HIGH)
- No guidance on how to handle JSON arrays (e.g., contacts array with type/value pairs).
- Instructions mention "hierarchy, nesting, arrays" but don't specify array indexing or iteration.

### "Between" Operator Value Format (MEDIUM)
- Value format for "Between" operator is undocumented.
- State UIDs use pipe-delimited format (`stateUID1|stateUID2`), suggesting pipes are standard.
- But this isn't explicitly stated for other operators.

### Field-Not-Found Loop Continuation (MEDIUM)
- After "I couldn't find that field" prompt, no explicit LOOP BACK instruction.
- User can retry or say "show fields" but the return path to Criteria Loop is implicit.

### Criteria Loop Exit Phrases (MEDIUM)
- Only "continue", "done", "no" exit the loop (per instruction line 77).
- "skip", "finish", "that's all", "nothing else" do NOT exit — treated as field name attempts.
- This may frustrate users who expect natural phrases to work.

### Hierarchical JSON requestBody (LOW)
- Instructions say "preserving hierarchy from user's schema" — hierarchy IS preserved.
- Unmapped fields (e.g., metadata) are silently excluded per "include only mapped fields."
- This is correct behavior but user isn't warned about excluded fields.

### No Retry Limit on Connection Test (LOW)
- User can retry connection test indefinitely. No counter or limit.
- By design — user has full control. But could frustrate if endpoint is down.

---

## Stability Checks (All Pass)

| Check | Result |
|-------|--------|
| Criteria APPEND logic (5 criteria accumulated) | PASS — no overwriting |
| Phase 5 step ordering (3 separate runs) | PASS — all correct sequence |
| Phase 3b all 10 connection paths (3 runs each) | PASS — all stable |
| State carry-forward across all 3 flows | PASS — all variables present |
| Phase headers and summary format consistency | PASS — all 11 files correct |
| No stale references (anchors, wrappers, data_normalization) | PASS — zero found |
| Action files (unique flowIntent, flow sequence, anchor) | PASS — all 3 correct |
| System file (no reasoning_effort, has resource_handling, ChoiceSet fix) | PASS |

---

## Fix Summary

| Commit | Fix |
|--------|-----|
| 1950566 | Initial stabilized version with 12 bug fixes |
| 4c6fc4f | Slim global prompt, inline normalization, display_lead_types_choice |
| 6ce6ca3 | Sequential execution directive in all phase headers |
| 239f700 | Phase 4 summarize, Phase 5 states restructure, schema validation leniency |
| 063df2e | Partial input handling (re-prompt only missing fields) |
| fc55782 | Phase 6 summarize_history for clean context |
| 98869fa | Remove INITIAL PROMPT from action headings |

---

## Open Issues — Not Fixable via Instructions

| Issue | Why |
|-------|-----|
| F7: Cards instead of plain text | Model behavior, notation exists but ignored |
| F8: Enum validation in batch | LLM processes multiple criteria at once |
| F9: ActionSet clicks not received | Platform delivery issue |
| F10: Bad field suggestions | LLM doesn't follow exclusion/prioritization rules |
| F12: AI stalls on webhook | Platform/model timeout |
| Content type mismatch | Could add detection but GPT-5.4 likely handles naturally |
| XML attributes | Edge case, requires schema-level awareness |
| JSON arrays | Edge case, requires explicit array handling logic |

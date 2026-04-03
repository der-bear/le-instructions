# Test Findings — delivery-original-stabilized

## Summary

Testing performed via:
- Live platform testing on GPT-5.4 Mini Model (dev.leadexec.app) — multiple runs
- 20 haiku agent traces covering all flows, edge cases, and stability checks
- 1 opus agent for stability test protocol creation
- 1 haiku agent for fix plan preparation

Date: 2026-04-03

---

## Live Platform Findings

### F1 — Phase 1 prompt duplicated (intermittent)
- **Severity:** Low
- **Status:** Partially fixed — intermittent (some runs show it, some don't)
- **Description:** Phase 1 intro prompt sometimes appears twice in the initial message.

### F2 — Phase 5 target states reordered
- **Severity:** High
- **Status:** PARTIALLY FIXED — no longer fully skipped, but reordered
- **Description:** Target states prompt now appears but AFTER the criteria loop exit, not BEFORE field suggestions as instructed. The flow goes: Price → Exclusive → Order → Field Suggestions → Criteria → "done" → Target States. Should be: Price → Exclusive → Order → Target States → Field Suggestions → Criteria.
- **Root cause:** Despite "Follow steps in order from top to bottom. Do NOT skip ahead." directive and removing the Silent label, the LLM still reorders. The CRITICAL directive on field suggestions is louder than the plain states prompt above it.
- **Attempted fixes:** Removed Silent label, collapsed state detection to one line, added CRITICAL before states, added sequential execution directive. None fully resolved.
- **Recommendation:** May need numbered states (like rework) or Phase 5 splitting to enforce order.

### F3 — Phase 4 summary appears after Phase 5 content
- **Severity:** High
- **Status:** Fixed
- **Fix:** Added summarize_history to Phase 4.

### F4 — Delivery Method Summary card shown twice
- **Severity:** Medium
- **Status:** Fixed (same fix as F3)

### F5 — JSON schema rejected despite being valid/fixable
- **Severity:** High
- **Status:** Fixed
- **Fix:** Changed to lenient validation. **Confirmed working in live test** — JSON without outer braces was accepted and processed.

### F6 — No Skip option on JSON parse failure
- **Severity:** Medium
- **Status:** Fixed

### F7 — Conversational prompts wrapped in adaptive cards
- **Severity:** Low
- **Status:** Open (intermittent model behavior)
- **Description:** Sometimes renders as cards, sometimes as plain text. Inconsistent.

### F8 — Enum criteria not validated (accepts invalid values)
- **Severity:** Medium
- **Status:** Open (LLM compliance)

### F9 — Schedule ActionSet shown 3 times
- **Severity:** Low
- **Status:** Open (platform issue with ActionSet click delivery)

### F10 — Recommended fields show contact/personal fields
- **Severity:** Medium
- **Status:** Open (confirmed in every run)
- **Description:** Shows TrackingNumber, RequestAssignmentDate, ContactPhoneExtension, ConsumerGeoPhoneAreaCode instead of LoanAmount, PropertyValue. Instructions say "Exclude contact/personal information lead fields" and "Prioritize top relevant industry-specific lead qualification business criteria" but LLM consistently ignores this.

### F11 — Partial input causes full re-prompt
- **Severity:** Medium
- **Status:** Fixed

### F12 — AI stalls during webhook path transitions
- **Severity:** High
- **Status:** Open (confirmed 3 times across multiple runs)
- **Description:** LLM stalls for >60s during summarize_history transitions in the webhook path. Confirmed at: (1) after webhook URL input, (2) after connection test skip. The conversation context after webhook+mapping is very large.
- **Root cause:** Phase 3 monolithic file (182 lines). The rework split this into separate files for exactly this reason.
- **Recommendation:** Split Phase 3 into router + type-specific files.

### F13 — Field mapping preview card skipped
- **Severity:** High
- **Status:** Open
- **Description:** After JSON parsing succeeded (lenient validation worked), the AI skipped the field mapping preview table card and went directly to Phase 3b connection test. Instructions say "CRITICAL: MUST execute the DISPLAY [adaptive_card] below IMMEDIATELY."

### F14 — Connection test hallucinated for Portal
- **Severity:** High
- **Status:** Open
- **Description:** Portal delivery type (connectionTestMode="none") should skip Phase 3b entirely. Instead, the AI shows a hallucinated prompt: "I'm ready to run the connection test for the delivery method. Please confirm: type 'continue' to run the test now or 'cancel' to stop." This prompt does NOT exist in any instruction file — it's completely fabricated by the LLM.
- **Root cause:** The LLM is not reading the connectionTestMode flag correctly, or Phase 3b's `IF connectionTestMode = "none": Go directly to summarize_history below.` is not being followed.

---

## Agent Trace Findings

### Content Type Mismatch (MEDIUM)
- No format mismatch detection exists. Could add but GPT-5.4 likely handles naturally.

### Batch Criteria Parsing (MEDIUM)
- Multiple criteria in one message skip per-criterion enum validation.
- Add instruction: "Process each criterion individually through Criteria Parsing."

### "Between" Operator Value Format (MEDIUM)
- Undocumented. Should use pipe-delimited format like state UIDs.

### Field-Not-Found Loop Continuation (MEDIUM)
- No explicit LOOP BACK after "I couldn't find that field" prompt.

### Criteria Loop Exit Phrases (LOW)
- Only "continue"/"done"/"no" exit. "skip"/"finish" don't.

---

## What's Working

| Feature | Status |
|---------|--------|
| Phase 1 client creation | ✓ Works reliably |
| Phase 2 display_lead_types_choice | ✓ Works correctly |
| Phase 3 schedule parsing | ✓ Works (24/7 tested) |
| Phase 3 delivery type selection | ✓ Works |
| Phase 4 summary in correct position | ✓ Fixed with summarize_history |
| Phase 5 price → exclusive → order ordering | ✓ Works correctly |
| Phase 5 criteria loop re-entry | ✓ Works ("Would you like to add another?") |
| Phase 5 "done" text exit from criteria loop | ✓ Works |
| Criteria APPEND logic | ✓ Verified in agent traces |
| State carry-forward across all phases | ✓ Verified |
| Single anchor DELIVERY_SETUP_START | ✓ All phases use it |
| JSON lenient validation | ✓ Accepts JSON without braces |
| Partial input handling | ✓ Asks only for missing fields |

---

## Persistent Issues Requiring Architectural Changes

These issues cannot be fixed with instruction-level tweaks alone:

| Issue | Root Cause | Recommended Fix |
|-------|-----------|-----------------|
| F2: Target states reordered | LLM doesn't follow sequential order in 160-line file | Split Phase 5 or add numbered states |
| F12: Webhook path stalls | Phase 3 monolithic file too large for summarize_history | Split Phase 3 into router + type files |
| F13: Mapping preview skipped | LLM rushes through long Phase 3 | Split Phase 3 |
| F14: Portal connection test hallucinated | LLM ignores connectionTestMode flag | Stronger Phase 3b structure or separate no-test path |
| F10: Bad field suggestions | LLM ignores exclusion/prioritization rules | May need explicit field list or tool-based suggestions |

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

# Issue Mapping — Test Findings to Fix Plan

## HIGH Severity Issues

### F2 — Phase 5 skips target states
- **Status:** Fixed (per test findings)
- **Fix applied:** Removed `(Silent)` label, collapsed state detection to one line, added "Follow steps in order" to phase headers
- **No additional fix required**

### F5 — JSON schema rejected despite being valid/fixable
- **Status:** Fixed (per test findings)
- **Fix applied:** Lenient validation with "use your best judgment, be lenient"
- **No additional fix required**

### Content Type Mismatch Detection
- **Source:** Agent Trace Findings, line 79-85
- **Severity:** HIGH
- **Status:** OPEN — Needs instruction fix
- **Proposed Fix:** FIX_PLAN Issue #1
- **Why not auto-fixed:** Lenient validation (F5 fix) handles imperfect formatting but not wrong format entirely
- **Instruction Location:** phase-3-create-delivery-method.md, lines 100-110

### Batch Criteria Parsing
- **Source:** Agent Trace Findings, line 87-91
- **Severity:** HIGH
- **Status:** OPEN — Needs instruction fix
- **Proposed Fix:** FIX_PLAN Issue #2
- **Why not auto-fixed:** Instructions don't forbid batching or enforce per-criterion validation
- **Instruction Location:** phase-5-create-delivery-account.md, lines 68-77

---

## MEDIUM Severity Issues

### F6 — No Skip option on JSON parse failure
- **Status:** Fixed (per test findings)
- **Fix applied:** Added "Skip mapping" as third option on parse failure prompt
- **No additional fix required**

### F11 — Partial input causes full re-prompt
- **Status:** Fixed (per test findings)
- **Fix applied:** Changed to "retain what was provided and re-prompt only for missing fields"
- **No additional fix required**

### "Between" Operator Value Format
- **Source:** Agent Trace Findings, line 103-106
- **Severity:** MEDIUM
- **Status:** OPEN — Needs instruction fix
- **Proposed Fix:** FIX_PLAN Issue #3
- **Why not auto-fixed:** Undocumented format creates ambiguity; pipe-delimited format implied but not explicit
- **Instruction Location:** phase-5-create-delivery-account.md, lines 128-135

### Field-Not-Found Loop Continuation
- **Source:** Agent Trace Findings, line 108-110
- **Severity:** MEDIUM
- **Status:** OPEN — Needs instruction fix
- **Proposed Fix:** FIX_PLAN Issue #4
- **Why not auto-fixed:** No explicit LOOP BACK instruction after field-not-found prompt; return path is implicit
- **Instruction Location:** phase-5-create-delivery-account.md, lines 125-127

### Criteria Loop Exit Phrases
- **Source:** Agent Trace Findings, line 112-115
- **Severity:** MEDIUM
- **Status:** OPEN — Needs instruction fix
- **Proposed Fix:** FIX_PLAN Issue #5
- **Why not auto-fixed:** Only three exit phrases documented; natural variants like "skip", "finish" not recognized
- **Instruction Location:** phase-5-create-delivery-account.md, lines 77, 69

---

## Issues NOT Addressed (Out of Scope)

### Platform/Model Behavior Issues

These cannot be fixed via instructions:

| Issue | Test Finding | Severity | Reason |
|-------|--------------|----------|--------|
| F1 — Phase 1 prompt duplicated | Intermittent | LOW | Platform behavior — action file context + Phase 1 resource both visible |
| F3 — Phase 4 summary after Phase 5 | **Fixed** | HIGH | Already fixed with summarize_history |
| F4 — Delivery Method Summary card shown twice | **Fixed** | MEDIUM | Same fix as F3 |
| F7 — Conversational prompts wrapped in cards | Intermittent | LOW | Model behavior — notation exists but ignored |
| F8 — Enum criteria not validated | Batch issue | MEDIUM | LLM processes multiple criteria at once (part of Issue #2) |
| F9 — Schedule ActionSet shown 3 times | Platform issue | LOW | ActionSet click delivery issue |
| F10 — Recommended fields show contact/personal | LLM compliance | LOW | LLM doesn't follow exclusion rules |
| F12 — AI stalls after webhook URL input | Timeout | MEDIUM | Platform/model timeout, possibly Phase 3 file size |

### Architectural Edge Cases (Not Fixable via Instructions)

| Issue | Test Finding | Severity | Reason |
|-------|--------------|----------|--------|
| XML attributes dropped | Edge case | HIGH | No guidance for attribute-based schemas; requires schema-level awareness |
| JSON array mapping undocumented | Edge case | HIGH | No array indexing/iteration logic; undocumented feature |
| Hierarchical JSON requestBody | Expected behavior | LOW | Correct — unmapped fields excluded; user isn't warned but per design |
| No retry limit on connection test | By design | LOW | User has full control; no fix needed |

---

## Summary

**Total HIGH severity issues to fix:** 2
- Issue #1: Content Type Mismatch Detection
- Issue #2: Batch Criteria Parsing

**Total MEDIUM severity issues to fix:** 3
- Issue #3: "Between" Operator Value Format
- Issue #4: Field-Not-Found Loop Continuation
- Issue #5: Criteria Loop Exit Phrases

**Already fixed:** 3 issues (F2, F3-F4, F5-F6, F11)

**Not fixable via instructions:** 8 issues (platform/model behavior or architectural edge cases)

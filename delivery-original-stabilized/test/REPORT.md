# Fix Plan Report — Instruction-Level Issues

**Analysis Date:** 2026-04-03
**Status:** Complete — Ready for implementation

---

## Executive Summary

Analysis of 5 HIGH and MEDIUM severity issues from test findings reveals:

- **2 HIGH severity** issues requiring fixes
- **3 MEDIUM severity** issues requiring fixes
- **2 files** require modification
- **~35 lines** of changes (additions, replacements, modifications)
- **No LOW severity** issues proposed per scope

All fixes are **instruction-level** (not platform/model behavior). No architectural refactoring required.

---

## Issues Identified

### HIGH Severity

1. **Content Type Mismatch Detection** (Issue #1)
   - File: phase-3-create-delivery-method.md, lines 100–110
   - Problem: Lenient validation handles imperfect formatting but not wrong format (e.g., JSON declared, XML pasted)
   - Fix: Add format mismatch detection even when user pre-selects type
   - Impact: Prevents silent data loss from format mismatches

2. **Batch Criteria Parsing** (Issue #2)
   - File: phase-5-create-delivery-account.md, lines 68–77
   - Problem: Multiple criteria in one message processed as batch; enum validation skipped
   - Fix: Detect batch input and process each criterion individually
   - Impact: Ensures per-criterion enum validation via ChoiceSet

### MEDIUM Severity

3. **"Between" Operator Value Format** (Issue #3)
   - File: phase-5-create-delivery-account.md, lines 128–135
   - Problem: Undocumented value format for "Between" operator
   - Fix: Explicitly document pipe-delimited format (e.g., "500000|1000000")
   - Impact: Removes ambiguity, clarifies intent

4. **Field-Not-Found Loop Continuation** (Issue #4)
   - File: phase-5-create-delivery-account.md, lines 125–127
   - Problem: After "couldn't find field" prompt, no explicit loop-back instruction
   - Fix: Add explicit handling for "show fields", retry, skip → loop back
   - Impact: Prevents LLM from getting stuck or exiting loop prematurely

5. **Criteria Loop Exit Phrases** (Issue #5)
   - File: phase-5-create-delivery-account.md, lines 69, 77
   - Problem: Only "continue"/"done"/"no" recognized as exits
   - Fix: Expand exit phrase list to include "skip", "finish", "that's all", etc.
   - Impact: Allows natural language exit without treating input as field name

---

## Files Affected

### phase-3-create-delivery-method.md
- **Changes:** 1 expansion (Issue #1)
- **Lines:** 100–110 (insert after line 110)
- **Scope:** ~11 lines added

### phase-5-create-delivery-account.md
- **Changes:** 4 modifications (Issues #2, #3, #4, #5)
- **Lines:** 69, 72, 125–127, 132–133, 77
- **Scope:** ~24 lines modified/added

---

## Risk Assessment

| Issue | Risk Level | Reason | Mitigation |
|-------|-----------|--------|-----------|
| #1 | Medium | Adds prompt in format mismatch cases | Only shown if mismatch detected |
| #2 | Medium | Adds prompts for batch input | Clearly signals multi-criterion processing |
| #3 | Low | Documentation only | No behavioral change, clarifies existing intent |
| #4 | Low | Improves clarity | Explicit loop-back prevents ambiguity |
| #5 | Low | Extends exit phrase list | Natural language improvement, no logic change |

**Overall Risk:** Low-Medium (all changes additive or clarifying, no breaking changes)

---

## Testing Requirements

### Agent Trace Tests (Recommended)

1. **Format mismatch scenarios**
   - User declares JSON, pastes XML → expects confirmation prompt
   - User declares XML, pastes JSON → expects confirmation prompt

2. **Batch criteria input**
   - User enters "LoanAmount > 500k, PropertyType = Condo, AnnualIncome > 100k"
   - Verify: each criterion processed separately with enum validation if needed

3. **Between operator serialization**
   - Verify: "Between 500000 and 1000000" → value field contains "500000|1000000"

4. **Field-not-found handling**
   - User enters invalid field name
   - User selects "show fields" → displays all available fields
   - Verify: proper loop-back to Criteria Loop (not exit)

5. **Exit phrase variations**
   - Test: "skip", "finish", "that's all", "nothing else", "no more"
   - Verify: all exit Criteria Loop without being treated as field names

### Stability Checks
- Verify: Criteria APPEND logic still works (no overwriting)
- Verify: State carry-forward unchanged
- Verify: All phase headers consistent

---

## Deliverables

Three documents provided:

1. **FIX_PLAN.md** — Detailed fix specifications with quotes, proposed text, rationale, and risks
2. **IMPLEMENTATION_LOCATIONS.md** — Exact file paths, line numbers, and implementation instructions
3. **ISSUE_MAPPING.md** — Mapping from test findings to fixes, showing what's already fixed vs. open

---

## Implementation Notes

### Recommended Order
1. Issue #5 (exit phrases) — lowest risk, fastest to verify
2. Issue #3 (Between format) — documentation only, unblocks other work
3. Issue #4 (field-not-found loop) — medium complexity, clear scope
4. Issue #1 (content type mismatch) — affects Phase 3, moderate complexity
5. Issue #2 (batch criteria) — highest complexity, most critical

### Estimated Effort
- Implementation: 1–2 hours
- Testing: 45 minutes (focused agent traces)
- Total: ~2–2.5 hours

### Rollback Strategy
All changes are additive/clarifying. If issues arise:
1. Remove batch detection logic (Issue #2) first — most complex
2. Remove format validation (Issue #1) — affects Phase 3
3. Exit phrases and documentation (Issues #3, #5) are safe to keep

---

## Sign-Off

**Analysis Complete:** ✓
**All HIGH/MEDIUM issues addressed:** ✓
**Scope limited to instruction-level only:** ✓
**Ready for implementation:** ✓

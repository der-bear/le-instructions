# Fix Plan Index — Navigation Guide

**Prepared:** 2026-04-03  
**Analysis Scope:** HIGH and MEDIUM severity issues from test findings  
**Status:** Ready for implementation

---

## Start Here

**→ Read first:** [REPORT.md](./REPORT.md) (5 minutes)
- Executive summary of 5 issues
- Risk assessment
- Testing requirements

---

## Detailed Planning Documents

| Document | Purpose | Audience | Length |
|----------|---------|----------|--------|
| [REPORT.md](./REPORT.md) | Executive overview and sign-off | Managers, tech leads | 3 pages |
| [FIX_PLAN.md](./FIX_PLAN.md) | Detailed specifications for each fix | Implementers, code reviewers | 8 pages |
| [IMPLEMENTATION_LOCATIONS.md](./IMPLEMENTATION_LOCATIONS.md) | Exact line numbers and file paths | Implementers | 4 pages |
| [ISSUE_MAPPING.md](./ISSUE_MAPPING.md) | Test findings → fix mapping | QA, analysts | 4 pages |
| [FIX_SUMMARY.txt](./FIX_SUMMARY.txt) | One-page summary (terminal-friendly) | Quick reference | 1 page |

---

## The 5 Issues

### HIGH Severity (2 issues)

**Issue #1: Content Type Mismatch Detection**
- **Problem:** Silent extraction when user declares JSON but pastes XML
- **Fix Location:** phase-3-create-delivery-method.md, lines 100–110
- **See:** FIX_PLAN.md (Issue 1), IMPLEMENTATION_LOCATIONS.md (Fix #1)
- **Status:** Specification ready

**Issue #2: Batch Criteria Parsing**
- **Problem:** Multiple criteria processed as batch, enum validation skipped
- **Fix Location:** phase-5-create-delivery-account.md, lines 68–77
- **See:** FIX_PLAN.md (Issue 2), IMPLEMENTATION_LOCATIONS.md (Fix #2)
- **Status:** Specification ready

### MEDIUM Severity (3 issues)

**Issue #3: "Between" Operator Value Format**
- **Problem:** Undocumented format for Between operator values
- **Fix Location:** phase-5-create-delivery-account.md, lines 128–135
- **See:** FIX_PLAN.md (Issue 3), IMPLEMENTATION_LOCATIONS.md (Fix #3)
- **Status:** Specification ready

**Issue #4: Field-Not-Found Loop Continuation**
- **Problem:** No explicit loop-back after "couldn't find field" prompt
- **Fix Location:** phase-5-create-delivery-account.md, lines 125–127
- **See:** FIX_PLAN.md (Issue 4), IMPLEMENTATION_LOCATIONS.md (Fix #4)
- **Status:** Specification ready

**Issue #5: Criteria Loop Exit Phrases**
- **Problem:** Only 3 exit phrases recognized, natural variants treated as field names
- **Fix Location:** phase-5-create-delivery-account.md, lines 69, 77
- **See:** FIX_PLAN.md (Issue 5), IMPLEMENTATION_LOCATIONS.md (Fix #5)
- **Status:** Specification ready

---

## Files to Modify

### File 1: phase-3-create-delivery-method.md
- **Location:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-3-create-delivery-method.md`
- **Changes:** 1 fix (Issue #1)
- **Lines:** 100–110 (add 11 lines)
- **Risk:** Medium (adds conditional prompt)

### File 2: phase-5-create-delivery-account.md
- **Location:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`
- **Changes:** 4 fixes (Issues #2, #3, #4, #5)
- **Lines:** 69, 72, 125–127, 132–133, 77
- **Risk:** Low–Medium (mostly clarifications and expansions)

---

## How to Implement

### Step 1: Read Specifications
1. Read REPORT.md (executive summary)
2. Read FIX_PLAN.md (detailed specs for each issue)
3. Review IMPLEMENTATION_LOCATIONS.md (exact line numbers)

### Step 2: Implement in Recommended Order
1. Issue #5 (exit phrases) — easiest, lowest risk
2. Issue #3 (Between format) — documentation only
3. Issue #4 (field-not-found loop) — moderate complexity
4. Issue #1 (content type mismatch) — affects Phase 3
5. Issue #2 (batch criteria) — most complex, highest value

### Step 3: Test Each Fix
- Use focused agent trace tests (see REPORT.md, "Testing Requirements")
- Verify each fix independently before moving to next
- Run stability checks after final fix

### Step 4: Commit
- Commit all 5 fixes together with reference to this plan
- Update git history with issue numbers

---

## Key Metrics

| Metric | Value |
|--------|-------|
| Total issues to fix | 5 |
| HIGH severity | 2 |
| MEDIUM severity | 3 |
| Files modified | 2 |
| Total lines changed | ~35 |
| Implementation time | 1–2 hours |
| Testing time | 45 minutes |
| Risk level | Low–Medium |

---

## Reference: Issues NOT in This Plan

The following issues from test findings are NOT addressed because they are out of scope (platform/model behavior, not instruction-level):

| Issue | Severity | Reason |
|-------|----------|--------|
| F1: Phase 1 prompt duplicated | LOW | Platform behavior |
| F7: Conversational prompts wrapped in cards | LOW | Model behavior |
| F8: Enum validation in batch | MEDIUM | LLM processes batch (related to Issue #2) |
| F9: Schedule ActionSet shown 3 times | LOW | Platform delivery issue |
| F10: Bad field suggestions | LOW | LLM compliance issue |
| F12: AI stalls after webhook | MEDIUM | Platform/model timeout |
| XML attributes dropped | HIGH | Edge case, schema-level awareness |
| JSON arrays undocumented | HIGH | Edge case, requires array logic |

Already fixed in previous commits: F2, F3–F4, F5–F6, F11

---

## Questions?

- **For fix specifications:** See FIX_PLAN.md
- **For implementation details:** See IMPLEMENTATION_LOCATIONS.md
- **For test findings context:** See ISSUE_MAPPING.md
- **For quick summary:** See FIX_SUMMARY.txt

---

**Ready to implement?** Start with REPORT.md, then move to IMPLEMENTATION_LOCATIONS.md.

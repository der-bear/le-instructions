# Fix Plan — delivery-original-stabilized

## Priority 1: Architectural Changes (HIGH)

These require structural changes, not just instruction tweaks.

### Fix A: Split Phase 3 (Addresses F12, F13)
**Problem:** Phase 3 is 182 lines. The webhook section alone is ~130 lines. This causes:
- Summarize_history stalls (>60s) during webhook path transitions
- Field mapping preview card skipped (LLM rushes through)

**Proposed fix:** Split Phase 3 into router + type-specific files (like the rework):
- `phase-3-create-delivery-method.md` → router only (schedule + type selection, ~30 lines)
- `phase-3-portal.md` → Portal creation (~15 lines)
- `phase-3-email.md` → Email creation (~15 lines)
- `phase-3-ftp.md` → FTP credentials + creation (~25 lines)
- `phase-3-webhook.md` → Webhook URL + mapping (~130 lines)

**Impact:** Reduces per-phase context, fixes stalls, makes mapping preview more reliable.
**Risk:** Adds more phase transitions and summarize_history calls.

### Fix B: Phase 5 Target States Ordering (Addresses F2)
**Problem:** Target states is consistently reordered — appears after criteria instead of before field suggestions. Multiple instruction-level fixes attempted (CRITICAL directive, removed Silent label, "Do NOT skip ahead") — none fully resolved.

**Option B1: Numbered states** (like rework)
Add explicit state numbers to Phase 5:
```
State 1: Collect price (WAIT)
State 2: Collect exclusivity (WAIT)
State 3: Collect order system (WAIT)
State 4: Load lead fields, detect state field, ask target states (WAIT)
State 5: Show field suggestions, criteria loop (WAIT)
State 6: Build criteria array, create account
```

**Option B2: Split Phase 5**
Extract criteria builder into Phase 5c (like rework).

**Option B3: Stronger directive**
More aggressive instruction at top of Phase 5:
```
MANDATORY STEP ORDER — do not reorder:
1. Price per lead
2. Exclusive or Shared
3. Order System
4. Target States ← MUST come before field suggestions
5. Field Suggestions and Criteria
```

**Recommendation:** Try B3 first (least invasive). If still fails, go to B1 or B2.

### Fix C: Phase 3b Portal/Email Skip (Addresses F14)
**Problem:** Phase 3b shows hallucinated connection test prompt for Portal (connectionTestMode="none"). The `IF connectionTestMode = "none": Go directly to summarize_history below.` is not followed.

**Proposed fix:** Make Phase 3 skip Phase 3b entirely for Portal/Email:
- Portal/Email summarize directly to Phase 4 (never load Phase 3b)
- FTP/Webhook summarize to Phase 3b as before

If Phase 3 is split (Fix A), this happens naturally — Portal and Email files summarize to Phase 4.

---

## Priority 2: Instruction Fixes (MEDIUM)

### Fix D: Content Type Mismatch Detection
**Where:** Schema Validation section in Phase 3 webhook

Add format type-checking when user pre-selects a format. If declared format doesn't match actual format, prompt user to confirm.

### Fix E: Batch Criteria Processing
**Where:** Handle User Response section in Phase 5

Add: "If user provides multiple criteria in one message, process each one individually through Criteria Parsing, including enum validation per criterion."

### Fix F: "Between" Operator Format
**Where:** Criteria Parsing section in Phase 5

Document pipe-delimited format for Between values (e.g., "100000|500000").

### Fix G: Field-Not-Found Loop Back
**Where:** After "I couldn't find that field" prompt in Phase 5

Add explicit LOOP BACK instruction and "done" as escape option.

### Fix H: Criteria Loop Exit Phrases
**Where:** Criteria Loop exit condition in Phase 5

Expand exit phrases: add "skip", "finish", "that's all", "nothing else", "no more".

### Fix I: Field Suggestions Quality
**Where:** Build Field Suggestions section in Phase 5

Add explicit exclusion list (contact, tracking, system fields) and inclusion list (financial, qualification, demographic fields).

---

## Implementation Order

1. **Fix B3** (strongest directive for target states) — quick test
2. **Fix C** (Portal/Email skip Phase 3b) — prevents hallucination
3. **Fix A** (Split Phase 3) — structural fix for F12/F13/F14
4. **Fixes D-I** (instruction tweaks) — batch after structural fixes

---

## Persistent Issues (Not Fixable via Instructions)

| Issue | Why |
|-------|-----|
| F1: Duplicate Phase 1 prompt | Platform loads PROMPT from both action context and resource |
| F7: Cards instead of plain text | Model behavior, intermittent |
| F8: Enum validation in batch | LLM processes multiple criteria at once (Fix E may help) |
| F9: ActionSet clicks not received | Platform delivery issue |

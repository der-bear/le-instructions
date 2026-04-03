# Fix Plan — HIGH and MEDIUM Severity Issues

**Date:** 2026-04-03
**Scope:** Instruction-level fixes only (excludes platform/model behavior issues)
**Files Affected:** 3 resource files in `/delivery-original-stabilized/resources/`

---

## Issue 1: Content Type Mismatch Detection (HIGH)

**File:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-3-create-delivery-method.md`

**Problem:** 
When user selects "JSON" but pastes XML (or any format mismatch), the LLM silently extracts fields without alerting the user. The lenient validation in Schema Validation (lines 112-122) handles imperfect formatting but NOT mismatched formats.

**Current Text (Lines 100-110):**
```
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
```

**Proposed Replacement (Lines 100-121):**
```
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
  ELSE:
    - Auto-detect actual format from postingInstructions (same detection logic as above): actualFormat
    - IF contentType != actualFormat:
        PROMPT: "You selected {contentType}, but the content looks like {actualFormat} format. Should I proceed with {contentType} or switch to {actualFormat}?"
        SUGGEST [adaptive_card]: ActionSet (Continue with {contentType} | Use {actualFormat} format)
        WAIT for user choice
        IF "Use {actualFormat} format": contentType = actualFormat
```

**Why This Fixes It:**
Adds explicit detection of format mismatch even when user pre-selects a content type. Forces user confirmation before proceeding with mismatched format, preventing silent data loss or parsing errors.

**Risks:**
- Adds one extra prompt in the happy path (user selects correct format) — mitigation: only shows if mismatch detected
- Could slow down fast users who intentionally want format conversion — acceptable trade-off for preventing data loss

---

## Issue 2: Batch Criteria Parsing (HIGH)

**File:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`

**Problem:**
When user provides multiple criteria in one message (e.g., "LoanAmount > 500k, PropertyType = Condo, AnnualIncome > 100k"), the LLM processes all as a batch. Per-criterion enum validation (ChoiceSet display) is skipped during batch processing. Instructions assume one criterion per invocation but don't explicitly forbid batching.

**Current Text (Lines 68-77):**
```
PROCESS (Criteria Loop):
  - PROMPT: "Would you like to add another criterion, see more fields, or continue?"
  - SUGGEST [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
  - WAIT for user input
  - If provides new criterion directly (typed) → parse it via Criteria Parsing, APPEND result to criteriaPayload array, add display entry to criteriaList → LOOP BACK to start of Criteria Loop (ask again)
  - If selects "Show more fields" OR asks to see more AND extraFields available:
      PROMPT: "Additional Fields (showing up to 10):\n• {list extraFields}"
      SUGGEST [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
      LOOP BACK to start of Criteria Loop
  - ONLY IF selects "Continue" OR says "continue"/"done"/"no" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array
```

**Current Text (Lines 79-135) — Criteria Parsing section:**
```
PROCESS (Criteria Parsing):
  - Parse natural language to operator keywords (minimum/at least→GreaterOrEqual, exactly→Equal, etc.)
  - Match field leadFieldName (fuzzy >90%)
  - Lookup matched field metadata from leadFields: leadFieldUID, leadFieldDataType, isEnumerated, leadFieldEnums, leadFieldSpecialBit
  - Extract values (comma/or-separated)
  ...
  [continues with enumeration and value validation]
```

**Proposed Replacement (Lines 68-77):**
```
PROCESS (Criteria Loop):
  - PROMPT: "Would you like to add another criterion, see more fields, or continue?"
  - SUGGEST [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
  - WAIT for user input
  - If provides new criterion directly (typed) → check if multiple criteria present (comma-separated operators or 2+ complete criteria):
      IF multiple criteria detected:
        PROMPT: "I see multiple criteria. Let me process each one separately to ensure accuracy."
        FOR EACH criterion in parsed list:
          - Parse individual criterion via Criteria Parsing (see below)
          - APPEND result to criteriaPayload array
          - Add display entry to criteriaList
        LOOP BACK to start of Criteria Loop (ask again)
      ELSE:
        - Parse single criterion via Criteria Parsing, APPEND result to criteriaPayload array, add display entry to criteriaList → LOOP BACK to start of Criteria Loop (ask again)
  - If selects "Show more fields" OR asks to see more AND extraFields available:
      PROMPT: "Additional Fields (showing up to 10):\n• {list extraFields}"
      SUGGEST [adaptive_card]: ActionSet (Show more fields | Continue) if extraFieldCount > 0, else ActionSet (Continue)
      LOOP BACK to start of Criteria Loop
  - ONLY IF selects "Continue" OR says "continue"/"done"/"no" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array
```

**Why This Fixes It:**
Explicitly detects batch input (multiple criteria in one message) and forces per-criterion processing through the full Criteria Parsing pipeline, including enum validation via ChoiceSet. Ensures each criterion gets validated individually with its appropriate prompt/ChoiceSet if enumerated.

**Risks:**
- Could result in more prompts for batch input — acceptable because it prevents validation bypass
- Requires logic to detect batch input (comma-separated with operators) — mitigation: clear regex pattern for operators (=, >, <, between, in, etc.)

---

## Issue 3: "Between" Operator Value Format (MEDIUM)

**File:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`

**Problem:**
"Between" operator value format is undocumented. State UIDs use pipe-delimited format (stateUID1|stateUID2), suggesting pipes are standard, but this isn't explicitly stated for other operators.

**Current Text (Lines 128-135):**
```
  - Create parsedCriteria object with these fields:
      leadFieldUID: (integer from matched field)
      type: "FieldValue"
      operator: (string value like "In", "Equal", "Between", etc.)
      value: (string - single enumUID for enums, direct value for scalar fields)
  - Create criteriaList entry for display: translate operator to plain English (GreaterOrEqual→"at least")
  - APPEND this parsedCriteria to criteriaPayload array (do NOT overwrite previous entries)
  - Add display entry to criteriaList array for summary display
```

**Proposed Replacement (Lines 128-140):**
```
  - Create parsedCriteria object with these fields:
      leadFieldUID: (integer from matched field)
      type: "FieldValue"
      operator: (string value like "In", "Equal", "Between", etc.)
      value: (string format rules):
        - Single values: "scalar_value" or "enumUID" (no delimiters)
        - Multiple values (In/NotIn/Between): pipe-delimited "value1|value2|value3"
        - State fields: pipe-delimited stateUID format (see Build Criteria Array section)
        - Example: "500000|1000000" for Between operator on numeric field
  - Create criteriaList entry for display: translate operator to plain English (GreaterOrEqual→"at least", Between→"between X and Y")
  - APPEND this parsedCriteria to criteriaPayload array (do NOT overwrite previous entries)
  - Add display entry to criteriaList array for summary display
```

**Why This Fixes It:**
Documents the universal pipe-delimited format for multi-value operators (In, NotIn, Between). Removes ambiguity and ensures consistent serialization across all field types. Clarifies that "Between" requires two pipe-delimited values.

**Risks:**
- Low risk — documentation only, no behavioral change needed
- Helps future debugging by establishing clear format contract

---

## Issue 4: Field-Not-Found Loop Continuation (MEDIUM)

**File:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`

**Problem:**
After "I couldn't find that field" prompt (line 126), there's no explicit LOOP BACK instruction. User can retry or say "show fields" but the return path to Criteria Loop is implicit, which can confuse the LLM about whether to exit or re-enter the loop.

**Current Text (Lines 125-127):**
```
  IF field not matched:
    PROMPT: "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options."

  [section ends, no explicit loop instruction]
```

**Proposed Replacement (Lines 125-136):**
```
  IF field not matched:
    PROMPT: "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options."
    ASK [conversational]: userInput
    WAIT for user input
    IF userInput = "show fields":
      PROMPT: "All Available Fields for {leadTypeName}:\n\n• {list all leadFields with field names and data types}"
      SUGGEST [adaptive_card]: ActionSet (Continue back to criteria entry)
      WAIT for user choice
      PROMPT: "Would you like to add another criterion, see more fields, or continue?"
      LOOP BACK to start of Criteria Loop
    ELSE:
      - Re-attempt field matching with new userInput via Criteria Parsing (recursive call)
      - If STILL no match after 2 attempts, PROMPT: "I couldn't match '{userInput}' to any field. Would you like to skip this criterion?"
      - If user says "skip", LOOP BACK to start of Criteria Loop
      - If user retries, LOOP BACK to field matching step

  [After field matched or user exits field matching]
  - Create parsedCriteria object with these fields:
```

**Why This Fixes It:**
Adds explicit LOOP BACK instruction after field-not-found handling. Clarifies that user can show fields, retry, or skip — and after each action, explicitly returns to Criteria Loop. Prevents LLM from getting stuck in field matching or exiting the loop prematurely.

**Risks:**
- Adds 1-2 extra prompts in the unhappy path (field not found) — acceptable cost for clarity
- Recursive re-attempt of Criteria Parsing could create infinite loop if not bounded — mitigation: max 2 retries before skip option

---

## Issue 5: Criteria Loop Exit Phrases (MEDIUM)

**File:** `/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`

**Problem:**
Only "continue", "done", "no" exit the loop (per current line 77 instruction: `says "continue"/"done"/"no"`). Natural phrases like "skip", "finish", "that's all", "nothing else" do NOT exit — they're treated as field name attempts, frustrating users.

**Current Text (Line 77):**
```
  - ONLY IF selects "Continue" OR says "continue"/"done"/"no" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array
```

**Proposed Replacement (Line 77):**
```
  - ONLY IF selects "Continue" OR says "continue"/"done"/"no"/"skip"/"finish"/"that's all"/"nothing else"/"no more" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array
```

**Also update Criteria Loop prompt (Line 69) for clarity:**

**Current Text (Line 69):**
```
  - PROMPT: "Would you like to add another criterion, see more fields, or continue?"
```

**Proposed Replacement (Line 69):**
```
  - PROMPT: "Would you like to add another criterion, see more fields, or continue? (Type a criterion, select an option, or say 'done'/'skip'/'finish' to stop)"
```

**Why This Fixes It:**
Expands exit phrase detection to match natural user language. Users who say "finish" or "that's all" will now properly exit instead of having their input treated as a field name. The updated prompt also sets user expectations about acceptable exit phrases.

**Risks:**
- Very low — extends existing behavior, no new logic required
- Could accidentally match "finish" or "skip" if user mentions these words in a criterion (e.g., "skip this until I finish thinking") — mitigation: only treat as exit if used at the START or END of message, not embedded in a criterion phrase

---

## Summary Table

| Issue | File | Lines | Type | Severity | Fix Type |
|-------|------|-------|------|----------|----------|
| Content type mismatch | phase-3-create-delivery-method.md | 100-110 → 100-121 | Add format validation | HIGH | Expansion |
| Batch criteria parsing | phase-5-create-delivery-account.md | 68-77 | Split batch into individual criteria | HIGH | Expansion |
| "Between" operator format | phase-5-create-delivery-account.md | 128-135 → 128-140 | Document pipe-delimited format | MEDIUM | Documentation |
| Field-not-found loop | phase-5-create-delivery-account.md | 125-127 → 125-136 | Add explicit loop back | MEDIUM | Expansion |
| Criteria exit phrases | phase-5-create-delivery-account.md | 77, 69 | Expand exit phrase list | MEDIUM | Expansion |

---

## Implementation Order (Recommended)

1. **First:** Issue 5 (Criteria exit phrases) — lowest risk, quick validation
2. **Second:** Issue 3 ("Between" operator) — documentation only, unblocks other fixes
3. **Third:** Issue 4 (Field-not-found loop) — medium complexity, clear scope
4. **Fourth:** Issue 1 (Content type mismatch) — medium complexity, affects Phase 3
5. **Fifth:** Issue 2 (Batch criteria) — highest complexity, requires batch detection logic

**Testing approach:** After each fix, re-run agent trace tests (particularly batching scenario for Issue 2, mismatch scenario for Issue 1, exit phrase variations for Issue 5).

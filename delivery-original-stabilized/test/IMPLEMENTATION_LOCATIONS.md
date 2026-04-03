# Implementation Locations — Exact File Paths and Line Numbers

## Quick Reference

**Total files to modify:** 2
**Total lines to add/change:** ~35 lines

---

## File 1: Phase 3 — Create Delivery Method

**Absolute Path:**
```
/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-3-create-delivery-method.md
```

### Fix #1: Content Type Mismatch Detection (HIGH)

**Section:** PROCESS (Auto-detect Content Type)
**Current lines:** 100–110
**Action:** EXPAND (add 11 lines after line 110)

**Current text (lines 100–110):**
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

**Insert after line 110 (before line 111):**
```
  ELSE:
    - Auto-detect actual format from postingInstructions (same detection logic as above): actualFormat
    - IF contentType != actualFormat:
        PROMPT: "You selected {contentType}, but the content looks like {actualFormat} format. Should I proceed with {contentType} or switch to {actualFormat}?"
        SUGGEST [adaptive_card]: ActionSet (Continue with {contentType} | Use {actualFormat} format)
        WAIT for user choice
        IF "Use {actualFormat} format": contentType = actualFormat
```

**Reason:** Detects format mismatch even when user pre-selects a type, preventing silent data loss from pasting wrong format.

---

## File 2: Phase 5 — Create Delivery Account

**Absolute Path:**
```
/Users/aderkach/Source/le-instructions/delivery-original-stabilized/resources/phase-5-create-delivery-account.md
```

### Fix #2: Batch Criteria Parsing (HIGH)

**Section:** PROCESS (Criteria Loop)
**Current lines:** 68–77
**Action:** EXPAND (add 8 lines within line 72)

**Current text (lines 68–77):**
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

**Replace line 72 with:**
```
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
```

**Reason:** Explicitly detects batch input and forces per-criterion processing, ensuring enum validation doesn't get skipped.

---

### Fix #3: "Between" Operator Value Format (MEDIUM)

**Section:** PROCESS (Criteria Parsing) — Create parsedCriteria object
**Current lines:** 128–135
**Action:** EXPAND (add 4 lines; replace line 132)

**Current text (lines 128–135):**
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

**Replace lines 132 (the "value:" line) with:**
```
      value: (string format rules):
        - Single values: "scalar_value" or "enumUID" (no delimiters)
        - Multiple values (In/NotIn/Between): pipe-delimited "value1|value2|value3"
        - State fields: pipe-delimited stateUID format (see Build Criteria Array section)
        - Example: "500000|1000000" for Between operator on numeric field
```

**Also update line 133 (the "Create criteriaList..." line):**
```
  - Create criteriaList entry for display: translate operator to plain English (GreaterOrEqual→"at least", Between→"between X and Y")
```

**Reason:** Documents pipe-delimited format explicitly for "Between" and other multi-value operators, removing ambiguity.

---

### Fix #4: Field-Not-Found Loop Continuation (MEDIUM)

**Section:** PROCESS (Criteria Parsing) — IF field not matched
**Current lines:** 125–127
**Action:** EXPAND (replace line 126 + add 10 lines after)

**Current text (lines 125–127):**
```
  IF field not matched:
    PROMPT: "I couldn't find that field. Please type the field name you'd like to use, or say 'show fields' to see all available options."

  [continues to Create parsedCriteria...]
```

**Replace with:**
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
```

**Then insert before "Create parsedCriteria object":**
```
  [After field matched or user exits field matching]

```

**Reason:** Adds explicit handling for "show fields" request and loop-back instructions, preventing LLM from getting stuck.

---

### Fix #5: Criteria Loop Exit Phrases (MEDIUM)

**Section:** PROCESS (Criteria Loop) — exit condition
**Current lines:** 69, 77
**Action:** MODIFY (expand 2 lines)

**Current text — Line 69:**
```
  - PROMPT: "Would you like to add another criterion, see more fields, or continue?"
```

**Replace with:**
```
  - PROMPT: "Would you like to add another criterion, see more fields, or continue? (Type a criterion, select an option, or say 'done'/'skip'/'finish' to stop)"
```

**Current text — Line 77:**
```
  - ONLY IF selects "Continue" OR says "continue"/"done"/"no" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array
```

**Replace with:**
```
  - ONLY IF selects "Continue" OR says "continue"/"done"/"no"/"skip"/"finish"/"that's all"/"nothing else"/"no more" → create summary string from criteriaList ("; " separated or "None" if empty) → RETAIN additionalCriteria (string) → GO TO Build Criteria Array
```

**Reason:** Expands exit phrase recognition to match natural user language, preventing input from being treated as field names.

---

## Summary Table

| Fix # | File | Lines | Type | Status |
|-------|------|-------|------|--------|
| 1 | phase-3-create-delivery-method.md | 100–110 (insert after 110) | Expansion | Ready |
| 2 | phase-5-create-delivery-account.md | 72 (replace) | Replacement | Ready |
| 3 | phase-5-create-delivery-account.md | 132–133 (replace) | Replacement | Ready |
| 4 | phase-5-create-delivery-account.md | 125–127 (expand) | Expansion | Ready |
| 5 | phase-5-create-delivery-account.md | 69, 77 (modify) | Modification | Ready |

---

## Testing Checklist

After implementing each fix, verify:

- [ ] Fix #1: Test JSON declared but XML pasted → prompt appears
- [ ] Fix #1: Test XML declared but JSON pasted → prompt appears
- [ ] Fix #2: Test "LoanAmount > 500k, PropertyType = Condo, AnnualIncome > 100k" → processes each separately
- [ ] Fix #2: Verify each criterion gets enum validation (ChoiceSet) if enumerated
- [ ] Fix #3: Verify Between operator serializes correctly (e.g., "500000|1000000")
- [ ] Fix #4: Test "show fields" → displays all fields → loops back correctly
- [ ] Fix #4: Test field not found → retry → loop back without exiting
- [ ] Fix #5: Test "skip" → exits loop
- [ ] Fix #5: Test "finish" → exits loop
- [ ] Fix #5: Test "that's all" → exits loop
- [ ] Fix #5: Test "nothing else" → exits loop

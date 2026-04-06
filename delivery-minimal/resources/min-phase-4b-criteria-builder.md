═══════════════════════════════════════
CURRENT PHASE: Phase 4b — Optional Criteria Builder
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect optional extra criteria before account creation, keep the interaction small, and return compiled criteria to Phase 4 so Phase 4 can create the account.

## IMPORTANT
- Do not call `summarize_history` in this phase.
- `compiledCriteria` must remain a native structured value for the immediate return to Phase 4.
- Deterministic loop contract:
  - suggestions prompt
  - capture exactly one criterion
  - concise success confirmation
  - explicit loop prompt
  - repeat until the user explicitly exits
- Global exit rule at every yield in this phase:
  - `continue` or `done` → keep any completed criteria collected so far, discard any pending in-progress criterion, and exit
  - `continue with geography only` → discard any pending in-progress criterion, clear all compiled criteria, and exit
- Do not mutate the account in this phase.

## DISCOVER
- If `leadFields` are missing, call `get_lead_type(leadTypeUID)` and retain `leadTypeName` and `leadFields`.

## BUILD
- On first entry for the current account flow, if `extraCriteriaStatus` is missing or `addExtraCriteriaChoice!="Yes"`:
  - retain `compiledCriteria=[]`
  - retain `additionalCriteriaSummaryList=[]`
  - clear `pendingEnumFieldName`, `pendingEnumFieldUID`, and `pendingEnumFieldOperator`
  - retain `extraCriteriaStatus="building"`
- Otherwise:
  - keep any already compiled criteria
  - keep any existing `additionalCriteriaSummaryList`
  - clear `pendingEnumFieldName`, `pendingEnumFieldUID`, and `pendingEnumFieldOperator`
  - retain `extraCriteriaStatus="building"`
- Build a short suggestion list:
  - prefer a few high-value enumerated or numeric fields
  - exclude state fields, contact fields, IDs, and tracking fields

## ASK
- Start with a short prompt that includes 3-5 suggested fields and examples.
- Ask for one criterion at a time.
- Accept:
  - a new criterion in plain language
  - `show fields`
  - `continue with geography only`
  - `continue`
  - `done`
- If the user sends multiple criteria in one message, compile only the first safely compilable criterion and tell the user the remaining criteria will be handled one at a time.

## COMPILE
- For each criterion:
  - handle loop-control keywords before field matching:
    - `add another`
    - `show fields`
    - `continue`
    - `done`
    - `continue with geography only`
  - match the field name against `leadFields` using exact match first, then clear fuzzy match only when confidence is above 90%
  - extract scalar values deterministically from the user's text before building the criterion
  - parse the operator deterministically:
    - `at least`, `>=`, `no less than` -> `GreaterOrEqual`
    - `more than`, `greater than`, `>` -> `Greater`
    - `at most`, `<=`, `no more than` -> `LessOrEqual`
    - `less than`, `<` -> `Less`
    - `between X and Y` -> `Between`
    - `not equal`, `is not`, `!=` -> `NotEqual`
    - `contains`, `includes` -> `Contains`
    - `does not contain`, `not contain` -> `DoesNotContain`
    - `one of`, `any of`, `is one of` -> `In`
    - `not in`, `except`, `exclude` -> `NotIn`
    - exact/equality phrasing -> `Equal`
    - if the field and a concrete scalar value are clear but no explicit operator phrase appears, use `Equal`
  - validate the parsed operator in this priority order before building the criterion:
    - special flags override `leadFieldDataType`
    - enumerated or state fields -> only `In` or `NotIn`
    - `Zip` -> `Equal`, `NotEqual`, `In`, `NotIn`, `Distance_Compare`
    - `PrimaryPhone` or `MobilePhone` -> `Equal`, `NotEqual`, `Greater`, `Less`, `GreaterOrEqual`, `LessOrEqual`, `Between`, `In`, `NotIn`
    - numeric types `Int`, `BigInt`, `Decimal`, `Float`, `Money` -> `Equal`, `NotEqual`, `Greater`, `Less`, `GreaterOrEqual`, `LessOrEqual`, `Between`, `In`, `NotIn`
    - `DateTime` -> `Equal`, `NotEqual`, `Greater`, `Less`, `GreaterOrEqual`, `LessOrEqual`, `Between`, `DateCompare`
    - `Varchar` -> `Equal`, `NotEqual`, `Contains`, `DoesNotContain`, `In`, `NotIn`
    - `Bit` -> `Equal`, `NotEqual`
  - if the parsed operator is invalid for the matched field:
    - for enumerated fields, use `NotIn` only when the user's wording clearly indicates exclusion; otherwise use `In`
    - for other fields, replace it with the first valid operator from the applicable list
  - each compiled criterion must use the native shape `{leadFieldUID:<int>, type:"FieldValue", operator:<string>, value:<string>}`
  - if a matched non-enumerated field does not have a concrete scalar value after parsing, do not compile it and ask for exactly one corrected criterion
  - for enumerated fields, use only `In` or `NotIn` and require a concrete value
  - if the user gives only the field name for an enumerated field, show a compact ChoiceSet plus Submit with no extra submit data
  - retain `pendingEnumFieldName`, `pendingEnumFieldUID`, and `pendingEnumFieldOperator` only for the active enum prompt
  - if the user provides an enum label or value in text, fuzzy-match it case-insensitively to exactly one enum only when confidence is above 85%, then store the selected `leadFieldEnumUID` as the `value` string
  - if an enum value is pending and the user types `continue`, `done`, or `continue with geography only`, honor that exit and discard the pending enum state instead of re-asking
  - enumerated criteria store the selected `leadFieldEnumUID` as the `value` string
  - `Between` serializes `value` as `min|max`
  - build one criterion object and append it to `compiledCriteria`
  - append a human-readable line to `additionalCriteriaSummaryList`
  - after each successful criterion, show one concise success line and then ask exactly:
    - `Add another`
    - `Show fields`
    - `Continue`
- If the field is ambiguous or invalid, offer only:
  - `Fix criteria`
  - `Continue with geography only`
- If the user chooses `Fix criteria`, clear only the pending enum state, keep any compiled criteria already collected, and ask for exactly one corrected criterion.
- If the user asks for more fields, show a short additional list and ask again.
- If the field is still ambiguous or invalid after one focused retry, do not guess.

## EXIT
- If the user chooses `continue` or `done`:
  - retain `compiledCriteria`
  - retain `additionalCriteriaSummary` as a semicolon-separated summary, or `None` if empty
  - retain `extraCriteriaStatus="done"`
  - clear `pendingEnumFieldName`, `pendingEnumFieldUID`, and `pendingEnumFieldOperator`
- If the user chooses `Add another`, ask for exactly one more criterion and wait.
- If the user chooses `Show fields`, show a short additional list and then return to asking for exactly one criterion.
- If the user chooses `continue with geography only`:
  - retain `compiledCriteria=[]`
  - retain `additionalCriteriaSummaryList=[]`
  - retain `additionalCriteriaSummary="None"`
  - retain `extraCriteriaStatus="done"`
  - retain `addExtraCriteriaChoice="No"`
  - clear `pendingEnumFieldName`, `pendingEnumFieldUID`, and `pendingEnumFieldOperator`

## FAILURE
- If a criterion still cannot be compiled safely after one focused retry, offer only:
  - `Fix criteria`
  - `Continue with geography only`

## RETURN
Return to Phase 4 by loading and executing mcp://resource/min-phase-4-create-delivery-account.

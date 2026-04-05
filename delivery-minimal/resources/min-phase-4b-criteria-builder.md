═══════════════════════════════════════
CURRENT PHASE: Phase 4b — Optional Criteria Builder
All prior phase summaries are completed history.
Execute ONLY the instructions below.
═══════════════════════════════════════

## GOAL
Collect optional extra criteria before account creation, keep the interaction small, and return compiled criteria back to Phase 4 without mutating the account.

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
  - `continue` → keep any completed criteria collected so far, discard any pending in-progress criterion, and exit
  - `continue with geography only` → discard any pending in-progress criterion, clear all compiled criteria, and exit
- Do not mutate the account in this phase.

## DISCOVER
- If `leadFields` are missing, call `get_lead_type(leadTypeUID)` and retain `leadTypeName` and `leadFields`.

## PROCESS
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
- If the user sends multiple criteria in one message, compile only the first safely compilable criterion and tell the user the remaining criteria will be handled one at a time.

## PROCESS
- For each criterion:
  - match the field name against `leadFields`
  - infer the safest valid operator
  - each compiled criterion must use the native shape `{leadFieldUID:<int>, type:"FieldValue", operator:<string>, value:<string>}`
  - for enumerated fields, require a concrete value
  - if the user gives only the field name for an enumerated field, show a compact ChoiceSet plus Submit with no extra submit data
  - retain `pendingEnumFieldName`, `pendingEnumFieldUID`, and `pendingEnumFieldOperator` only for the active enum prompt
  - if an enum value is pending and the user types `continue` or `continue with geography only`, honor that exit and discard the pending enum state instead of re-asking
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
- If the user asks for more fields, show a short additional list and ask again.
- If the field is still ambiguous or invalid after one focused retry, do not guess.

## EXIT
- If the user chooses `continue`:
  - retain `compiledCriteria`
  - retain `additionalCriteriaSummary` as a semicolon-separated summary, or `None` if empty
  - retain `extraCriteriaStatus="done"`
- If the user chooses `Add another`, ask for exactly one more criterion and wait.
- If the user chooses `Show fields`, show a short additional list and then return to asking for exactly one criterion.
- If the user chooses `continue with geography only`:
  - retain `compiledCriteria=[]`
  - retain `additionalCriteriaSummary="None"`
  - retain `extraCriteriaStatus="done"`
  - retain `addExtraCriteriaChoice="No"`

## FAILURE
- If a criterion still cannot be compiled safely after one focused retry, offer only:
  - `Fix criteria`
  - `Continue with geography only`

## NEXT
Load and execute mcp://resource/min-phase-4-create-delivery-account.

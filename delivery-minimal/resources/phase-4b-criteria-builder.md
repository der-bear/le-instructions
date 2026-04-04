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

## PROCESS
- For each criterion:
  - match the field name against `leadFields`
  - infer the safest valid operator
  - for enumerated fields, require a concrete value
  - build one criterion object and append it to `compiledCriteria`
  - append a human-readable line to `additionalCriteriaSummaryList`
- If the field is ambiguous or invalid, offer only:
  - `Fix criteria`
  - `Continue with geography only`
- If the user asks for more fields, show a short additional list and ask again.

## EXIT
- If the user chooses `continue`:
  - retain `compiledCriteria`
  - retain `additionalCriteriaSummary` as a semicolon-separated summary, or `None` if empty
  - retain `extraCriteriaStatus="done"`
- If the user chooses `continue with geography only`:
  - retain `compiledCriteria=[]`
  - retain `additionalCriteriaSummary="None"`
  - retain `extraCriteriaStatus="done"`
  - retain `addExtraCriteriaChoice="No"`

## FAILURE
- Do not mutate the account in this phase.
- If a criterion still cannot be compiled safely after one focused retry, do not guess.

## NEXT
Load and execute mcp://resource/min-phase-4-create-delivery-account.

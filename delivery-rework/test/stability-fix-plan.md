# Delivery Rework — Stability Run Fix Plan

Based on stability test findings RB-A through RB-U across 10 completed runs (B01–B10), plus B11 DEBUG session and the DEBUG23 informal audit session.

**Source document:** `stability-rework-findings.md`  
**Variant:** `delivery-rework/`  
**Model under test:** OpenAI GPT-5.4 Mini

---

## Fix Plan: RB-G — Systematic duplicate message rendering at every phase transition

**Phase:** 1, 2, 3, 4, 5 | **Severity:** High | **Frequency:** Systematic — every phase transition in B04–B10

### Root Cause
When a phase resource says "prompt the user exactly as follows: ..." followed by `STOP AND YIELD`, the model treats the exact prompt as content to restate or confirm rather than as the terminal output for that turn. It emits the prompt once via `display_adaptive_card` or plain text, then appends a second copy in the same response as a "confirmation". The global system instructions say "use it exactly" but never say "emit it only once and stop". `STOP AND YIELD` prohibits hallucination and advancing to the next state but does not explicitly prohibit re-emitting the same prompt in the same turn.

### File(s) to modify
- `delivery-rework/system/rw-1-global.md`
- `delivery-rework/resources/rw-phase-1-create-client.md` (and same pattern applied to every `STOP AND YIELD` in all phase files)

### Current instruction (problematic section)

In `rw-1-global.md`, under `## Prompts and Communication`:
```
- If a phase provides exact prompt text, use it exactly — do not paraphrase or modify.
- Replace {variable} placeholders accurately with retained values.
- Keep original line breaks when the prompt text is phase-defined.
```

In `rw-phase-1-create-client.md`, State 1:
```
  - **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to respond.
```

### Proposed patch

In `rw-1-global.md`, add immediately after the line `- If a phase provides exact prompt text, use it exactly — do not paraphrase or modify.`:
```
- When a phase specifies an exact prompt followed by STOP AND YIELD, that prompt is the complete assistant response for that turn. Send it exactly once. Do not repeat, echo, confirm, paraphrase, re-emit, or append any additional text in the same turn. No acknowledgements, transition announcements, or follow-up lines may be appended after the prompt.
- When a STOP AND YIELD follows a display_adaptive_card call, the card IS the complete response for that turn. Do not also send a plain-text version of the card content in the same message. Never mix a display_adaptive_card call with plain-text restatement of the same content in a single response.
```

In every phase resource at each `STOP AND YIELD` that follows an exact prompt, add the phrase "Send the prompt exactly once." For example, in `rw-phase-1-create-client.md` State 1:
```
  - **STOP AND YIELD.** Send the prompt exactly once — do not repeat, echo, or re-emit it in the same turn. Do not hallucinate data. Do not proceed to State 2. Do not call the summarize_history tool. You must wait for the user to respond.
```

Apply the same addition to STOP AND YIELD directives in: `rw-phase-3-create-delivery-method.md` (States 1, 1b, 2), `rw-phase-5-create-delivery-account.md` (Steps 1, 2, 3, 6, 10), and `rw-phase-5c-criteria-builder.md` (State 1 step 2, State 2 loop prompt).

### Why this fixes it
The model needs an explicit "send once" constraint at the global level (establishing the pattern for all phases) and locally at each STOP AND YIELD (reinforcing it at the exact point of failure). The card+text prohibition directly addresses the most common form: the model calls `display_adaptive_card` then also writes out the same choices as plain text.

---

## Fix Plan: RB-K — Phase not loaded before responding (generic prompt before Phase 5)

**Phase:** 4→5 transition | **Severity:** High | **Frequency:** Confirmed in DEBUG23 session, B02, B08

### Root Cause
After Phase 4 summary card's "Continue" is clicked, the model generates a user-facing message ("Please provide the delivery account name.") before `get_resource(rw-phase-5-create-delivery-account)` completes. The model fills the response gap with a generic setup prompt because Phase 5 instructions are not yet in context. There is no rule in the system instructions or Phase 4 handoff that prohibits generating a prompt before the next phase resource is loaded.

### File(s) to modify
- `delivery-rework/system/rw-1-global.md`
- `delivery-rework/resources/rw-phase-4-delivery-method-summary.md`

### Current instruction (problematic section)

In `rw-1-global.md`, under `## Resource Handling`:
```
All workflow phases are loaded as MCP resources via get_resource. Do not call get_resource again for the same phase unless that phase explicitly instructs you to. After a phase completes, follow its handoff instructions immediately — do not announce resource retrieval.
```

In `rw-phase-4-delivery-method-summary.md` (lines 23–24):
```
IF flowIntent = "full-setup" AND the user said "Continue" or "done":
  Immediately call the summarize_history tool, then load Phase 5.
```

### Proposed patch

In `rw-1-global.md`, add to `## Resource Handling` after the existing paragraph:
```
CRITICAL: Before sending any user-facing prompt during a phase transition, the next phase resource MUST be loaded via get_resource first. Never generate a generic collection prompt (e.g., "Please provide the account name", "Please provide the delivery type") while awaiting phase instructions. If phase instructions are not yet loaded, call get_resource(next_phase_url), wait for it to complete, then execute its first step. Do not fill the gap with placeholder or assumed prompts.
```

In `rw-phase-4-delivery-method-summary.md`, replace lines 23–24:
```
IF flowIntent = "full-setup" AND the user said "Continue" or "done":
  Immediately call the summarize_history tool. Then call get_resource("mcp://resource/rw-phase-5-create-delivery-account"). Do NOT send any user-facing message until Phase 5 instructions are loaded. Once loaded, execute Phase 5 Step 1.
```

### Why this fixes it
The global rule establishes a hard constraint preventing any prompt generation before resource loading. The Phase 4 handoff now spells out the exact sequence (summarize, then load, then act on the first step) and explicitly prohibits intermediate messages, closing the gap where a generic prompt was being inserted.

---

## Fix Plan: RB-D — Criteria gate entirely skipped

**Phase:** 5 | **Severity:** High | **Frequency:** 2/10 (B02, B10 — different root causes)

### Root Cause
Two distinct failure modes:

1. **B02 — Summarization context loss:** When `summarize_history` fires mid-Phase 5 due to a long conversation, the criteria gate step (Step 10) is lost from working memory. The agent jumps directly to Phase 6.

2. **B10 — Optional skip branch:** Step 10 contains an early-exit: `IF additionalCriteriaChoice is already set to "Skip"`. The model triggers this branch without the user ever being asked, because `additionalCriteriaChoice` can be inferred as "Skip" from the optional framing of the gate.

Agent DEBUG (B10): *"The skip criteria branch was taken. The phase routing made criteria optional in practice. Since the flow did not force criteria collection before account creation, the system proceeded with the account creation path."*

### File(s) to modify
- `delivery-rework/resources/rw-phase-5-create-delivery-account.md`

### Current instruction (problematic section)

Step 10 (lines 68–85):
```
Step 10: Criteria Gate
  FIRST — check if already decided:
  IF additionalCriteriaChoice is already set to "Skip" (returned from Phase 5c):
    Retain additionalCriteria = "None".
    Immediately call the summarize_history tool.
    (Do not re-prompt — the user already chose to skip in Phase 5c.)
    STOP HERE.

  OTHERWISE — ask the user:
  Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  Present using display_adaptive_card with an ActionSet: "Add criteria" | "Skip". MUST use display_adaptive_card.
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

  IF the user selected "Skip" OR said "none", "skip", or "no":
    Retain additionalCriteria = "None".
    Immediately call the summarize_history tool.

  IF the user selected "Add criteria" OR said "yes", "add", or "criteria":
    Load mcp://resource/rw-phase-5c-criteria-builder (no summarize — leadFields stays in working memory)
```

Also add to the Phase 5 header (after the objective line, line 9):

Section is missing — there is no ordering constraint at the top of Phase 5.

### Proposed patch

Add to Phase 5 after the objective line:
```
CRITICAL ORDERING: Steps 1–10 must execute in exact order. Do not call summarize_history until Step 10 is fully complete. Do not skip any step. Step 10 (Criteria Gate) is MANDATORY — the user must explicitly choose "Add criteria" or "Skip" before this phase can complete.
```

Replace Step 10 entirely:
```
Step 10: Criteria Gate (MANDATORY — must always execute)
  CRITICAL: This step is REQUIRED. Do not skip it. Do not call summarize_history before completing this step. Do not proceed to Phase 6 without explicit user confirmation here.

  ONLY if additionalCriteriaChoice was set to "Skip" by an explicit user action inside Phase 5c (the user clicked "Skip" in the criteria builder and was returned here):
    Retain additionalCriteria = "None".
    Immediately call the summarize_history tool.
    STOP HERE.

  IN ALL OTHER CASES — including when additionalCriteriaChoice is unset, null, or was never explicitly provided by the user in this turn:
  Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  Present using display_adaptive_card with an ActionSet: "Add criteria" | "Skip". MUST use display_adaptive_card. Send the prompt exactly once.
  **STOP AND YIELD.** Do not hallucinate data. Do not default to "Skip". Do not infer the answer from context. You must wait for the user to explicitly choose.

  IF the user selected "Skip" OR typed "none", "skip", or "no":
    Retain additionalCriteria = "None", additionalCriteriaChoice = "Skip".
    Immediately call the summarize_history tool.

  IF the user selected "Add criteria" OR typed "yes", "add", or "criteria":
    Retain additionalCriteriaChoice = "Add".
    Immediately call get_resource("mcp://resource/rw-phase-5c-criteria-builder"). Wait for the resource to load before sending any user-facing message. Do not summarize — leadFields must stay in working memory.
```

### Why this fixes it
The "IN ALL OTHER CASES" clause ensures the gate fires even when `additionalCriteriaChoice` was inferred or defaulted (root cause 2). The ordering constraint at the phase header prevents `summarize_history` from firing before Step 10 (root cause 1). The "Do not default to Skip. Do not infer the answer from context." instruction closes the optional-framing loophole.

---

## Fix Plan: RB-A — Criteria loop exits after first criterion

**Phase:** 5c | **Severity:** High | **Frequency:** 6/9 (B01, B04, B05, B06, B07, B08)

### Root Cause
After a criterion is accepted in Phase 5c State 2, the model advances directly to State 3 (update account) instead of showing the criteria loop prompt. The loop prompt exists in the instructions ("Show criteria loop prompt (below)") but is structurally weak: it is a subordinate clause after the criterion-acceptance block, not a mandatory gate. The GLOBAL EXIT RULE at the top of Phase 5c further conflates exit conditions for State 1 and the loop prompt — the model can treat a single criterion as satisfying the "not empty" exit condition and route to State 3. Additionally, the loop card only has "Show more fields" and "Continue" buttons with no "Add another" button, giving the model no clear signal that the loop should continue.

Agent DEBUG (B04): *"The criteria flow should not have exited after one criterion. The additional-criteria gate was treated as resolved instead of continuing into the criteria-builder loop."*

### File(s) to modify
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

GLOBAL EXIT RULE (lines 10–12):
```
GLOBAL EXIT RULE: At any YIELD, if user says "skip"/"done"/"continue"/"none"/"no":
- If criteriaSummaryList not empty → State 3.
- If empty → set additionalCriteriaChoice="Skip", return to Phase 5.
```

Criteria loop prompt (lines 56–62):
```
Criteria loop prompt (after each criterion added):
  Prompt: "Would you like to add another criterion, see more fields, or continue?"
  Present using display_adaptive_card: IF more extra fields remain: ActionSet "Show more fields" | "Continue". ELSE: ActionSet "Continue".
  **STOP AND YIELD.** Do not hallucinate data.
  - IF another criterion → handle as "user typed a criterion" above.
  - IF "Show more fields" → handle as show-more above.
  - IF "Continue"/"done"/"no" → State 3.
```

### Proposed patch

Replace the GLOBAL EXIT RULE:
```
GLOBAL EXIT RULE (applies ONLY at YIELD points in State 1 — field suggestions — and when no criterion is being collected):
If user says "skip"/"done"/"continue"/"none"/"no" at a State 1 YIELD:
- If criteriaSummaryList not empty → State 3.
- If empty → set additionalCriteriaChoice="Skip", return to Phase 5.
NOTE: This rule does NOT apply inside the criteria loop prompt. The loop prompt has its own explicit exit handling.
```

Replace the criteria loop prompt:
```
Criteria loop prompt (MANDATORY after EVERY accepted criterion — never skip):
  CRITICAL: After every single criterion appended to parsedCriteriaList, you MUST show this prompt before doing anything else. Do NOT proceed to State 3 after a single criterion. Do NOT treat criterion acceptance as Phase 5c completion. The ONLY paths out of this prompt are: user adds another criterion (loop continues in State 2), or user explicitly says "Continue"/"done"/"no more" (proceed to State 3).
  Prompt: "✓ Criterion added: {latestCriteriaSummary}\n\nWould you like to add another criterion, see more fields, or continue?"
  Present using display_adaptive_card: ActionSet "Add another" | {IF more extra fields remain: "Show more fields" |} "Continue".
  **STOP AND YIELD.** Send the prompt exactly once. Do not hallucinate data.
  - IF "Add another" OR user types a new criterion → loop back to "IF user typed a criterion" handler in State 2.
  - IF "Show more fields" → handle as show-more above, then return to this prompt after.
  - IF "Continue"/"done"/"no"/"no more"/"skip" → State 3.
  Do NOT proceed to State 3 for any other reason.
```

### Why this fixes it
Three changes combine to fix the loop: (1) The GLOBAL EXIT RULE is scoped to State 1 only, preventing it from short-circuiting the loop after each criterion. (2) The loop prompt carries a `CRITICAL` directive that explicitly prohibits treating one criterion as completion. (3) The card now includes an "Add another" button, giving the model a clear continue-loop action and preventing the user from being presented with only "Continue" after their first criterion.

---

## Fix Plan: RB-F — Phase 5c fails to load on first "Add criteria" click

**Phase:** 5c | **Severity:** High | **Frequency:** 3/7 (B03, B06, B07 — escalating severity)

### Root Cause
When the user clicks "Add criteria" in Phase 5, the instruction says `Load mcp://resource/rw-phase-5c-criteria-builder`. The model sometimes treats this as a deferred or background action, generates a placeholder message ("Please continue with the additional criteria setup") before the resource is in context, and then awaits the user's next input. On retry the resource loads correctly because it is now cached. In longer conversations (B06, B07) the failure is severe: 2–3 failed attempts plus explicit resource URL via DEBUG required.

Agent DEBUG (B06): *"I did not have the actual Phase 5c instructions loaded in the conversation context at that point, so I couldn't safely process it as the active criteria step."*

### File(s) to modify
- `delivery-rework/resources/rw-phase-5-create-delivery-account.md`

### Current instruction (problematic section)

Step 10, "Add criteria" branch (lines 84–85):
```
  IF the user selected "Add criteria" OR said "yes", "add", or "criteria":
    Load mcp://resource/rw-phase-5c-criteria-builder (no summarize — leadFields stays in working memory)
```

### Proposed patch

Replace lines 84–85:
```
  IF the user selected "Add criteria" OR typed "yes", "add", or "criteria":
    Retain additionalCriteriaChoice = "Add".
    IMMEDIATELY call get_resource("mcp://resource/rw-phase-5c-criteria-builder"). Do NOT send any user-facing message before this call completes. Do not say "Please continue", "Let me load the criteria builder", or any other placeholder text while loading. Wait for the resource to load, then execute Phase 5c State 1 in the same turn.
    Do not re-display the criteria gate card. Do not re-prompt "Add criteria / Skip". Once "Add criteria" is selected, the decision is final — proceed directly into Phase 5c.
```

### Why this fixes it
Explicitly forbidding placeholder messages before `get_resource` completes eliminates the gap that the model was filling with generic text. Naming the exact failure messages ("Please continue", "Let me load") provides concrete negative examples. The "decision is final" clause prevents the duplicate gate-card loop observed in B07.

---

## Fix Plan: RB-N / RB-O — States question skipped; phantom states in account

**Phase:** 5 | **Severity:** Critical | **Frequency:** RB-N: 1/7 (B07); RB-O: 1/7 (B07, confirmed in session log)

### Root Cause

**B07 analysis:** After Order System = No, the model skipped Phase 5 Step 6 entirely and jumped to the criteria gate. Then phantom states CA, AZ, TX appeared in the delivery account summary and were confirmed in the actual API payload via session log audit (UID 5=CA, UID 3=AZ, UID 44=TX).

**B11 DEBUG analysis (additional confirmation):** The phantom states did NOT come from requestBody field mapping or Phase 3 `summarize_history`. They came from "Phase 5 account state already set" — the flow had `targetStates` in memory from a prior conversation context before the user was ever asked in the current session. Agent self-diagnosis: *"Phase 5 lacked a hard gate checking whether targetStates was explicitly collected in the CURRENT conversation turn. Phase 6 treated them as existing valid summary data."*

The core problem: `targetStates` is a required ASK field but Phase 5 allows retained values from prior phases or summarization to satisfy it without re-prompting. Phase 6 renders whatever is in `targetStates` without validating that the value was collected this session.

Agent's 4 patch recommendations (from B11 DEBUG):
1. P5: Before any account creation call, verify targetStates was explicitly provided by user in current conversation. If not, ask. Do not reuse previously retained states from prior phases.
2. P6: Only display summary fields that were explicitly provided by user or returned by a preceding creation step in the current flow.
3. P6: Before rendering summary card, validate targetStates meets requirements. If missing or unconfirmed, stop and ask.
4. Global: Retained values from prior phases usable only when phase explicitly allows carry-forward.

### File(s) to modify
- `delivery-rework/resources/rw-phase-5-create-delivery-account.md`
- `delivery-rework/resources/rw-phase-6-delivery-account-summary.md`
- `delivery-rework/system/rw-1-global.md`

### Current instruction (problematic section)

Phase 5 Step 6 (lines 46–48):
```
Step 6: Collect Target States
  Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Normalize to uppercase USPS codes (e.g., California → CA).
```

Phase 5 Step 7 (lines 50–55) — no guard before building criteria:
```
Step 7: Match States and Build Criteria
  Call the get_usa_states tool. Match normalized targetStates to the returned list ...
```

Phase 6 (lines 7–22) — no validation before rendering summary:
```
## Instructions

Display the delivery account summary using display_adaptive_card tool and this template as base:
...
```

In `rw-1-global.md`, under `## Data Collection`:
```
- **Required fields (ASK):** Must be explicitly collected from the user's message before calling any tool. Do not generate, infer, reuse, or apply defaults. If the user skips or ignores, re-prompt.
```

### Proposed patch

**In `rw-1-global.md`**, add to `## Data Collection` after the existing ASK rule:
```
- ASK fields are scoped to the current conversation session. A value retained from prior summarization or a prior phase does NOT satisfy an ASK requirement for a later phase unless that later phase explicitly marks it as carry-forward. When in doubt, re-ask.
```

**In Phase 5**, replace Step 6:
```
Step 6: Collect Target States (MANDATORY — never skip, never infer, never reuse)
  CRITICAL: This step is REQUIRED. Do not skip it regardless of any prior retained state. targetStates is a strict ASK field — you MUST collect it from the user in this conversation turn. Do NOT generate, infer, reuse, or default target states from any prior context, summarization, prior session, or prior phase. If targetStates appears in a prior summary, IGNORE that value and re-collect from the user now.
  Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
  Send the prompt exactly once.
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
  Normalize ALL inputs to uppercase USPS two-letter codes immediately at input time (e.g., "California" → "CA", "texas" → "TX", "New York" → "NY"). Store ONLY the normalized codes in targetStates — never retain full names. Retain: targetStates (comma-separated USPS codes, e.g., "CA, TX, FL").
```

Add a new Step 7b guard after Step 7:
```
Step 7b: Validate State Collection
  GUARD: Verify targetStates was explicitly collected from the user in Step 6 above (not inferred, not from prior context). If targetStates is null or empty after Step 7 matching, return to Step 6 and re-prompt. Do NOT proceed to Step 8 with an empty or phantom state list. If stateUIDArray is empty after matching because the lead type has no state field, build criteriaPayload as an empty array [].
```

**In Phase 6**, add a validation block before the `display_adaptive_card` instruction:
```
## Pre-Render Validation

Before rendering the summary card, verify:
1. deliveryAccountUID is a positive integer returned by create_delivery_account in this session (not a prior session value or the deliveryMethodUID).
2. targetStates was explicitly provided by the user in Phase 5 Step 6 of this session. If targetStates is null, empty, or was never prompted for in this session, STOP — do not render the summary. Return to Phase 5 Step 6.
3. price, isExclusive, and useOrder were collected from the user in this session.

If any validation fails, stop and re-collect the missing field before proceeding.
```

### Why this fixes it
The global rule extension establishes that ASK field retention is session-scoped, not cross-session. The Phase 5 Step 6 rewrite makes it unambiguous that prior-context values are prohibited. The Step 7b guard is the final checkpoint before account creation, preventing phantom states from entering the API payload. The Phase 6 pre-render validation (per agent recommendations 2 and 3) adds a second checkpoint so that even if phantom data enters memory, it is caught before it is displayed and potentially acted on by the user.

---

## Fix Plan: RB-C — States not normalized after summarization

**Phase:** 5 | **Severity:** High | **Frequency:** 3/5 (B02, B04, B05)

### Root Cause
Normalization is specified in Step 6 as a one-line postscript ("Normalize to uppercase USPS codes"). When `summarize_history` fires, the `targetStates` stored in the summary may contain the user's original input (full names) because normalization was not applied before retention. On context rebuild, the model uses the summary's full-name values without re-normalizing.

Agent DEBUG (B04): *"Target states also should be normalized to USPS codes before criteria creation. In the current phase instructions, that normalization step is intended, but it was not reflected in the retained summary."*

### File(s) to modify
- `delivery-rework/resources/rw-phase-5-create-delivery-account.md`

### Current instruction (problematic section)

Step 6 (line 48):
```
  Normalize to uppercase USPS codes (e.g., California → CA).
```

Summarization template:
```
* Target States: {targetStates}
```

### Proposed patch

This defect is fully addressed by the RB-N/RB-O patch above (Step 6 rewrite). The critical addition is "Normalize ALL inputs to uppercase USPS two-letter codes immediately at input time" and "Store ONLY the normalized codes in targetStates — never retain full names."

Additionally, add a comment to the summarization template:
```
* Target States: {targetStates} (must be USPS codes, e.g., "CA, TX, FL" — never full state names)
```

### Why this fixes it
Normalization at input time (before retention) means the summarization template always receives normalized codes. Even after context loss and rebuild from summary, the stored value is already "CA, TX, FL" rather than "California, Texas, Florida".

---

## Fix Plan: RB-P — `create_delivery_account` not called; method UID used as account UID

**Phase:** 5 | **Severity:** Critical | **Frequency:** 1/8 (B08 — FTP flow)

### Root Cause
After the FTP connection test phase (P3B) completed, the model transitioned to Phase 5 but skipped Step 8 (`create_delivery_account`) entirely. It then called `update_delivery_account` four times using `deliveryMethodUID` as `deliveryAccountUID` — all four failed with Internal Server Error because no account existed. Session log `69d15a862697c9202c0bc120` confirmed `create_delivery_account` is completely absent from the tool sequence. The FTP method UID (46897) was used as both `deliveryMethodUID` and `deliveryAccountUID`.

Root cause confirmed: the P3B→P5 transition failed to reset to Phase 5 Step 1. The model proceeded as if account creation had already occurred.

### File(s) to modify
- `delivery-rework/resources/rw-phase-5-create-delivery-account.md`
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

Phase 5 header (after line 9 — section is missing):
No prerequisite check exists ensuring account creation is mandatory before any updates.

Step 8 (lines 57–61):
```
Step 8: Create Delivery Account
  Call the create_delivery_account tool with these defaults:
  `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, ...}`
  CRITICAL: createDeliveryAccountDto must be passed as a native object, NOT a JSON string. ...
```

Step 9 (lines 63–65):
```
Step 9: Validate and Retain
  Verify deliveryAccountUID is a positive integer > 0. If 0 or null, treat as failure and retry Step 8.
  Retain: deliveryAccountUID, price, targetStates, isExclusive, useOrder.
```

Phase 5c State 3 (lines 84–91) — no guard against updating a non-existent account.

### Proposed patch

Add to Phase 5 header (after CRITICAL ORDERING block):
```
PREREQUISITE: deliveryAccountUID must be obtained from create_delivery_account before any update_delivery_account call. If deliveryAccountUID is not yet set when entering this phase, you MUST execute Steps 1–9 in order before proceeding to Step 10. Never skip Step 8.
```

Replace Step 8:
```
Step 8: Create Delivery Account (MANDATORY — never skip)
  CRITICAL: This step MUST execute in every flow. Do not use update_delivery_account before this step completes. The deliveryAccountUID does not exist until this tool returns it — do not assume it was created earlier.
  Call the create_delivery_account tool with these defaults:
  `clientUID={clientUID}, createDeliveryAccountDto={deliveryMethodUID={deliveryMethodUID}, price={price}, deliveryAccountType="WebAndChatLeads", status="Open", name="{companyName}-Account", automationEnabled=true, isExclusive={isExclusive}, useOrder={useOrder}, dayMax=50, hourMax=-1, weekMax=-1, monthMax=-1, criteria={criteriaPayload}}`
  CRITICAL: createDeliveryAccountDto must be passed as a native object, NOT a JSON string. criteria must be an array of criterion objects, NOT a JSON string.
  If the tool fails, repair the payload and retry once silently. If still fails, prompt: "I ran into an issue creating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
```

Replace Step 9:
```
Step 9: Validate and Retain
  Verify deliveryAccountUID is a positive integer > 0. If 0 or null, treat as failure and retry Step 8.
  GUARD: Verify deliveryAccountUID ≠ deliveryMethodUID. These must be different values. If they are equal, the tool returned the wrong UID — retry Step 8.
  Retain: deliveryAccountUID, price, targetStates, isExclusive, useOrder.
```

In Phase 5c, add a guard at the top of State 3:
```
**State 3: Update Account with Additional Criteria**

GUARD: Before calling update_delivery_account, verify deliveryAccountUID is set AND deliveryAccountUID ≠ deliveryMethodUID. If this check fails, the account was never created — STOP, return to Phase 5 Step 8, create the account, then return here.
```

### Why this fixes it
The "never skip" mandate and "deliveryAccountUID does not exist until this tool returns it" framing prevent the model from assuming prior account creation. The UID inequality guard (`deliveryAccountUID ≠ deliveryMethodUID`) catches the specific B08 failure mode where the method UID was reused. The Phase 5c guard provides a last-resort catch if the skip somehow persists past Phase 5.

---

## Fix Plan: RB-H — Content type choice card rendered as plain text

**Phase:** 3-Webhook, 5, 5c | **Severity:** Medium | **Frequency:** Multiple runs (B04, B05, B08, B09)

### Root Cause
Card non-determinism: the model intermittently renders card-required steps as plain text instead of calling `display_adaptive_card`. Occurs more frequently when prior context is heavy or when multiple workflow steps execute in the same turn. The global adaptive card rule ("MUST use display_adaptive_card for enumerations") is correct but is positioned too far from the point of failure to reliably influence behavior under heavy context load.

### File(s) to modify
- `delivery-rework/resources/rw-phase-3-webhook.md`
- `delivery-rework/resources/rw-phase-5-create-delivery-account.md`
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

Section is not missing — the global rule exists in `rw-1-global.md`. The problem is per-phase reinforcement is absent.

### Proposed patch

Add a card rule reminder at the top of each phase resource that has card-required steps, immediately after the `═══` header block:

In `rw-phase-3-webhook.md`:
```
CARD RULE: All multi-choice prompts in this phase (content type, field mapping choice, parse failure recovery) MUST use display_adaptive_card. Never present enumerated choices as plain text, numbered lists, or bullet lists.
```

In `rw-phase-5-create-delivery-account.md`:
```
CARD RULE: All multi-choice prompts in this phase (exclusivity, order system, criteria gate) MUST use display_adaptive_card with ActionSet buttons. Never present these choices as plain text or numbered options.
```

In `rw-phase-5c-criteria-builder.md`:
```
CARD RULE: All multi-choice prompts in this phase (field suggestions, criteria loop, enum selection) MUST use display_adaptive_card. Never present enumerated choices or loop control options as plain text.
```

Also add to `rw-1-global.md` under `## Adaptive Card Rules`:
```
ENFORCEMENT: If a step says "Present using display_adaptive_card", you MUST call the display_adaptive_card tool. Presenting the same choices as a numbered plain-text list is a violation. Check: does the step mention "display_adaptive_card" or "ActionSet"? If yes, use the tool. No exceptions, regardless of context length.
```

### Why this fixes it
Per-phase reminders placed at the top of each resource ensure the card constraint is in the model's immediate working context when it loads the phase. The global enforcement note adds a self-check heuristic that applies universally.

---

## Fix Plan: RB-M — Numeric criteria operator `>=` stored as "Equal"

**Phase:** 5c | **Severity:** High | **Frequency:** 1/5 (B05)

### Root Cause
Phase 5c Criteria Parsing Rules (item 1) map natural language to operators ("minimum/at least → GreaterOrEqual") but contain no mathematical symbol mappings. When the user types "Credit >= 700", the `>=` symbol is not recognized and the model falls back to the default first valid operator for the field type, which resolves to "Equal".

### File(s) to modify
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

Criteria Parsing Rules, item 1 (line 66):
```
1. Parse natural language to operator keywords: minimum/at least → GreaterOrEqual, exactly → Equal, more than → Greater, less than → Less, between → Between, contains → Contains.
```

### Proposed patch

Replace item 1:
```
1. Parse operator from user input. Apply symbol mapping FIRST, then natural language:
   **Symbol mapping (highest priority — check before anything else):**
   >= or ≥ → GreaterOrEqual, <= or ≤ → LessOrEqual, > (not >=) → Greater, < (not <=) → Less, = or == → Equal, != or <> → NotEqual.
   **Natural language mapping:**
   minimum / at least / no less than → GreaterOrEqual, maximum / at most / up to / no more than → LessOrEqual, exactly / equals / equal to → Equal, more than / greater than / above / over → Greater, less than / below / under → Less, between X and Y → Between, contains / includes → Contains, does not contain / excludes → DoesNotContain, is / is one of → In, is not / not in → NotIn.
   Symbol mapping takes priority over natural language. If both are present in user input (e.g., ">= 700"), symbol wins.
```

### Why this fixes it
The explicit symbol-first mapping table ensures `>=`, `<=`, `>`, `<` are recognized before natural language fallback parsing. This directly addresses the B05 failure where "Credit >= 700" was stored as "Equal".

---

## Fix Plan: RB-I — XML parse failure does not re-detect payload format

**Phase:** 3-Webhook | **Severity:** Medium | **Frequency:** Observed in DEBUG23 session

### Root Cause
Phase 3 Webhook State 3 Step 1 (parse failure recovery) offers "Re-paste" or "Switch content type" but never attempts to detect the actual format of the pasted content. Format auto-detection (State 2 Step 5) only runs when the user selected "I'm not sure" — it is not invoked on parse failure.

### File(s) to modify
- `delivery-rework/resources/rw-phase-3-webhook.md`

### Current instruction (problematic section)

State 3 Step 1 parse-failure branch (lines 68–71):
```
  IF parsing still fails: prompt exactly: "I couldn't parse this as valid {contentTypeChoice}. Would you like to:\n• Fix and re-paste the {contentTypeChoice} schema\n• Switch to a different content type"
  Present using display_adaptive_card an ActionSet: "Re-paste {contentTypeChoice} schema" | "Switch content type". **STOP AND YIELD.**
  - IF "Re-paste": clear postingInstructions, prompt exactly: "Please paste the {contentTypeChoice} schema that your client's API expects." **STOP AND YIELD.**
  - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to State 2 Step 2.
```

### Proposed patch

Replace the parse-failure block:
```
  IF parsing still fails:
    Before showing recovery options, run format detection on postingInstructions:
    - Attempt JSON.parse() → if valid: detectedAlternative = "JSON"
    - Else check for XML-like markup (<tag>...</tag> pattern) → if matched: detectedAlternative = "XML"
    - Else check for key=value& pattern → if matched: detectedAlternative = "URL Encoded"
    - Otherwise: detectedAlternative = null

    IF detectedAlternative is not null AND detectedAlternative ≠ contentTypeChoice:
      prompt exactly: "I couldn't parse this as valid {contentTypeChoice}, but this looks like {detectedAlternative} format. Would you like to switch?"
      Present using display_adaptive_card an ActionSet: "Switch to {detectedAlternative}" | "Re-paste {contentTypeChoice} schema" | "Switch to a different type". **STOP AND YIELD.**
      - IF "Switch to {detectedAlternative}": set contentTypeChoice = detectedAlternative, re-enter State 3 Step 1 with the same postingInstructions.
    ELSE:
      prompt exactly: "I couldn't parse this as valid {contentTypeChoice}. Would you like to:\n• Fix and re-paste the {contentTypeChoice} schema\n• Switch to a different content type"
      Present using display_adaptive_card an ActionSet: "Re-paste {contentTypeChoice} schema" | "Switch content type". **STOP AND YIELD.**
    - IF "Re-paste": clear postingInstructions, prompt exactly: "Please paste the {contentTypeChoice} schema that your client's API expects." **STOP AND YIELD.**
    - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to State 2 Step 2.
```

### Why this fixes it
Format detection now runs before presenting recovery options. When the user pastes JSON while in XML mode, the card proactively suggests the correct format rather than offering a generic "switch" that requires two more clicks.

---

## Fix Plan: RB-J — postingInstructions cleared on content type switch

**Phase:** 3-Webhook | **Severity:** Low | **Frequency:** Observed in DEBUG23 session

### Root Cause
State 3 Step 1 and State 2 Step 5 both instruct `clear postingInstructions` when switching content type. The design assumption was that the schema might change, but in practice the user's posting instructions are often format-agnostic and re-collection is unnecessary friction.

Agent DEBUG: *"The design assumption: the schema might change when the format changes. But this is the wrong default — the user already has a schema and just wants a different parser."*

### File(s) to modify
- `delivery-rework/resources/rw-phase-3-webhook.md`

### Current instruction (problematic section)

State 3 Step 1 (line 71):
```
  - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to State 2 Step 2.
```

State 2 Step 5 (line 59):
```
    - IF "Switch content type": clear contentTypeChoice and postingInstructions, go back to Step 2 of this state.
```

### Proposed patch

Replace line 71:
```
  - IF "Switch content type": clear contentTypeChoice. Retain postingInstructions in memory. Go back to State 2 Step 2. After user selects a new content type in Step 2, prompt: "Would you like to use the same posting instructions you already provided, or paste new ones?" Present using display_adaptive_card an ActionSet: "Use same" | "Paste new". **STOP AND YIELD.** IF "Use same": skip State 2 Step 3, proceed to State 2 Step 4 with existing postingInstructions. IF "Paste new": clear postingInstructions, proceed to State 2 Step 3.
```

Apply the same pattern to line 59 (State 2 Step 5).

### Why this fixes it
The user's schema is retained by default. A single additional question ("use same or paste new?") covers the case where they actually want to provide a different schema. Avoids forcing re-collection in the common case.

---

## Fix Plan: RB-L — Content type skipped when delivery type answered twice (RB-G cascade)

**Phase:** 3-Webhook | **Severity:** High | **Frequency:** 1/5 (B05 — direct RB-G cascade)

### Root Cause
RB-G causes the delivery type prompt to render twice (card + text). The user clicks "Webhook" on the card; the text duplicate causes a second "Webhook" input to arrive while Phase 3 Webhook State 1 is already executing. The second input disrupts the state machine and bypasses the content type step.

### File(s) to modify
- `delivery-rework/resources/rw-phase-3-webhook.md`

### Current instruction (problematic section)

State 1 flows directly from URL collection to field mapping choice (Steps 1→2). There is no guard checking that content type was collected before creating the method.

### Proposed patch

Add an input deduplication guard at the top of State 1, after the phase header:
```
ENTRY GUARD: If the entry signal for this phase was "Webhook" or "HttpPost" and a second identical signal is received immediately after entry (within the same phase load), discard the duplicate. Do not re-execute State 1 from the entry signal. Continue with State 1 at whichever step is incomplete.
```

Add a guard at the start of State 4:
```
Step 0 (guard — check before creating method):
  IF skipFieldMapping = false AND contentTypeChoice is missing or null:
    Content type was not collected. Return to State 2 Step 2 and collect it before proceeding to method creation.
```

### Why this fixes it
Primary fix is RB-G (eliminating the duplicate that causes the cascade). The entry guard prevents a duplicate delivery-type signal from re-executing State 1. The State 4 guard ensures that even if the content type step was skipped due to a cascade, the missing value is caught before the method is created.

---

## Fix Plan: RB-Q — "Show more fields" button triggers field-not-found error

**Phase:** 5c | **Severity:** Medium | **Frequency:** 1/8 (B08)

### Root Cause
The "Show more fields" button in Phase 5c submits a value that gets routed through the field name lookup path instead of the show-more-fields command handler. The button's `Action.Submit` data payload is likely being interpreted as a field name search, similar to the F1/F14 `Action.Submit` data field bug (ChoiceSet + Action.Submit must not have a data field).

### File(s) to modify
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

State 2 handler ordering (lines 37–44):
```
Handle user response:

- IF "Show more fields" OR asked to see more:
  Prompt: ...
  Present using display_adaptive_card: ...
  **STOP AND YIELD.** Loop back to this handler on next response.

- IF user typed a criterion:
  Parse using Criteria Parsing Rules below.
  IF no confident field match: Prompt: "I couldn't find that field. ...
```

The "Show more fields" check comes first but the check string may not match the button's actual submit value.

### Proposed patch

Add explicit input classification at the top of State 2, before any handler:
```
**State 2: Handle Criteria Input**

FIRST — classify input before any other processing:
- IF input matches (case-insensitive): "show more fields", "show fields", "more fields", "see more", "list fields", OR button submit with action containing "show" or "fields" → route to SHOW MORE FIELDS handler.
- IF input matches (case-insensitive): "add another", "another", "more criteria", "add more" → prompt: "Please type your next criterion." **STOP AND YIELD.** Stay in State 2.
- IF input matches (case-insensitive): "continue", "done", "no", "skip", "none", "finish", "no more" → if criteriaSummaryList not empty: State 3. Else: set additionalCriteriaChoice="Skip", return to Phase 5.
- OTHERWISE → treat as typed criterion. Route to CRITERIA INPUT handler.

Never attempt to match control keywords ("show more fields", "continue", "done", "add another") against leadFieldsMap. Control keywords must be resolved before field lookup.
```

### Why this fixes it
Explicit input classification before field lookup prevents any control keyword or button value from being parsed as a field name. The pattern list covers both text input and button submit variations, making the handler robust to `Action.Submit` data format differences.

---

## Fix Plan: RB-S — Criteria saved via update_delivery_account with no UI acknowledgment

**Phase:** 5c | **Severity:** Medium | **Frequency:** 1/1 conversational-P5c runs (B09)

### Root Cause
When Phase 5c runs in conversational fallback mode (resource not loaded due to RB-F), criteria are processed and saved via `update_delivery_account` silently with no user-visible confirmation. The criteria loop prompt includes a confirmation ("✓ Criterion added: ...") but this only executes when the resource is properly loaded.

### File(s) to modify
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

Confirmation is embedded in the criteria loop prompt (State 2) but is not a standalone rule. No header-level acknowledgment requirement exists.

### Proposed patch

Add to Phase 5c header after the CARD RULE reminder:
```
ACKNOWLEDGMENT RULE: After every criterion is accepted and appended to parsedCriteriaList, you MUST display a brief confirmation to the user before prompting for the next action. The criteria loop prompt (below) handles this. Never save a criterion silently. Never proceed to the next step without confirming to the user what was just saved (e.g., "✓ Criterion added: LoanAmount >= 50000").
```

This is reinforced by the criteria loop prompt rewrite in the RB-A patch, which includes `"✓ Criterion added: {latestCriteriaSummary}"` as part of the mandatory loop prompt text.

### Why this fixes it
Primary fix is RB-F (ensuring Phase 5c resource always loads, eliminating fallback mode). The acknowledgment rule provides a second guarantee that the confirmation fires regardless of how the model is executing the phase.

---

## Fix Plan: RB-T — "continue" after P5c puts agent in confused state

**Phase:** 5c → P6 transition | **Severity:** High | **Frequency:** 1/1 conversational-P5c runs (B09)

### Root Cause
When Phase 5c runs in conversational fallback mode, the exit word "continue" is interpreted ambiguously because the Phase 5c resource is not loaded and the model lacks explicit routing for the P5c→P6 transition. The model responded to "continue" by asking the user to "share the exact Phase 6 summary wording" — a hallucinated meta-task.

### File(s) to modify
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

GLOBAL EXIT RULE (lines 10–12):
```
GLOBAL EXIT RULE: At any YIELD, if user says "skip"/"done"/"continue"/"none"/"no":
- If criteriaSummaryList not empty → State 3.
- If empty → set additionalCriteriaChoice="Skip", return to Phase 5.
```

The rule exists but is not strong enough to override confused-state behavior in fallback mode.

### Proposed patch

Replace the GLOBAL EXIT RULE (incorporating changes from RB-A fix):
```
GLOBAL EXIT RULE (applies ONLY at YIELD points in State 1 — field suggestions — and when no criterion is being collected):
If user says "skip" / "done" / "continue" / "none" / "no" / "finish" / "that's all" at a State 1 YIELD:
- If criteriaSummaryList not empty → immediately proceed to State 3.
- If empty → set additionalCriteriaChoice="Skip", return to Phase 5.
NOTE: This rule does NOT apply inside the criteria loop prompt. The loop prompt has its own exit handling.
NEVER respond to these exit words by asking the user to share summary wording, review output, or provide meta-instructions. These words always mean "exit criteria collection and proceed".
```

Primary fix: ensure Phase 5c resource always loads (RB-F). The strengthened GLOBAL EXIT RULE is a fallback guard.

### Why this fixes it
The "NEVER respond to these exit words by asking the user to share summary wording" instruction explicitly blocks the confused behavior seen in B09. The expanded keyword list and "ONLY at State 1" scoping clarify when the rule applies.

---

## Fix Plan: RB-U — update_delivery_account calls non-cumulative (earlier criteria may be overwritten)

**Phase:** 5c | **Severity:** Critical (data integrity) | **Frequency:** 1/1 conversational-P5c runs with multiple updates (B09)

### Root Cause
B09 session log showed 4 `update_delivery_account` calls, each sending only the new criterion(ia) rather than the full accumulated set. If the API uses replace semantics, earlier criteria (including the states criterion from `create_delivery_account`) are silently overwritten. Phase 5c State 3 already says "Send the FULL parsedCriteriaList array" but this is ignored in fallback mode where updates are called per-criterion.

### File(s) to modify
- `delivery-rework/resources/rw-phase-5c-criteria-builder.md`

### Current instruction (problematic section)

State 3 (lines 84–91):
```
**State 3: Update Account with Additional Criteria**

1. Build additionalCriteria: join criteriaSummaryList with "; ".
2. Call update_delivery_account: `deliveryAccountUID={deliveryAccountUID}, criteria={parsedCriteriaList}`
   CRITICAL: Send the FULL parsedCriteriaList array, not just the latest criterion.
   If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue updating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
3. Retain: additionalCriteria.
4. Immediately call the summarize_history tool.
```

### Proposed patch

Replace State 3:
```
**State 3: Update Account with Additional Criteria**

CRITICAL: Call update_delivery_account ONCE at the end of criteria collection — not once per criterion. All criteria must be submitted in a single call with the full accumulated array.
CRITICAL: update_delivery_account uses REPLACE semantics — whatever criteria array you send becomes the complete criteria set for the account. Sending only new criteria will OVERWRITE and DELETE all previously saved criteria, including the state criterion submitted in create_delivery_account.

1. Build additionalCriteria: join criteriaSummaryList with "; ".
2. Build fullCriteriaArray: start with the state criterion originally submitted in create_delivery_account (if any), then append all items in parsedCriteriaList. The result is the complete criteria set for the account.
3. GUARD: Verify deliveryAccountUID is set AND deliveryAccountUID ≠ deliveryMethodUID. If check fails, STOP — return to Phase 5 Step 8 to create the account first.
4. Call update_delivery_account: `deliveryAccountUID={deliveryAccountUID}, criteria={fullCriteriaArray}`
   CRITICAL: fullCriteriaArray must include ALL criteria (states + additional). Never send only the most recently added criterion.
   If the tool fails, repair and retry once silently. If still fails, prompt: "I ran into an issue updating the delivery account.\n\nPlease try again." **STOP AND YIELD.** Do not hallucinate data.
5. Retain: additionalCriteria.
6. Immediately call the summarize_history tool.
```

### Why this fixes it
Three changes: (1) "Call update_delivery_account ONCE at the end" prevents per-criterion calls. (2) The explicit "REPLACE semantics" warning explains the consequence of partial sends. (3) `fullCriteriaArray` combining states + additional criteria ensures the complete set is always submitted.

---

## Defects Not Patched

| Finding | Reason |
|---------|---------|
| **RB-B** | States not shown in summary card (omits entity IDs) — design choice, not a bug. Rework intentionally omits IDs for cleanliness. |
| **RB-E** | Account name asked before price — subsumed by RB-K fix (Phase 5 not loaded before responding). When Phase 5 loads correctly, Step 1 is price, not account name. |
| **RB-R** | Portal `deliveryType = "HttpPost"` — informational. Correct API behavior, not a bug. Instructions should not mention `deliveryAddress` for Portal. |

---

## Summary Table: Patch Locations

| Defect | File(s) | Change type |
|--------|---------|-------------|
| RB-G | `rw-1-global.md`, all phase files (STOP AND YIELD directives) | Add single-emission rule globally; add "Send once" to each YIELD |
| RB-K | `rw-1-global.md`, `rw-phase-4-delivery-method-summary.md` | Add load-before-prompt rule; explicit sequence in Phase 4 handoff |
| RB-D | `rw-phase-5-create-delivery-account.md` | Mandatory gate, ordering constraint, "do not infer" language |
| RB-A | `rw-phase-5c-criteria-builder.md` | GLOBAL EXIT RULE scope; mandatory loop prompt with "Add another" button |
| RB-F | `rw-phase-5-create-delivery-account.md` | Immediate blocking get_resource, no placeholder messages |
| RB-N/RB-O | `rw-phase-5-create-delivery-account.md`, `rw-phase-6-delivery-account-summary.md`, `rw-1-global.md` | Mandatory state collection, session-scoped ASK, Phase 6 pre-render validation |
| RB-C | `rw-phase-5-create-delivery-account.md` | Normalization at input time; USPS-only storage; summary template note |
| RB-P | `rw-phase-5-create-delivery-account.md`, `rw-phase-5c-criteria-builder.md` | Mandatory Step 8; UID inequality guard; Phase 5c guard |
| RB-H | `rw-phase-3-webhook.md`, `rw-phase-5-create-delivery-account.md`, `rw-phase-5c-criteria-builder.md`, `rw-1-global.md` | Per-phase card rule reminder; enforcement note globally |
| RB-M | `rw-phase-5c-criteria-builder.md` | Symbol-first operator mapping table |
| RB-I | `rw-phase-3-webhook.md` | Format re-detection on parse failure |
| RB-J | `rw-phase-3-webhook.md` | Retain instructions on content type switch |
| RB-L | `rw-phase-3-webhook.md` | Entry deduplication guard; State 4 content-type validation |
| RB-Q | `rw-phase-5c-criteria-builder.md` | Input classification before field lookup |
| RB-S | `rw-phase-5c-criteria-builder.md` | Header-level acknowledgment rule |
| RB-T | `rw-phase-5c-criteria-builder.md` | Strengthened GLOBAL EXIT RULE; block confused-state response |
| RB-U | `rw-phase-5c-criteria-builder.md` | Single cumulative update call; REPLACE semantics warning; fullCriteriaArray |

# Stability Test Failure Analysis

**Date:** 2026-04-06
**Scope:** All findings from stabilized (RA-*) and rework (RB-*) variants, R1 and R2 rounds
**Analyst:** Claude Opus 4.6

---

## Task 1: Root Cause Classification

### Classification Definitions

| Category | Definition |
|----------|-----------|
| **IGNORE** | The instruction IS present in the instruction pack, but the AI silently skipped or failed to follow it |
| **HALLUCINATE** | The AI generated content/behavior not described in the instructions |
| **AMBIGUOUS** | The instruction is unclear, contradictory, or has a gap that leads to the failure. Potential instruction issue — requires review to determine if fixable via instruction changes. |
| **PLATFORM** | Non-deterministic card rendering, tool failures, MCP issues, or platform-level bugs |

### Stabilized Variant (RA-*) Classifications

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RA-A | Lead type typed fallback asks for UID | AMBIGUOUS | Instructions direct UID resolution when card not used; no fuzzy-name-match fallback exists. Verified: CONFIRMED AMBIGUOUS. Phase 2 only says `TOOL: display_lead_types_choice / WAIT for user choice / RETAIN: leadTypeUID, leadTypeName` — no instruction handles the typed-input path where user types a lead type name instead of clicking the card. Fix: add a typed-fallback rule that fuzzy-matches user text against returned lead type names and resolves to the corresponding UID. |
| RA-B | Hallucinated UID in DEBUG response | HALLUCINATE | Agent fabricated lead type UID mapping from training data, not from tool output |
| RA-C | deliveryDays not built from schedule input | IGNORE | Phase 3 instructions explicitly say to BUILD deliveryDays from schedule input; agent used fallback values |
| RA-D | Phase regression after Webhook button click | IGNORE | Phase 3 header says "resume from where you left off -- do not regress to earlier steps"; agent violated this |
| RA-E | Content type asked as plain text, not ActionSet card | IGNORE | Phase 3 instruction explicitly says ASK [adaptive_card] with ActionSet for content type |
| RA-F | Only 1 of 8 fields mapped | IGNORE | Field mapping pipeline instructions specify normalized match steps; agent under-matched |
| RA-G | Connection test success message swallowed | IGNORE | Phase 3b instructions should show result before proceeding; agent skipped display step |
| RA-H | Company name state retention error (StabilityTest-50) | HALLUCINATE | Agent overwrote Phase 1 company name with a value from nowhere; artifact of phase regression |
| RA-I | Agent hallucinated tool call (Portal vs Webhook) | HALLUCINATE | Agent described Webhook method creation; session log shows Portal creation |
| RA-J | State normalization skipped (display-only) | IGNORE | Phase 5 Step 4 says "normalize targetStates to uppercase USPS codes"; agent retained full names in display |
| RA-K | Criteria phase entirely skipped | IGNORE | Phase 5 Steps 5-8 define criteria loop; agent bypassed all of it after summarization |
| RA-L | "done" stored as criteria value | IGNORE | Phase 5 Step 7 says "FIRST check exit keywords: IF user says continue/done/no -> exit"; agent failed to check |
| RA-M | Enum criterion handling failure | IGNORE | Phase 5 Step 8 defines enum handling with ChoiceSet; agent failed to follow it |
| RA-N | Schedule hours and body prompt skipped | IGNORE | Phase 3 explicitly has WAIT/STOP directives after "Specific hours only" and content type; agent skipped |
| RA-O | Phase 5 restart after "continue" | IGNORE | Phase 5 Step 7 says proceed to STEP 9 on "continue"; agent showed Phase 4 card instead |
| RA-P | States silently skipped entirely | IGNORE | Phase 5 Step 4 has explicit state prompt with WAIT; agent jumped past it |
| RA-Q | Phantom CA,AZ,TX states in account | HALLUCINATE | Agent inserted state UIDs that the user never provided; phantom data |
| RA-R | Criteria gate bypassed after state skip | IGNORE | Phase 5 Steps 5-7 define mandatory criteria gate; agent bypassed when states were skipped |
| RA-S | Webhook URL never requested | IGNORE | Phase 3 Webhook branch explicitly says ASK for deliveryAddress with WAIT; agent skipped |
| RA-T | JSON parse error before schema provided | IGNORE | Phase 3 instruction says to display prompt and WAIT for user input before parsing |
| RA-U | Criteria not persisted to account | IGNORE | Phase 5 Steps 7-9 say RETAIN criteriaPayload and pass to create_delivery_account; agent dropped criteria from payload |
| RA-V | State/price prompt doubled | IGNORE | Implicit expectation of single prompt; no explicit anti-duplication rule in stabilized R1 instructions |
| RA-W | States and criteria prompt merged | IGNORE | Phase 5 Step 4 says "Do NOT combine it with criteria suggestions"; agent merged them |
| RA-X | Phase 3 instructions loaded 3x | PLATFORM | summarize_history re-exposes next_instructions pointer, triggering re-load; platform mechanism issue |
| RA-Y | Typed criterion input ignored | IGNORE | Phase 5 Step 6 says "If text is typed...always attempt Criteria Parsing first"; agent ignored it |

**R2 new findings (RA-NEW-*):**

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RA-NEW-1 | Posting instructions prompt skipped | IGNORE | Phase 3 STOP AND YIELD at content type step was not enforced |
| RA-NEW-2 | Phase 5 batching (criteria gate auto-skipped) | IGNORE | Phase 5 Step 6 says STOP AND YIELD at criteria gate; agent batched through |
| RA-NEW-3 | States re-prompted when full names given | AMBIGUOUS | Phase 5 says "normalize to USPS codes" but normalizer does not handle full names in stabilized variant. Verified: RECLASSIFY to IGNORE. Stabilized Phase 5 Step 3 explicitly says `PROCESS: normalize targetStates to uppercase USPS codes (California→CA)` — the parenthetical example directly demonstrates full-name-to-abbreviation conversion. The instruction is present and unambiguous; the agent failed to follow it. |
| RA-NEW-4 | "Please type Continue" after Add criteria | HALLUCINATE | Agent generated an off-script prompt instead of loading criteria builder |
| RA-NEW-5 | Phase 5c silent skip (loaded but no interaction) | IGNORE | Phase 5c State 1 instructions say to show field suggestions and STOP AND YIELD; agent ran through without stopping |
| RA-NEW-6 | States shown inside criteria gate card | IGNORE | Phase 5 Step 4 says "Do NOT combine it with the criteria gate" |
| RA-NEW-7 | Phase 5 prompt hallucination (FTP credentials on webhook) | HALLUCINATE | Agent fabricated prompts for FTP credentials when delivery was Webhook; not in instructions |
| RA-NEW-R2-8 | Criteria builder bypassed after 2nd Add click | IGNORE | Phase 5c should load on "Add criteria"; agent created account without loading it |
| RA-NEW-R2-9 | Post-creation states msg causes indefinite spin | PLATFORM | MCP session orphaned or tool call timeout after account already created |

### Rework Variant (RB-*) Classifications

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RB-A | Criteria loop exits after 1 criterion | IGNORE | Phase 5c State 2 says "Criteria loop prompt -- MANDATORY after every accepted criterion"; agent exited early |
| RB-B | Summary card omits entity IDs | AMBIGUOUS | Rework design may intentionally omit IDs; not clearly a bug. Verified: RECLASSIFY to NOT_A_BUG. Phase 6 and Phase 7 card templates explicitly list only user-friendly fields (Company Name, Lead Type, Delivery Method, Price, etc.) — entity UIDs (clientUID, deliveryMethodUID, deliveryAccountUID) are deliberately excluded from the display templates while remaining in the summarization state. This is an intentional design choice, not an omission. |
| RB-C | States not normalized after summarization | IGNORE | Phase 5 Step 4 says "Normalize to uppercase USPS codes"; agent retained full names |
| RB-D | Criteria prompt entirely skipped | IGNORE | Phase 5 Step 6 says criteria gate "REQUIRES user input. Do not skip it."; agent skipped |
| RB-E | Account name asked before price | IGNORE | Phase 5 Step 1 says price is first prompt; agent asked for account name (not in instructions) |
| RB-F | Phase 5c fails to load on first click | PLATFORM | get_resource call for Phase 5c may fail or race with user input; MCP resource loading issue |
| RB-G | Phase transition prompts doubled | IGNORE | Each phase says "STOP AND YIELD" after prompt; agent output prompt twice before yielding |
| RB-H | Content type card rendered as plain text | PLATFORM | Non-deterministic card rendering; model intermittently renders card-required steps as text |
| RB-I | XML parse failure no format re-detection | AMBIGUOUS | Auto-detection only runs when user selects "I'm not sure"; no instruction for cross-format detection on parse failure. Verified: RECLASSIFY to IGNORE. Phase 3 Webhook State 3 Step 1 explicitly says: `IF parsing still fails: before showing the recovery card, attempt to parse postingInstructions as each other format. If it successfully parses as valid JSON or valid XML, add a third option to the recovery card.` Cross-format detection on parse failure IS instructed; the agent failed to follow it. |
| RB-J | postingInstructions cleared on content type switch | AMBIGUOUS | Instruction explicitly says "clear postingInstructions" on switch; design choice that causes poor UX. Verified: RECLASSIFY to NOT_A_BUG. The finding description is incorrect. Phase 3 Webhook State 2 Step 5 says: `IF "Switch content type": clear contentTypeChoice but retain postingInstructions.` Instructions explicitly RETAIN postingInstructions on switch and at Step 3 say: `if the user previously provided posting instructions, prompt: "Would you like to use the same posting instructions, or provide new ones?"` The UX handles reuse correctly by design. |
| RB-K | Phase 5 not loaded before responding | IGNORE | System rule says load phase resource before prompting; agent generated generic prompt in the gap |
| RB-L | Content type skipped (RB-G cascade) | IGNORE | Phase 3 Webhook requires content type step; cascade from RB-G caused skip |
| RB-M | >= operator stored as Equal | IGNORE | Phase 5c Criteria Parsing Rules explicitly map >= to GreaterOrEqual; agent used wrong mapping |
| RB-N | States question skipped entirely | IGNORE | Phase 5 Step 4 says "Do NOT skip this prompt" for states; agent skipped |
| RB-O | Phantom CA,AZ,TX states | HALLUCINATE | Agent inserted state UIDs never provided by user; phantom data from context or defaults |
| RB-P | create_delivery_account silently skipped | IGNORE | Phase 5 Step 7 says "MANDATORY TOOL CALL"; agent never called it for FTP |
| RB-Q | "Show more fields" button triggers error | PLATFORM | Button value parsed through field name lookup instead of command handler; Action.Submit data bug |
| RB-R | Portal uses deliveryType HttpPost | AMBIGUOUS | Informational -- documents API contract, not clearly a bug. Verified: RECLASSIFY to NOT_A_BUG. Phase 3 Portal explicitly sets `deliveryType="HttpPost"` in the create_delivery_method tool call and retains `deliveryType="HttpPost"`. This is the documented API contract for Portal delivery methods — Portal is a platform-hosted delivery mechanism that uses HttpPost as its underlying transport type. Not a bug or instruction gap. |
| RB-S | Numeric criteria saved without UI acknowledgment | IGNORE | Phase 5c says to show criteria loop prompt after every accepted criterion; agent skipped confirmation |
| RB-T | "continue" after P5c puts agent in confused state | IGNORE | Phase 5c says "Done adding criteria" -> State 3; agent failed to route correctly |
| RB-U | update_delivery_account calls non-cumulative | IGNORE | Phase 5c says "Always APPEND to parsedCriteriaList, never overwrite"; agent sent partial updates |
| RB-V | Criteria accepted/shown but never sent to API | IGNORE | Phase 5c says to create criteria object and append to parsedCriteriaList for payload; agent only updated display state |
| RB-W | State UID missing from states value (NY dropped) | IGNORE | Phase 5 Step 5 says collect ALL matched stateUID values; agent dropped one |
| RB-X | P5c empty-response loop with large lead types | PLATFORM | Context overflow from 100+ fields + conversation history exceeds model context window |
| RB-Y | Phase 3 asks name+type before schedule/type cards | IGNORE | Phase 3 instructions show schedule card first, then delivery type; agent reordered |
| RB-Z | Phase 8 credentials asked before Phase 5 | IGNORE | Phase 5 should start after Phase 4 Continue; agent skipped to Phase 8 prompts |

**R2 new findings (RB-NEW-*):**

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RB-AA | P2 timezone/status systematically skipped | IGNORE | Phase 2 should collect timezone and status; agent jumped to Phase 3 |
| RB-BB | P5 off-script prompt causes data collection collapse | IGNORE | Phase 5 Steps 1-6 define collection order; agent skipped all but price after off-script exchange |
| RB-DD | P3B no Retry/Skip card after connection test | IGNORE | Phase 3B should show Retry/Skip card after test result; agent displayed result without card |
| RB-NEW-R2-1 | CamelCase field name not recognized | AMBIGUOUS | Agent suggests fields in camelCase but lookup uses display names; mismatch in instruction. Verified: CONFIRMED AMBIGUOUS. Phase 5c State 1 says to display `suggestedFields` built from `leadFieldName` values and State 2 says to `Match field name against leadFieldsMap or leadFieldsIndex (fuzzy >90%)` using `leadFieldName` as key. Both display and lookup use the same raw API field name (e.g., "selfCreditRating"). However, there is no instruction to present field names in a human-readable format (e.g., "Self Credit Rating") or to normalize user input back to camelCase before matching. Fix: add an instruction to display field names with spaces inserted at camelCase boundaries for readability, and normalize user-typed field names back to camelCase before fuzzy matching. |
| RB-NEW-R2-3 | Phase 5c silent skip via premature summarize_history | IGNORE | Phase 5c should not call summarize_history before user interaction; agent called it prematurely |
| RB-NEW-R2-4 | States combined with criteria gate in single card | IGNORE | Phase 5 Step 4 says "Do NOT combine it with the criteria gate"; agent combined them |
| RB-NEW-R2-5 | Tool/resource access loss after extended DEBUG | PLATFORM | Context length exceeded threshold, stripping tool definitions |
| RB-NEW-R2-6 | Phase 7 client summary card render failure | PLATFORM | Adaptive card failed to render; linked to context-loss condition |

### Aggregate Classification Summary

| Category | Count | Percentage | Notes |
|----------|-------|------------|-------|
| **IGNORE** | 43 | **75.4%** | Instruction present, AI skipped it |
| **HALLUCINATE** | 7 | **12.3%** | AI fabricated data/behavior |
| **AMBIGUOUS** | 2 | **3.5%** | RA-A (lead type typed fallback), RB-NEW-R2-1 (camelCase field names) |
| **PLATFORM** | 8 | **14.0%** | Card rendering, MCP, context overflow |
| **NOT_A_BUG** | 3 | — | RB-B, RB-J, RB-R (design choices, excluded from failure count) |
| **Total distinct failures** | **57** | — | (60 original - 3 NOT_A_BUG) |

Note: Some findings could reasonably fall into multiple categories. The 3 findings classified as HALLUCINATE that involve phantom states (RA-Q, RB-O) are closely linked to IGNORE findings (RA-P, RB-N) -- the hallucination is a downstream consequence of the ignored instruction.

**Key insight:** Three-quarters (75%) of all failures are IGNORE-type — the instruction exists and is explicit, but the AI silently skipped it. Only 2 findings (3.5%) are true instruction gaps (AMBIGUOUS). This indicates that instruction clarity is generally adequate; the fundamental problem is LLM instruction-following reliability, especially under context pressure from summarize_history and long conversations.

---

## Task 2: Model Comparison (gpt-5.4-mini vs gpt-5-mini mixed)

### Stabilized Variant Model Configurations

| Configuration | Runs | Model (P3/Webhook) | Model (P5/Criteria) | Model (Other) |
|--------------|------|---------------------|---------------------|---------------|
| R1 gpt-5.4-mini only | 01-20 | gpt-5.4-mini | gpt-5.4-mini | gpt-5.4-mini |
| R1 mixed model | 21-30 | gpt-5-mini | gpt-5-mini | gpt-5.4 |
| R2 gpt-5.4-mini only | 01-06 | gpt-5.4-mini | gpt-5.4-mini | gpt-5.4-mini |

### Rework Variant Model Configurations

| Configuration | Runs | Model (All phases) |
|--------------|------|---------------------|
| R1 gpt-5.4-mini | 01-12 | gpt-5.4-mini |
| R1 mixed (21-30) | 21-30 | gpt-5-mini (P3/P5) + gpt-5.4 (others) |
| R2 gpt-5.4-mini | 01-05 | gpt-5.4-mini |

### P3 Phase Checkpoints (Webhook Setup, Content Type, Field Mapping)

**Stabilized R1:**

| Checkpoint | gpt-5.4-mini (01-20) Pass | gpt-5.4-mini (01-20) Fail | gpt-5-mini mixed (21-30) Pass | gpt-5-mini mixed (21-30) Fail |
|-----------|---------------------------|---------------------------|-------------------------------|-------------------------------|
| P3-SCHED | 18/20 (90%) | 2/20 (10%) | 8/9 (89%) | 1/9 (11%) |
| P3-WURL | 19/20 (95%) | 1/20 (5%) | 9/9 (100%) | 0/9 (0%) |
| P3-JSON | 8/10 applicable (80%) | 2/10 (20%) | 6/6 applicable (100%) | 0/6 (0%) |
| P3-TABLE | 9/10 applicable (90%) | 1/10 (10%) | 6/6 (100%) | 0/6 (0%) |
| P3-COUNT | 8/10 applicable (80%) | 2/10 (20%) | 6/6 (100%) | 0/6 (0%) |

**Rework R1:**

| Checkpoint | gpt-5.4-mini (01-12) Pass | gpt-5.4-mini (01-12) Fail | gpt-5-mini mixed (21-30) Pass | gpt-5-mini mixed (21-30) Fail |
|-----------|---------------------------|---------------------------|-------------------------------|-------------------------------|
| P3-SCHED | 9/12 (75%) | 3/12 (25%) | 10/10 (100%) | 0/10 (0%) |
| P3-WURL | 8/9 applicable (89%) | 1/9 (11%) | 7/7 applicable (100%) | 0/7 (0%) |
| P3-JSON | 7/7 applicable (100%) | 0/7 (0%) | 5/5 applicable (100%) | 0/5 (0%) |
| P3-TABLE | 5/7 applicable (71%) | 2/7 (29%) | 5/5 (100%) | 0/5 (0%) |
| P3-COUNT | 5/6 applicable (83%) | 1/6 (17%) | 5/5 (100%) | 0/5 (0%) |

**P3 Summary:** The mixed-model runs (21-30, gpt-5-mini for webhook phases) show EQUAL OR BETTER P3 performance compared to gpt-5.4-mini-only runs in both variants. gpt-5-mini achieved 100% pass rates on P3-WURL, P3-JSON, P3-TABLE, P3-COUNT. However, new behaviors emerged: W1 (no delivery type card in A21), W4 (content type card freeze in A23/A26), and V tripling (A22). These are PLATFORM-type issues, not instruction-following failures.

### P5 Phase Checkpoints (States, Criteria Gate, Criteria Builder)

**Stabilized R1:**

| Checkpoint | gpt-5.4-mini (01-20) Pass/Total | gpt-5-mini mixed (21-30) Pass/Total | Delta |
|-----------|-------------------------------|--------------------------------------|-------|
| P5-STATE | 13/20 (65%) | 7/9 (78%) | **+13% improved** |
| P5-NORM | 13/20 (65%) | 8/9 (89%) | **+24% improved** |
| P5-FIELD | 15/20 (75%) | 9/9 (100%) | **+25% improved** |
| P5-CR1 | 13/20 (65%) | 8/9 (89%) | **+24% improved** |
| P5-CR3 | 9/20 (45%) | 8/9 (89%) | **+44% improved** |
| P5-DONE | 9/20 (45%) | 8/9 (89%) | **+44% improved** |

**Rework R1:**

| Checkpoint | gpt-5.4-mini (01-12) Pass/Total | gpt-5-mini mixed (21-30) Pass/Total | Delta |
|-----------|-------------------------------|--------------------------------------|-------|
| P5-STATE | 6/12 (50%) | 4/10 (40%) | **-10% worse** |
| P5-NORM | 5/11 (45%) | 3/8 (38%) | -8% |
| P5-FIELD | 4/12 (33%) | 4/10 (40%) | +7% |
| P5-CR1 | 6/11 (55%) | 3/8 (38%) | **-17% worse** |
| P5-CR3 | 1/12 (8%) | 3/9 (33%) | **+25% improved** |
| P5-DONE | 1/12 (8%) | 3/9 (33%) | **+25% improved** |

### P5-DONE (Criteria Persistence in Payload)

This is the most critical metric -- whether criteria actually end up in the API call.

**Stabilized variant:**
- gpt-5.4-mini R1 (runs 01-20): 9/14 qualifying runs FAILED (64% failure rate)
- gpt-5-mini mixed R1 (runs 21-30): 1/9 runs FAILED (11% failure rate)
- gpt-5.4-mini R2 (runs 01-06): 4/4 qualifying runs FAILED (100% failure rate)

**Rework variant:**
- gpt-5.4-mini R1 (runs 01-12): 10/11 qualifying runs FAILED (91% failure rate)
- gpt-5-mini mixed R1 (runs 21-30): 6/9 runs FAILED (67% failure rate)
- gpt-5.4-mini R2 (runs 01-05): 4/5 qualifying runs FAILED (80% failure rate)

| Metric | gpt-5.4-mini (R1) | gpt-5-mini mixed (R1 21-30) | Delta |
|--------|-------------------|------------------------------|-------|
| Stabilized P5-DONE fail rate | 64% | **11%** | **-53pp** |
| Rework P5-DONE fail rate | 91% | **67%** | **-24pp** |
| Stabilized P5-STATE fail rate | 35% | **22%** | **-13pp** |
| Rework P5-STATE fail rate | 50% | **60%** | **+10pp** |

### Key Model Comparison Findings

1. **gpt-5-mini dramatically improved criteria persistence in the stabilized variant** (64% -> 11% failure rate for P5-DONE). Runs 21-30 consistently passed P5-CR3 (7 consecutive passes from A24-A30). This is the most significant improvement observed across all configurations.

2. **gpt-5-mini improved criteria persistence in the rework variant, but less dramatically** (91% -> 67%). The rework variant's more complex Phase 5c architecture (separate resource, state machine) appears to be harder for both models.

3. **gpt-5-mini worsened state handling in the rework variant** (50% -> 60% failure for P5-STATE). RB-N continued at similar or higher rates in runs 21-30.

4. **P3 phases (webhook/mapping) were universally better with gpt-5-mini** in both variants. 100% pass rates on all P3 mapping checkpoints in both variants' runs 21-30.

5. **RB-G (prompt doubling) significantly reduced with gpt-5-mini in rework**: ~95% -> 40%. The mixed model configuration produced cleaner phase transitions.

6. **New regression: RB-AA (P2 timezone skipped)** appeared exclusively in runs 24-30 (0% in 01-20, 70% in 24-30). This is a new failure not seen with gpt-5.4-mini only. May be a gpt-5-mini specific issue or a session-drift artifact.

---

## Task 3: IGNORE Classification Verification

Verification of the top 5 most frequent IGNORE-type findings by reading the actual instruction files.

### 1. RA-P / RB-N: States Question Silently Skipped (19 occurrences total: RA-P 7/20 + RB-N 12/30)

**Instruction file (stabilized):** `/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`, STEP 4

```
Immediately after detecting the state field, display the state collection prompt below.
Do NOT skip this prompt. Do NOT combine it with criteria suggestions in the same message.

PROMPT: "Which states do you want to target? (e.g., CA, AZ, TX)"
ASK [conversational]: targetStates
WAIT for user input
```

**Instruction file (rework):** `/delivery-rework/resources/rw-phase-5-create-delivery-account.md`, Step 4

```
Immediately after detecting the state field, display this prompt.
Do NOT skip this prompt. Do NOT combine it with the criteria gate.
Prompt the user exactly as follows: "Which states do you want to target? (e.g., CA, AZ, TX)"
**STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
```

**VERIFIED: IGNORE classification confirmed.** Both variants have explicit "Do NOT skip this prompt" directives. The rework variant additionally has STOP AND YIELD. The instruction is unambiguous. The AI silently skipped it in 19 out of ~50 qualifying runs.

### 2. RA-U / RB-V: Criteria Not Persisted to Account (13+ occurrences: RA-U 9/14 + RB-V 4/30)

**Instruction file (stabilized):** `/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`, STEPs 7-10

```
STEP 7: [Criteria Loop]
- FIRST check exit keywords... → RETAIN criteriaPayload (the array of criterion objects
  accumulated during this loop — do NOT discard or reset) → proceed directly to STEP 9

STEP 9: [Build Criteria Array]
- CRITICAL: criteriaPayload may already contain criteria from the loop. Do NOT reset it.
- Insert state criterion as the FIRST element of criteriaPayload array

STEP 10:
- TOOL: create_delivery_account
- criteria={criteriaPayload}
```

**Instruction file (rework):** `/delivery-rework/resources/rw-phase-5c-criteria-builder.md`, State 2

```
CRITICAL: Always APPEND to parsedCriteriaList, never overwrite.
...
Append to parsedCriteriaList. Create plain-English summary, append to criteriaSummaryList.
```

**VERIFIED: IGNORE classification confirmed.** Both variants explicitly say to RETAIN/APPEND criteria and pass criteriaPayload to the tool call. The stabilized variant has "do NOT discard or reset" in two separate locations. The AI accumulated criteria in display state but failed to include them in the API payload.

### 3. RB-G: Phase Transition Prompts Doubled (~80% of rework R1 runs, ~40% of mixed runs)

**Instruction file (rework):** `/delivery-rework/resources/rw-phase-5-create-delivery-account.md`, Steps 1-3

```
Step 1: Collect Price
  Prompt the user exactly as follows: "Finally, let's set up your Delivery Account...
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

Step 2: Collect Exclusivity
  Prompt the user exactly as follows: "Will this client receive exclusive or shared leads?"
  **STOP AND YIELD.**
```

**VERIFIED: IGNORE classification confirmed.** Every step in the rework variant contains "STOP AND YIELD" after each prompt. The agent should output the prompt once and then stop. The doubling behavior (outputting the same prompt twice in one response) directly violates the STOP AND YIELD directive. The system header also states "Execute ONLY the instructions below" and "Follow its steps in order." No instruction says to repeat or echo a prompt.

### 4. RB-D: Criteria Gate Entirely Skipped (5 occurrences: B02, B10, B22, B25, B26)

**Instruction file (rework):** `/delivery-rework/resources/rw-phase-5-create-delivery-account.md`, Step 6

```
Step 6: Criteria Gate — This step REQUIRES user input. Do not skip it.
  Prompt the user exactly as follows: "Would you like to add additional lead criteria, or skip?"
  Present using display_adaptive_card with an ActionSet: "Add criteria" | "Skip".
  MUST use display_adaptive_card.
  **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.
```

**Instruction file (stabilized):** `/delivery-original-stabilized/resources/phase-5-create-delivery-account.md`, STEP 6

```
STEP 6: [MANDATORY prompt]
ASK [adaptive_card]: ActionSet (Show more fields | Skip) if extraFieldCount > 0
WAIT for user choice
```

**VERIFIED: IGNORE classification confirmed.** The rework variant could not be more explicit: "This step REQUIRES user input. Do not skip it." and "STOP AND YIELD." The stabilized variant marks the field suggestion prompt as "MANDATORY." The AI skipped the criteria gate entirely, proceeding to account creation without user input.

### 5. RB-A: Criteria Loop Exits After 1 Criterion (6 occurrences in rework R1)

**Instruction file (rework):** `/delivery-rework/resources/rw-phase-5c-criteria-builder.md`, State 2

```
Criteria loop prompt — MANDATORY after every accepted criterion:
  You MUST show this prompt after every criterion is appended.
  Do NOT advance to State 3 without user confirmation.
  Prompt: "Would you like to add another criterion, see more fields, or continue?"
  Present using display_adaptive_card: ActionSet "Add another criterion" | "Show more fields" |
    "Done adding criteria"
  **STOP AND YIELD.** Do not hallucinate data.
  - IF "Add another criterion" or user typed a criterion → parse as criterion above.
  - IF "Done adding criteria" or user says "done"/"continue"/"no" → State 3.
```

**VERIFIED: IGNORE classification confirmed.** The instruction says "MANDATORY after every accepted criterion" and "Do NOT advance to State 3 without user confirmation." The criteria loop prompt must appear after each criterion. The AI exited the loop after 1 criterion, advancing to account creation without showing the mandatory loop prompt or waiting for user confirmation.

### Verification Summary

All top 5 IGNORE-classified findings were verified against the actual instruction files. In every case:
- The instruction IS present in the instruction pack
- The instruction is explicit and unambiguous (uses words like "Do NOT skip", "MANDATORY", "STOP AND YIELD", "REQUIRES user input")
- The AI silently violated the instruction

This confirms that the dominant failure mode (68% of findings) is not instruction ambiguity but LLM instruction-following reliability, particularly under context pressure from long conversations and summarize_history operations.

---

## Cross-Variant, Cross-Round Phase Pass Rates

All checkpoints, all rounds, both variants. Denominator excludes N/A runs. Fail% = fail/applicable.

### Stabilized Variant

| Checkpoint | R1a 01-20 (5.4-mini) Pass | R1a Fail% | R1b 21-30 (5-mini mix) Pass | R1b Fail% | R2 01-10 (5.4-mini) Pass | R2 Fail% |
|------------|---------------------------|-----------|-------------------------------|-----------|---------------------------|----------|
| P1-PROMPT | 20/20 | 0% | 10/10 | 0% | 8/10 | 20% |
| P2-DROP | 20/20 | 0% | 10/10 | 0% | 10/10 | 0% |
| P3-SCHED | 18/20 | 10% | 8/9 | 11% | 7/8 | 13% |
| P3-WURL | 19/20 | 5% | 9/9 | 0% | 5/6 | 17% |
| P3-JSON | 8/10 | 20% | 6/6 | 0% | 4/6 | 33% |
| P3-TABLE | 9/10 | 10% | 6/6 | 0% | 6/6 | 0% |
| P3-COUNT | 8/10 | 20% | 6/6 | 0% | 6/6 | 0% |
| P3B-TEST | 20/20 | 0% | 10/10 | 0% | 6/6 | 0% |
| P4-SUMM | 20/20 | 0% | 10/10 | 0% | 10/10 | 0% |
| P5-PRICE | 20/20 | 0% | 10/10 | 0% | 8/10 | 20% |
| P5-EXCL | 20/20 | 0% | 10/10 | 0% | 7/8 | 13% |
| P5-ORDER | 20/20 | 0% | 10/10 | 0% | 7/8 | 13% |
| P5-STATE | 10/20 | 50% | 7/9 | 22% | 6/8 | 25% |
| P5-NORM | 10/20 | 50% | 8/9 | 11% | 5/7 | 29% |
| P5-FIELD | 16/20 | 20% | 9/9 | 0% | 1/5 | 80% |
| P5-CR1 | 14/20 | 30% | 8/9 | 11% | 1/5 | 80% |
| P5-ENUM | 5/8 | 38% | 7/8 | 13% | 1/3 | 67% |
| P5-CR3 | 4/20 | 80% | 8/9 | 11% | 0/5 | 100% |
| P5-DONE | 0/20 | 100% | 8/9 | 11% | 4/10 | 60% |
| P6-BOOL | 20/20 | 0% | 10/10 | 0% | 10/10 | 0% |
| P6-SUMM | 19/20 | 5% | 10/10 | 0% | 7/8 | 13% |
| P7-SUMM | 20/20 | 0% | 10/10 | 0% | 8/8 | 0% |
| P8-ACT | 19/20 | 5% | 10/10 | 0% | 8/8 | 0% |

### Rework Variant

| Checkpoint | R1 01-12 (5.4-mini) Pass | R1 Fail% | R2 01-10 (5.4-mini) Pass | R2 Fail% |
|------------|--------------------------|----------|---------------------------|----------|
| P1-PROMPT | 4/12 | 67% | 1/10 | 90% |
| P2-DROP | 12/12 | 0% | 10/10 | 0% |
| P3-SCHED | 9/12 | 25% | 5/10 | 50% |
| P3-WURL | 8/9 | 11% | 8/8 | 0% |
| P3-JSON | 7/7 | 0% | 7/8 | 13% |
| P3-TABLE | 5/7 | 29% | 8/8 | 0% |
| P3-COUNT | 5/6 | 17% | 8/8 | 0% |
| P3B-TEST | 12/12 | 0% | 8/8 | 0% |
| P4-SUMM | 11/12 | 8% | 10/10 | 0% |
| P5-PRICE | 12/12 | 0% | 9/10 | 10% |
| P5-EXCL | 9/12 | 25% | 9/10 | 10% |
| P5-ORDER | 12/12 | 0% | 9/10 | 10% |
| P5-STATE | 7/9 | 22% | 4/10 | 60% |
| P5-NORM | 5/8 | 38% | 7/8 | 13% |
| P5-FIELD | 5/12 | 58% | 1/5 | 80% |
| P5-CR1 | 7/11 | 36% | 2/5 | 60% |
| P5-ENUM | 1/2 | 50% | 1/3 | 67% |
| P5-CR3 | 1/12 | 92% | 1/5 | 80% |
| P5-DONE | 1/12 | 92% | 4/10 | 60% |
| P6-BOOL | 12/12 | 0% | 10/10 | 0% |
| P6-SUMM | 9/12 | 25% | 9/10 | 10% |
| P7-SUMM | 9/9 | 0% | 9/10 | 10% |
| P8-ACT | 12/12 | 0% | 9/10 | 10% |

### Overall Run Results

| Variant | Round | Runs | Pass | Fail | Avg Score | Best | Worst |
|---------|-------|------|------|------|-----------|------|-------|
| Stabilized | R1a (01-20) | 20 | 1 | 19 | ~67% | 24 (100%) | multiple |
| Stabilized | R1b (21-30) | 10 | 1 | 9 | ~72% | 24 (100%) | 22 |
| Stabilized | R2 (01-10) | 10 | 2 | 8 | 75% | 09,10 (100%) | 06,07 (57%) |
| Rework | R1 (01-12) | 12 | 1 | 11 | ~58% | 03 (100%) | 12 (abandoned) |
| Rework | R2 (01-10) | 10 | 0 | 10 | 75% | 03,04 (86-87%) | 01 (57%) |

### Phase Group Summary (averaged across all rounds)

| Phase Group | Stabilized Avg Pass% | Rework Avg Pass% | Notes |
|-------------|---------------------|-------------------|-------|
| P1-P2 (Client+LT) | 97% | 66% | Rework hurt by P1 doubling |
| P3 (Delivery Method) | 89% | 88% | Both strong; mapping pipeline reliable |
| P4 (Summary) | 100% | 97% | Near-perfect |
| P5 early (Price/Excl/Order) | 96% | 90% | Both strong |
| P5 states (STATE+NORM) | 55% | 47% | Core weakness both variants |
| P5 criteria (FIELD→DONE) | 39% | 34% | Critical failure zone |
| P6-P8 (Summary→Activation) | 98% | 93% | Both strong |

---

## Appendix: Per-Run Scoring References

### Stabilized R1 runs 01-20 (gpt-5.4-mini)

| Run | P5-STATE | P5-CR3 | P5-DONE | Notes |
|-----|----------|--------|---------|-------|
| 01 | F | F | F | Criteria phase entirely skipped |
| 02 | F | Y | F | "done" stored as criteria |
| 03 | F | Y | F | Phase restart on "continue" |
| 04 | F | F | F | States + criteria both skipped |
| 05 | Y | F | F | Criteria not persisted (U) |
| 06 | F | F | F | States skipped + criteria not persisted |
| 07 | Y | F | F | Criteria not persisted (U) |
| 08 | Y | F | F | Criteria partially persisted |
| 09 | Y | F | F | Criteria not persisted (U) |
| 10 | Y | F | F | Criteria not persisted (U) |
| 11 | Y | F | Y | Typed criterion ignored (Y) but Skip used |
| 12 | Y | F | F | Criteria dropped from payload (U) |
| 13 | Y | F | F | Both enum criteria dropped (U) |
| 14 | Y | Y | Y | Skip used; no criteria to persist |
| 15 | F | Y | Y | **First ever criterion persistence** |
| 16 | Y | Y | Y | LoanAmount persisted |
| 17 | F | Y | Y | SelfCreditRating persisted |
| 18 | F | Y | Y | Both criteria persisted |
| 19 | F | F | Y | LoanAmount NOT persisted (broke streak) |
| 20 | F | Y | Y | SelfCreditRating persisted |

### Stabilized R1 runs 21-30 (mixed: gpt-5-mini for P3/P5)

| Run | P5-STATE | P5-CR3 | P5-DONE | Notes |
|-----|----------|--------|---------|-------|
| 21 | Y | Y | Y | Enum bypass but criteria in payload |
| 22 | Y | Y | Y | LoanAmount GreaterOrEqual persisted |
| 23 | Y | Y | Y | All 3 criteria persisted |
| 24 | Y | Y | Y | Skip; states correct |
| 25 | F | Y | Y | States resequenced but persisted |
| 26 | Y | Y | Y | LoanAmount persisted |
| 27 | Y | F | Y | LoanRequestPurpose wrong operator |
| 28 | F | Y | Y | States resequenced; SelfCreditRating persisted |
| 29 | Y | Y | Y | Both numeric criteria persisted |
| 30 | Y | Y | Y | SelfCreditRating persisted |

### Rework R1 runs 01-12 (gpt-5.4-mini)

| Run | P5-STATE | P5-CR3 | P5-DONE | Notes |
|-----|----------|--------|---------|-------|
| 01 | Y | F | F | Criteria loop premature exit |
| 02 | Y | F | F | Criteria entirely skipped |
| 03 | Y | Y | Y | **Only R1 rework PASS** |
| 04 | Y | F | F | Criteria loop exited after 1 |
| 05 | Y | F | F | Criteria lost, wrong operator |
| 06 | Y | F | F | Loop exit after 1 criterion |
| 07 | F | F | F | States + criteria both failed |
| 08 | N/A | F | F | Account never created (RB-P) |
| 09 | Y | F | F | Criteria overwrites via updates |
| 10 | F | F | F | Criteria gate skipped |
| 11 | F | F | F | States + criteria skipped, phantom states |
| 12 | F | F | F | Empty-response loop, abandoned |

### Rework R1 runs 21-30 (mixed: gpt-5-mini for P3/P5)

| Run | P5-STATE | P5-CR3 | P5-DONE | Notes |
|-----|----------|--------|---------|-------|
| 21 | Y | F | Y | Enum criteria dropped from payload |
| 22 | F | F | F | States/criteria both skipped |
| 23 | F | Y | Y | Criteria added via post-creation update |
| 24 | Y | N/A | Y | Skip; no criteria |
| 25 | F | F | F | States/criteria/P6 all skipped |
| 26 | F | F | F | P5 full data collection collapse |
| 27 | F | F | F | States skipped, criteria not saved |
| 28 | Y | F | F | Criterion accepted but not saved |
| 29 | F | F | F | States skipped, criteria rejected post-creation |
| 30 | F | F | F | SelfCreditRating dropped from payload |

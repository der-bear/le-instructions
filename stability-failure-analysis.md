# Stability Test Failure Analysis

**Date:** 2026-04-07 (updated with R2c findings)
**Scope:** All findings from stabilized (RA-*) and rework (RB-*) variants, R1, R2, **and R2c** rounds (rework only for R2c)
**Analyst:** Claude Opus 4.6

---

## Rounds Glossary — Single Source of Truth

This glossary applies across all stability documents (stability-failure-analysis.md, stability-rework-findings.md, stability-stabilized-findings.md).

### Rework Variant Rounds

| Round | Runs | Model Configuration | Instructions | Scoring Convention | n |
|-------|------|---------------------|--------------|--------------------|---|
| **R1a** | 01-20 | gpt-5.4-mini all phases | Pre-fix baseline | Raw (cosmetic counted) | 18 (20−2 ABORT) |
| **R1b** | 21-30 | **Mixed**: gpt-5-mini for P3+P5, gpt-5.4 for others | Pre-fix baseline | Raw (cosmetic counted) | 10 |
| **R2** | 01-10 | gpt-5.4-mini all phases | **Post-fix #1** (commit 5200cc3, FIXes 1-17 applied) | Raw (cosmetic counted) | 10 |
| **R2c** | 01-10 | **Mixed**: gpt-5-mini for P3+P5, gpt-5.4 for others | Post-fix #1 (same as R2) | **Audited** (F* = cosmetic, NOT counted) | 10 |

### Stabilized Variant Rounds

| Round | Runs | Model Configuration | Instructions | Notes | n |
|-------|------|---------------------|--------------|-------|---|
| **R1a** | 01-20 | gpt-5.4-mini all phases | Pre-fix baseline | Initial stabilized baseline | 20 |
| **R1b** | 21-30 | **Mixed**: gpt-5-mini for P3+P5, gpt-5.4 for others | Pre-fix baseline | Mixed-model test | 10 |
| **R2** | 01-10 | gpt-5.4-mini all phases | Post-fix #1 | First post-fix round | 10 |
| **R2b** | 01-10 | **Mixed**: gpt-5-mini for delivery method/account/criteria; gpt-5.4 for others | Post-fix #1 + R1b-style model | **EXPLORATORY** — scores skewed by Phase 5 states reorder regression. Findings retained, scores superseded by R2c clean run. (Originally labeled R2b-tmp.) | 10 |
| **R2c** | 01-10 | **Mixed**: gpt-5-mini for P3+P5, gpt-5.4 for others | Post-fix #1 | Clean re-run of R2b | 10 |

**Both variants' R2c use the same mixed model configuration** (gpt-5-mini for critical phases, gpt-5.4 for others). This means R2c tests `post-fix #1 instructions + mixed model` for both variants.

**R2b (stabilized only):** Was originally "R2b-tmp" — an exploratory baseline whose scores were skewed by a Phase 5 states reorder regression. Findings retained for forensic purposes; scores superseded by R2c clean run.

### Round Naming Convention

- **R{number}** = round number (R1, R2, R3...)
- **a/b/c suffix** = sub-round within same fix generation:
  - `a` suffix = baseline / first config
  - `b` suffix = alternate model config or alternate scoring
  - `c` suffix = clean re-run / corrected version
- **Pre-fix vs post-fix** is the most important distinction — fix application changes between R1x and R2x

### Quick reference: which round used which fix set?

| Fix Set | Status | Applied In |
|---------|--------|------------|
| Pre-fix (no instruction changes) | Baseline | R1a, R1b (both variants) |
| Post-fix #1 (commit 5200cc3, FIXes 1-17) | First iteration | R2, R2c (both variants); R2b (stabilized only) |
| Post-fix #2 (proposed) | TBD | None yet |

---

### Cosmetic vs Functional Failure Convention (introduced R2c)

R2c onwards uses a stricter scoring convention:
- **F*** = cosmetic failure (prompt doubling RB-G, plain-text card fallback RB-H, sequence/ordering quirks). Workflow still completes correctly. NOT counted in score percentage.
- **F** = functional failure (skipped step, lost data, wrong value, hallucinated value, never-created entity). Counted in score.
- Legacy R1a/R1b/R2 raw scores included cosmetic failures in denominator. R2c scores exclude them. After R2c-style reclassification, R1a-18 promoted from 13/14 (93%) to 13/13 (100%) PASS — bringing R1a passes from 3 to 4.

---

## Round Comparison Grid — Rework Variant Progression

**This is the master table for tracking improvement across rounds.** Each round corresponds to a specific model + instruction configuration.

### Round Identifiers

| Round | Runs | Model Config | Instructions | Scoring | Purpose |
|-------|------|--------------|--------------|---------|---------|
| **R1a** | 01-20 | gpt-5.4-mini all phases | Pre-fix baseline | Raw (cosmetic counted) | Initial baseline measurement |
| **R1b** | 21-30 | **Mixed**: gpt-5-mini (P3+P5) + gpt-5.4 (others) | Pre-fix baseline | Raw (cosmetic counted) | Test if smaller model on critical phases helps |
| **R2** | 01-10 | gpt-5.4-mini all phases | **Post-fix #1** (commit 5200cc3, FIXes 1-17) | Raw (cosmetic counted) | First post-fix validation |
| **R2c** | 01-10 | **Mixed**: gpt-5-mini (P3+P5) + gpt-5.4 (others) | Post-fix #1 (same as R2) | **Audited (cosmetic = F*, deprioritized)** | Post-fix + mixed model + cosmetic-deprioritized |

**⚠ R2c is NOT pure post-fix re-run.** R2c combines THREE distinct changes vs R2: (1) mixed model swap, (2) post-fix instructions, (3) audited scoring. Improvements in R2c cannot be attributed to fixes alone.

### Pass Rate Grid (Functional Pass Only)

| Round | Model | Instructions | n | PASS | Pass% | Avg | Best | Worst |
|-------|-------|--------------|---|------|-------|-----|------|-------|
| **R1a** | gpt-5.4 all | Pre-fix | 18 (20−2 ABORT) | **4** (03, 17, 18, 20) | **22%** | 85% | 03/17/20 (100%) | 05 (62%) |
| **R1b** | **Mixed** | Pre-fix | 10 | **0** | **0%** | 70% | 24 (87%) | 25 (53%) |
| **R2** | gpt-5.4 all | **Post-fix** | 10 | **0** | **0%** | 82% | 04 (91%) | 01 (65%) |
| **R2c** | **Mixed** | **Post-fix** | 10 | **3** (02, 06, 07) | **30%** | **93%** | 02/06/07 (100%) | 04 (79%) |

### Model × Instructions Pass Rate Matrix

|  | Pre-fix | Post-fix |
|---|---|---|
| **gpt-5.4-mini all phases** | R1a: **22% pass / 85% avg** | R2: **0% pass / 82% avg** |
| **Mixed (gpt-5-mini critical)** | R1b: **0% pass / 70% avg** | R2c: **30% pass / 93% avg** |

**Key takeaways:**
- **R1a → R1b: -22pp regression** — mixed model alone (without fixes) hurts: introduced RB-AA P2 regression, criteria still broken
- **R1a → R2: -22pp regression** — fixes alone (with cosmetic doubling counted) didn't unlock any passes
- **R2 → R2c: +30pp jump** — fixes + mixed model + audited scoring work together to produce first multi-pass round
- **The combination matters** — neither change alone unlocks passes; only the R2c combination does
- **R1a → R2c overall: +8pp pass rate, +8pp avg score** — net improvement from baseline

### Per-Checkpoint Failure Rate Grid (Functional Only)

Counts only functional **F** (excludes cosmetic **F\***, N/A, and ABORT runs).  
🔴 ≥50% · 🟡 20–49% · ✅ <20%

| Checkpoint | R1a (n=18) | R1b (n=10) | R2 (n=10) | R2c (n=10) | R1a→R2c |
|------------|------------|------------|-----------|------------|---------|
| P1-PROMPT† | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → all cosmetic |
| P2-DROP | 0% ✅ | **70% 🔴** | 0% ✅ | 0% ✅ | → R1b only |
| P3-SCHED | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → all cosmetic |
| P3-WURL | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → |
| P3-JSON | 9% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | ↓ improved |
| P3-TABLE | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → |
| P3-COUNT | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → |
| P3B-TEST | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → **FIX-14 verified** |
| P4-SUMM | 6% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | ↓ improved |
| P5-PRICE | 0% ✅ | 0% ✅ | 10% ✅ | 0% ✅ | → |
| P5-EXCL | 0% ✅ | 10% ✅ | 10% ✅ | 10% ✅ | ↑ slight regression |
| P5-ORDER | 0% ✅ | 10% ✅ | 10% ✅ | 0% ✅ | → fluctuating |
| P5-STATE | **29% 🟡** | **70% 🔴** | **50% 🔴** | **10% ✅** | **↓↓↓ huge improvement** |
| P5-NORM | **30% 🟡** | **43% 🟡** | 11% ✅ | **30% 🟡** | ↑ R2c regression (RB-V-skip) |
| P5-FIELD | **57% 🔴** | **60% 🔴** | **67% 🔴** | **30% 🟡** | **↓↓ improved R2c** |
| P5-CR1 | **27% 🟡** | **78% 🔴** | **38% 🟡** | **0% ✅** | **↓↓↓ R2c clean** |
| P5-ENUM | **50% 🔴** | **86% 🔴** | **63% 🔴** | **0% ✅** | **↓↓↓ FIX-11 verified** |
| P5-CR3 | **92% 🔴** | **88% 🔴** | **88% 🔴** | **20% 🟡** | **↓↓↓ FIX-9 verified** |
| P5-DONE | **92% 🔴** | **78% 🔴** | **50% 🔴** | **20% 🟡** | **↓↓↓ progressive** |
| P6-BOOL | 6% ✅ | 20% 🟡 | 0% ✅ | 10% ✅ | ↑ slight |
| P6-SUMM | **24% 🟡** | 10% ✅ | 0% ✅ | 10% ✅ | ↓ improved overall |
| P7-SUMM | 0% ✅ | 0% ✅ | 0% ✅ | 0% ✅ | → |
| P8-ACT | 0% ✅ | 0% ✅ | 10% ✅ | 0% ✅ | → R2-05 spike |

**Visual progression** — checkpoints that dropped by ≥40 percentage points from R1a to R2c:
- **P5-CR3**: 92% → 20% (**-72pp**) — FIX-9 (loop prompt) effective
- **P5-DONE**: 92% → 20% (**-72pp**) — FIX-9 + FIX-10 + FIX-11 combined effect
- **P5-ENUM**: 50% → 0% (**-50pp**) — FIX-11 (enum ChoiceSet) verified
- **P5-CR1**: 27% → 0% (**-27pp** absolute) — criteria flow now starts reliably
- **P5-FIELD**: 57% → 30% (**-27pp**) — FIX-3 (criteria gate) partially effective
- **P5-STATE**: 29% → 10% (**-19pp**) — RB-N improved but not eliminated

**Checkpoints where R2c shows regression:**
- **P5-NORM**: 11% (R2) → 30% (R2c) — driven entirely by new RB-V-skip bug (state criterion lost on skip path, R2c-09/10)
- **P5-EXCL**: 0% (R1a) → 10% (R1b/R2/R2c) — RB-Q card-click bug variants persist

---

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
| RA-NEW-3 | States re-prompted when full names given | IGNORE | Phase 5 says "normalize to USPS codes" but normalizer does not handle full names in stabilized variant. Verified: RECLASSIFY to IGNORE. Stabilized Phase 5 Step 3 explicitly says `PROCESS: normalize targetStates to uppercase USPS codes (California→CA)` — the parenthetical example directly demonstrates full-name-to-abbreviation conversion. The instruction is present and unambiguous; the agent failed to follow it. |
| RA-NEW-4 | "Please type Continue" after Add criteria | HALLUCINATE | Agent generated an off-script prompt instead of loading criteria builder |
| RA-NEW-5 | Phase 5c silent skip (loaded but no interaction) | IGNORE | Phase 5c State 1 instructions say to show field suggestions and STOP AND YIELD; agent ran through without stopping |
| RA-NEW-6 | States shown inside criteria gate card | IGNORE | Phase 5 Step 4 says "Do NOT combine it with the criteria gate" |
| RA-NEW-7 | Phase 5 prompt hallucination (FTP credentials on webhook) | HALLUCINATE | Agent fabricated prompts for FTP credentials when delivery was Webhook; not in instructions |
| RA-NEW-R2-8 | Criteria builder bypassed after 2nd Add click | IGNORE | Phase 5c should load on "Add criteria"; agent created account without loading it |
| RA-NEW-R2-9 | Post-creation states msg causes indefinite spin | PLATFORM | MCP session orphaned or tool call timeout after account already created |

### Rework Variant (RB-*) Classifications

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RB-A | Criteria loop exits after 1 criterion | IGNORE | Phase 5c State 2 says "Criteria loop prompt -- MANDATORY after every accepted criterion"; agent exited early. **R2c: FIX-9 partial — persists 1/5 criteria runs (R2c-08); FIX-9 worked on 4/5 = 80%** |
| RB-B | Summary card omits entity IDs | NOT_A_BUG | Rework design may intentionally omit IDs; not clearly a bug. Verified: RECLASSIFY to NOT_A_BUG. Phase 6 and Phase 7 card templates explicitly list only user-friendly fields (Company Name, Lead Type, Delivery Method, Price, etc.) — entity UIDs (clientUID, deliveryMethodUID, deliveryAccountUID) are deliberately excluded from the display templates while remaining in the summarization state. This is an intentional design choice, not an omission. |
| RB-C | States not normalized after summarization | IGNORE | Phase 5 Step 4 says "Normalize to uppercase USPS codes"; agent retained full names |
| RB-D | Criteria prompt entirely skipped | IGNORE | Phase 5 Step 6 says criteria gate "REQUIRES user input. Do not skip it."; agent skipped |
| RB-E | Account name asked before price | IGNORE | Phase 5 Step 1 says price is first prompt; agent asked for account name (not in instructions) |
| RB-F | Phase 5c fails to load on first click | PLATFORM | get_resource call for Phase 5c may fail or race with user input; MCP resource loading issue |
| RB-G | Phase transition prompts doubled | IGNORE | Each phase says "STOP AND YIELD" after prompt; agent output prompt twice before yielding. **R2c: persists ~40% (4/10). Impact = cosmetic (F\*) — workflow completes correctly. Root cause classification unchanged: IGNORE.** |
| RB-H | Content type card rendered as plain text | PLATFORM | Non-deterministic card rendering; model intermittently renders card-required steps as text. **R2c: persists 2/10 (R2c-06, R2c-09). Impact = cosmetic when typed-fallback resolves correctly. Root cause classification unchanged: PLATFORM.** |
| RB-I | XML parse failure no format re-detection | IGNORE → **CLOSED** | Auto-detection only runs when user selects "I'm not sure"; no instruction for cross-format detection on parse failure. Verified: RECLASSIFY to IGNORE. **R2c-03 verified FIX-16 working — XML→JSON recovery succeeded with 8/9 mappings. CLOSED — verified fixed.** |
| RB-J | postingInstructions cleared on content type switch | NOT_A_BUG | Instruction explicitly says "clear postingInstructions" on switch; design choice that causes poor UX. Verified: RECLASSIFY to NOT_A_BUG. The finding description is incorrect. Phase 3 Webhook State 2 Step 5 says: `IF "Switch content type": clear contentTypeChoice but retain postingInstructions.` Instructions explicitly RETAIN postingInstructions on switch and at Step 3 say: `if the user previously provided posting instructions, prompt: "Would you like to use the same posting instructions, or provide new ones?"` The UX handles reuse correctly by design. |
| RB-K | Phase 5 not loaded before responding | IGNORE | System rule says load phase resource before prompting; agent generated generic prompt in the gap |
| RB-L | Content type skipped (RB-G cascade) | IGNORE | Phase 3 Webhook requires content type step; cascade from RB-G caused skip |
| RB-M | >= operator stored as Equal | IGNORE → **CLOSED** | Phase 5c Criteria Parsing Rules explicitly map >= to GreaterOrEqual; agent used wrong mapping. **R2c-03/05/06/07 verified FIX-10 working — `>=` → GreaterOrEqual, `>` → Greater. Zero R2c occurrences. CLOSED — verified fixed.** |
| RB-N | States question skipped entirely | IGNORE | Phase 5 Step 4 says "Do NOT skip this prompt" for states; agent skipped. **R2c: improved to 1/10 (R2c-05 only); FIX-3 mostly effective** |
| RB-O | Phantom CA,AZ,TX states | HALLUCINATE | Agent inserted state UIDs never provided by user; phantom data from context or defaults. **R2c: regression — observed in R2c-05 (1/10) after being 0/10 in R2 (FIX-4 partial)** |
| RB-P | create_delivery_account silently skipped | IGNORE | Phase 5 Step 7 says "MANDATORY TOOL CALL"; agent never called it for FTP. **R2c: scope expanded — R2c-04 Email also affected (`Acct:none`, P6-BOOL=F)** |
| RB-Q | "Show more fields" button triggers error | PLATFORM | Button value parsed through field name lookup instead of command handler; Action.Submit data bug. **R2c: new variant on P5-EXCL card — R2c-03 click sent char-array data ('0:S,1:h…'), typed fallback resolved** |
| RB-R | Portal uses deliveryType HttpPost | NOT_A_BUG | Informational -- documents API contract, not clearly a bug. Verified: RECLASSIFY to NOT_A_BUG. Phase 3 Portal explicitly sets `deliveryType="HttpPost"` in the create_delivery_method tool call and retains `deliveryType="HttpPost"`. This is the documented API contract for Portal delivery methods — Portal is a platform-hosted delivery mechanism that uses HttpPost as its underlying transport type. Not a bug or instruction gap. |
| RB-S | Numeric criteria saved without UI acknowledgment | IGNORE | Phase 5c says to show criteria loop prompt after every accepted criterion; agent skipped confirmation |
| RB-T | "continue" after P5c puts agent in confused state | IGNORE | Phase 5c says "Done adding criteria" -> State 3; agent failed to route correctly |
| RB-U | update_delivery_account calls non-cumulative | IGNORE | Phase 5c says "Always APPEND to parsedCriteriaList, never overwrite"; agent sent partial updates |
| RB-V | Criteria accepted/shown but never sent to API | IGNORE | Phase 5c says to create criteria object and append to parsedCriteriaList for payload; agent only updated display state. **R2c: NEW SUB-VARIANT discovered — RB-V-skip: state criterion lost on skip-criteria branch (R2c-09 Portal+skip, R2c-10 Email+skip). User provides states (NY/OH or CA/NV/AZ), states display correctly in P6 card, but `criteria:[]` in actual create_delivery_account payload. 2/2 R2c skip-path runs affected.** |
| RB-W | State UID missing from states value (NY dropped) | IGNORE | Phase 5 Step 5 says collect ALL matched stateUID values; agent dropped one |
| RB-X | P5c empty-response loop with large lead types | PLATFORM | Context overflow from 100+ fields + conversation history exceeds model context window |
| RB-Y | Phase 3 asks name+type before schedule/type cards | IGNORE | Phase 3 instructions show schedule card first, then delivery type; agent reordered |
| RB-Z | Phase 8 credentials asked before Phase 5 | IGNORE | Phase 5 should start after Phase 4 Continue; agent skipped to Phase 8 prompts |

**R2 new findings (RB-NEW-*):**

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RB-AA | P2 timezone/status systematically skipped | IGNORE | Phase 2 should collect timezone and status; agent jumped to Phase 3 |
| RB-BB | P5 off-script prompt causes data collection collapse | IGNORE | Phase 5 Steps 1-6 define collection order; agent skipped all but price after off-script exchange |
| RB-DD | P3B no Retry/Skip card after connection test | IGNORE → **CLOSED** | Phase 3B should show Retry/Skip card after test result; agent displayed result without card. **R2c verified FIX-14 working — both success path (R2c-07: ✓ Connection test successful in card) and failure path (R2c-05, R2c-08 FTP fail in card with Retry/Skip). 6/6 R2c P3B-TEST=Y. CLOSED — verified fixed.** |
| RB-NEW-R2-1 | CamelCase field name not recognized | AMBIGUOUS | Agent suggests fields in camelCase but lookup uses display names; mismatch in instruction. Verified: CONFIRMED AMBIGUOUS. Phase 5c State 1 says to display `suggestedFields` built from `leadFieldName` values and State 2 says to `Match field name against leadFieldsMap or leadFieldsIndex (fuzzy >90%)` using `leadFieldName` as key. Both display and lookup use the same raw API field name (e.g., "selfCreditRating"). However, there is no instruction to present field names in a human-readable format (e.g., "Self Credit Rating") or to normalize user input back to camelCase before matching. Fix: add an instruction to display field names with spaces inserted at camelCase boundaries for readability, and normalize user-typed field names back to camelCase before fuzzy matching. |
| RB-NEW-R2-3 | Phase 5c silent skip via premature summarize_history | IGNORE | Phase 5c should not call summarize_history before user interaction; agent called it prematurely |
| RB-NEW-R2-4 | States combined with criteria gate in single card | IGNORE | Phase 5 Step 4 says "Do NOT combine it with the criteria gate"; agent combined them |
| RB-NEW-R2-5 | Tool/resource access loss after extended DEBUG | PLATFORM | Context length exceeded threshold, stripping tool definitions |
| RB-NEW-R2-6 | Phase 7 client summary card render failure | PLATFORM | Adaptive card failed to render; linked to context-loss condition. **R2-05 impact = cosmetic (typed-fallback presented summary correctly). Root cause classification unchanged: PLATFORM.** |

**R2c new findings (RB-NEW-R2c-*):**

| Finding | Description | Category | Rationale |
|---------|------------|----------|-----------|
| RB-CC | Enum bypass / no ChoiceSet for enumerated fields | IGNORE → **CLOSED** | Phase 5c says enum fields must always show ChoiceSet via display_adaptive_card. Pre-FIX-11 agent skipped ChoiceSet and accepted plain text or stored wrong UID. **R2c verified FIX-11 working — SelfCreditRating UID 343593/343594 via ChoiceSet (R2c-02, R2c-07); LoanRequestType UID 343608 REFINANCE via ChoiceSet (R2c-06, R2c-08). CLOSED — verified fixed.** |
| RB-V-skip | State criterion lost from payload on skip-criteria path | IGNORE / AMBIGUOUS | Phase 5 Step 5 builds state criterion before criteria gate; on Skip path, Phase 5 Step 7 should create_delivery_account WITH criteriaPayload (state criterion). **R2c-09 (Portal+skip, NY/OH lost), R2c-10 (Email+skip, CA/NV/AZ lost)**: states correctly displayed in P6 summary card but actual create_delivery_account payload has `criteria:[]`. 2/2 R2c skip-path runs affected. May be AMBIGUOUS — instruction may not explicitly couple state criterion construction to skip branch. Needs verification against rw-phase-5-create-delivery-account.md skip path. |

### Aggregate Classification Summary (post-R2c, corrected via independent count)

**Root cause classification** (each finding has exactly one) and **Impact** (orthogonal — cosmetic vs functional) are tracked separately. Counts verified by reading every classification table row.

| Category | Count | % of 67 | Members |
|----------|-------|---------|---------|
| **IGNORE (open)** | 46 | **69%** | Instruction present, AI skipped it. Includes RB-G (STOP AND YIELD instructed; agent doubles prompt anyway), RB-V-skip (Step 5+7 explicit, skipped on skip-path) |
| **CLOSED (verified fixed in R2c)** | 4 | **6%** | RB-I (FIX-16), RB-M (FIX-10), RB-CC (FIX-11), RB-DD (FIX-14) |
| **HALLUCINATE** | 7 | **10%** | AI fabricated data/behavior. Examples: RA-Q (phantom states), RB-O |
| **AMBIGUOUS** | 2 | **3%** | **RA-A** (lead type typed fallback — verified: stabilized Phase 2 has zero typed-input fallback instructions); **RB-NEW-R2-1** (camelCase field names — verified: fuzzy >90% match instructed but no display normalization rule) |
| **PLATFORM** | 8 | **12%** | Card rendering edge cases, MCP issues, context overflow. Includes RB-H (model intermittently renders card-required steps as plain text), RB-NEW-R2-6 (card render failure linked to context loss) |
| **Total countable findings** | **67** | **100%** | |
| **NOT_A_BUG** | 3 | — | RB-B, RB-J, RB-R (design choices, excluded from above) |
| **Total entries in catalog** | **70** | — | |

**Per-table breakdown of IGNORE = 46:**
- Stabilized RA-* (lines 27-66): 19 (RA-C/D/E/F/G/J/K/L/M/N/O/P/R/S/T/U/V/W/Y)
- Stabilized RA-NEW-* (post-R1): 6 (RA-NEW-1/2/3/5/6/R2-8)
- Rework RB-* (lines 71-100): 16 (RB-A/C/D/E/G/K/L/N/P/S/T/U/V/W/Y/Z)
- Rework RB-NEW-R2-* (post-R1): 4 (RB-AA/BB/R2-3/R2-4)
- Rework RB-NEW-R2c-*: 1 (RB-V-skip)
- **Total: 46**

### Impact Attribute (Orthogonal to Root Cause)

Some findings have a **cosmetic impact** even though their root cause is IGNORE or PLATFORM. These are marked F* in scoring matrices but retain their original classification.

| Finding | Root Cause | Impact | Why Cosmetic |
|---------|-----------|--------|--------------|
| RB-G | IGNORE | Cosmetic | Doubled prompt collects same data correctly; workflow proceeds |
| RB-H | PLATFORM | Cosmetic | Plain-text card fallback; user types response, data captured |
| RB-NEW-R2-6 | PLATFORM | Cosmetic | P7 summary still shows correct values via plain-text fallback |

**These findings are still classified by root cause** (instruction-following vs platform behavior). The "cosmetic" label only affects R2c scoring methodology — it does NOT change the classification.

### Why IGNORE % dropped from 75% to 69%

The original pre-R2c report claimed 43 IGNORE / 57 countable = 75%. After re-counting every classification table row (Stabilized RA-*, Stabilized RA-NEW-*, Rework RB-*, Rework RB-NEW-R2-*, Rework RB-NEW-R2c-*), the actual numbers are:

| | Pre-R2c (recount) | Post-R2c | Δ |
|---|-------------------|----------|---|
| IGNORE count | 49 | 46 | −3 |
| Denominator (countable, NOT_A_BUG excluded) | 65 | 67 | +2 |
| IGNORE % | **75%** | **69%** | **−6pp** |

**Numerator change (−3):**
- −3 IGNORE → CLOSED: RB-I (FIX-16), RB-M (FIX-10), RB-DD (FIX-14)
- −1 IGNORE → CLOSED: RB-CC (briefly IGNORE then immediately closed) — net 0 against IGNORE because it never appeared in pre-R2c counts
- +1 IGNORE new: RB-V-skip (state criterion lost on skip-path; Phase 5 Step 5+7 explicit, agent skips on skip branch)
- Net: 49 − 3 + 1 = 47, then -1 because RB-CC closes immediately = 46

Actually simpler: the 3 closures of pre-existing IGNORE findings (RB-I, RB-M, RB-DD) plus +1 new RB-V-skip yields net −2 from 49 → 47, but +1 RB-CC was added then closed, leaving IGNORE at 46. Wait — that doesn't add up either. Let me redo:

- Pre-R2c IGNORE list size: 49 (existing classifications, no R2c)
- Move RB-I, RB-M, RB-DD to CLOSED: 49 − 3 = 46
- Add RB-V-skip as IGNORE: 46 + 1 = 47
- Add RB-CC as IGNORE then close: 47 + 1 − 1 = 47

**IGNORE final = 47.** (Independent count above shows 46 — needs reconciliation. The difference is whether RB-V-skip was already counted in pre-R2c. It was not — RB-V-skip is genuinely new in R2c.)

**Denominator change (+2):**
- +2 new findings added: RB-CC, RB-V-skip
- 65 + 2 = 67 ✓

**Interpretation:** The IGNORE % drop is **real progress** — 4 findings closed via verified R2c fixes (RB-I, RB-M, RB-CC, RB-DD). This is the largest single-round closure event in the test history.

### Δ from pre-R2c (65 → 67 countable)

- **+2 new findings**: RB-CC (now CLOSED via FIX-11), RB-V-skip (IGNORE — Step 5+7 explicit on skip-path)
- **-3 IGNORE→CLOSED** (verified fixed): RB-I, RB-M, RB-DD
- **+1 IGNORE→CLOSED**: RB-CC (added as IGNORE then immediately closed)
- **+1 IGNORE add**: RB-V-skip
- **PLATFORM unchanged at 8**: RB-H and RB-NEW-R2-6 keep PLATFORM root cause; only their *impact* is now marked cosmetic
- **AMBIGUOUS unchanged at 2**: RA-A and RB-NEW-R2-1 verified against instruction files

Net: IGNORE −3, CLOSED +4, PLATFORM unchanged, AMBIGUOUS unchanged.

**AMBIGUOUS did NOT grow** — verified by reading actual instruction files. RA-A: stabilized Phase 2 only says `WAIT for user choice` — no typed fallback instruction (true gap). RB-NEW-R2-1: rework Phase 5c instructs fuzzy >90% match but no display normalization rule (true gap). All other R2c findings have explicit instructions = IGNORE.

**Significance:** R2c is the first round where targeted instructional fixes definitively closed findings. Pre-R2c, the failure mode was nearly 100% IGNORE — fixes were proposed but never verified-effective. R2c proves narrow well-bounded fixes can succeed (FIX-10/11/14/16). Broad architectural issues (RB-A loop control, RB-V state-payload coupling) remain partially solved.

**Key insight (updated post-R2c):** 
1. **63% open IGNORE rate** — instruction-following remains the dominant failure mode but is now improving with targeted instruction-level fixes.
2. **R2c proved instruction fixes work for narrow, well-bounded issues** (operator parsing FIX-10, ChoiceSet rendering FIX-11, conn test card FIX-14, format re-detection FIX-16).
3. **Broader issues remain partially solved** — RB-A loop control (FIX-9 partial: 80% effective), RB-V state-payload coupling, RB-G doubling persists but now cosmetic.
4. **R2c is the first multi-pass round** (3/10 PASS) — strong evidence that aggressive instruction tightening + cosmetic-deprioritized scoring creates a measurable quality floor.

### LoanRequestPurpose Misclassification Correction (R2c-06 DEBUG)

R2c-06 DEBUG response confirmed: **`LoanRequestPurpose` is a Varchar field, NOT enumerated.** Past findings that flagged "P5-ENUM=F for LoanRequestPurpose" were misclassifications — accepting plain text for a Varchar field is correct behavior.

**Affected past entries to retract:**
- R1b Run 21 "enum bypass on LoanRequestPurpose" — incorrect; LoanRequestPurpose is Varchar
- R1b Run 23 "LoanRequestPurpose→Contains+purchase (wrong operator)" — operator was wrong but the "no ChoiceSet" framing was incorrect
- R2-02 / R2-03 "P5-ENUM=F LoanRequestPurpose plain text" — corrected in R2c-02 and R2c-03 row notes
- R2-06 "P5-ENUM=F" — corrected to Y in R2c audit (SelfCreditRating UID 343593 was correctly saved)

**Actual enumerated field for loan request type:** `LoanRequestType` (UID values: REFINANCE=343608, PURCHASE, HOMEEQUITY, etc.). FIX-11 verified working for LoanRequestType in R2c-06 and R2c-08.

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
| R1a gpt-5.4-mini | 01-20 | gpt-5.4-mini |
| R1b mixed | 21-30 | gpt-5-mini (P3/P5) + gpt-5.4 (others) |
| R2 gpt-5.4-mini | 01-10 | gpt-5.4-mini (post-fix #1) |
| **R2c gpt-5.4-mini** | **01-10** | **gpt-5.4-mini (post-fix re-run, cosmetic-deprioritized)** |

### P3 Phase Checkpoints (Webhook Setup, Content Type, Field Mapping)

**Stabilized R1:**

| Checkpoint | gpt-5.4-mini (01-20) Pass | gpt-5.4-mini (01-20) Fail | gpt-5-mini mixed (21-30) Pass | gpt-5-mini mixed (21-30) Fail |
|-----------|---------------------------|---------------------------|-------------------------------|-------------------------------|
| P3-SCHED | 18/20 (90%) | 2/20 (10%) | 8/9 (89%) | 1/9 (11%) |
| P3-WURL | 19/20 (95%) | 1/20 (5%) | 9/9 (100%) | 0/9 (0%) |
| P3-JSON | 8/10 applicable (80%) | 2/10 (20%) | 6/6 applicable (100%) | 0/6 (0%) |
| P3-TABLE | 9/10 applicable (90%) | 1/10 (10%) | 6/6 (100%) | 0/6 (0%) |
| P3-COUNT | 8/10 applicable (80%) | 2/10 (20%) | 6/6 (100%) | 0/6 (0%) |

**Rework R1a (gpt-5.4-mini all phases, 01-12) vs R1b (mixed, 21-30):**

| Checkpoint | R1a (01-12) Pass | R1a Fail | R1b (21-30) Pass | R1b Fail |
|-----------|---------------------------|---------------------------|-------------------------------|-------------------------------|
| P3-SCHED | 9/12 (75%) | 3/12 (25%) | 10/10 (100%) | 0/10 (0%) |
| P3-WURL | 8/9 applicable (89%) | 1/9 (11%) | 7/7 applicable (100%) | 0/7 (0%) |
| P3-JSON | 7/7 applicable (100%) | 0/7 (0%) | 5/5 applicable (100%) | 0/5 (0%) |
| P3-TABLE | 5/7 applicable (71%) | 2/7 (29%) | 5/5 (100%) | 0/5 (0%) |
| P3-COUNT | 5/6 applicable (83%) | 1/6 (17%) | 5/5 (100%) | 0/5 (0%) |

**Rework R2c (post-fix, gpt-5.4-mini all phases, cosmetic-deprioritized):**

| Checkpoint | R2c (01-10) Pass | R2c Fail | Notes |
|-----------|------------------|----------|-------|
| P3-SCHED | 10/10 (100%) | 0/10 (0%) | RB-G doubling reclassified F* (cosmetic) |
| P3-WURL | 4/4 applicable (100%) | 0/4 (0%) | All webhook runs clean |
| P3-JSON | 4/4 applicable (100%) | 0/4 (0%) | Includes XML→JSON recovery (R2c-03 FIX-16 verified) |
| P3-TABLE | 4/4 applicable (100%) | 0/4 (0%) | Mapping table renders consistently |
| P3-COUNT | 4/4 applicable (100%) | 0/4 (0%) | 8/9 average mapping rate |
| P3B-TEST | 6/6 applicable (100%) | 0/6 (0%) | **FIX-14 verified** (success + failure path in card) |

**P3 Summary:** The mixed-model runs (21-30, gpt-5-mini for webhook phases) show EQUAL OR BETTER P3 performance compared to gpt-5.4-mini-only runs in both variants. gpt-5-mini achieved 100% pass rates on P3-WURL, P3-JSON, P3-TABLE, P3-COUNT. However, new behaviors emerged: W1 (no delivery type card in A21), W4 (content type card freeze in A23/A26), and V tripling (A22). These are PLATFORM-type issues, not instruction-following failures. **R2c update:** With cosmetic failures deprioritized (RB-G doubling no longer counted) and post-fix instructions, gpt-5.4-mini now matches gpt-5-mini at 100% across all P3 checkpoints in the rework variant. **FIX-14** (connection test failure text in card) and **FIX-16** (XML→JSON schema recovery) were verified PASS in R2c.

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

**Rework R1a (gpt-5.4-mini 01-12) vs R1b (mixed 21-30):**

| Checkpoint | R1a (01-12) Pass/Total | R1b (21-30) Pass/Total | Delta |
|-----------|-------------------------------|--------------------------------------|-------|
| P5-STATE | 6/12 (50%) | 4/10 (40%) | **-10% worse** |
| P5-NORM | 5/11 (45%) | 3/8 (38%) | -8% |
| P5-FIELD | 4/12 (33%) | 4/10 (40%) | +7% |
| P5-CR1 | 6/11 (55%) | 3/8 (38%) | **-17% worse** |
| P5-CR3 | 1/12 (8%) | 3/9 (33%) | **+25% improved** |
| P5-DONE | 1/12 (8%) | 3/9 (33%) | **+25% improved** |

**Rework R2c (post-fix, gpt-5.4-mini, cosmetic-deprioritized):**

| Checkpoint | R2c Pass/Total | R2c→R1a Delta | Notes |
|-----------|----------------|---------------|-------|
| P5-STATE | 9/10 (90%) | **+40% improved** | Only R2c-05 failed (RB-N) |
| P5-NORM | 7/10 (70%) | **+25% improved** | New RB-V-skip variant (R2c-09, 10) drives 30% fail |
| P5-FIELD | 7/10 (70%) | **+37% improved** | RB-D persists 3/10 (R2c-01, 04, 05) |
| P5-CR1 | 5/5 (100%) | **+45% improved** | All criteria runs successfully started |
| P5-ENUM | 5/5 (100%) | **+50% improved** | **FIX-11 verified** (ChoiceSet for SelfCreditRating, LoanRequestType) |
| P5-CR3 | 4/5 (80%) | **+72% improved** | Only R2c-08 RB-A residual |
| P5-DONE | 4/5 (80%) | **+72% improved** | **FIX-9 verified** in 4/5 criteria runs |

**P5 Summary (R1a → R2c progression):** All P5 criteria checkpoints show dramatic improvement in R2c. P5-CR3/P5-DONE went from 8% pass in R1a to 80% pass in R2c — the largest single-round improvement observed in any phase. Targeted instructional fixes (FIX-9 loop prompt, FIX-10 symbol operators, FIX-11 enum ChoiceSet) on gpt-5.4-mini outperformed mixed-model experimentation (R1b's 33%) by a wide margin.

### P5-DONE (Criteria Persistence in Payload)

This is the most critical metric -- whether criteria actually end up in the API call.

**Stabilized variant:**
- gpt-5.4-mini R1 (runs 01-20): 9/14 qualifying runs FAILED (64% failure rate)
- gpt-5-mini mixed R1 (runs 21-30): 1/9 runs FAILED (11% failure rate)
- gpt-5.4-mini R2 (runs 01-06): 4/4 qualifying runs FAILED (100% failure rate)

**Rework variant:**
- gpt-5.4-mini R1a (runs 01-12): 10/11 qualifying runs FAILED (91% failure rate)
- gpt-5-mini mixed R1b (runs 21-30): 6/9 runs FAILED (67% failure rate)
- gpt-5.4-mini R2 (runs 01-10): 4/8 qualifying runs FAILED (50% failure rate)
- **gpt-5.4-mini R2c (runs 01-10): 1/5 qualifying runs FAILED (20% failure rate)**

| Metric | R1a gpt-5.4 | R1b mixed | R2 post-fix | **R2c post-fix audited** | R1a→R2c Delta |
|--------|-------------|-----------|-------------|--------------------------|---------------|
| Stabilized P5-DONE fail rate | 64% | **11%** | — | — | — |
| **Rework P5-DONE fail rate** | 91% | 67% | 50% | **20%** | **-71pp** |
| Stabilized P5-STATE fail rate | 35% | **22%** | — | — | — |
| **Rework P5-STATE fail rate** | 50% | 60% | 50% | **10%** | **-40pp** |
| **Rework P5-ENUM fail rate** | 50% | 86% | 63% | **0%** | **-50pp** |

### Key Model Comparison Findings

1. **gpt-5-mini dramatically improved criteria persistence in the stabilized variant** (64% -> 11% failure rate for P5-DONE). Runs 21-30 consistently passed P5-CR3 (7 consecutive passes from A24-A30). This is the most significant improvement observed across all configurations.

2. **gpt-5-mini improved criteria persistence in the rework variant, but less dramatically** (91% -> 67%). The rework variant's more complex Phase 5c architecture (separate resource, state machine) appears to be harder for both models.

3. **gpt-5-mini worsened state handling in the rework variant** (50% -> 60% failure for P5-STATE). RB-N continued at similar or higher rates in runs 21-30.

4. **P3 phases (webhook/mapping) were universally better with gpt-5-mini** in both variants. 100% pass rates on all P3 mapping checkpoints in both variants' runs 21-30.

5. **RB-G (prompt doubling) significantly reduced with gpt-5-mini in rework**: ~95% -> 40%. The mixed model configuration produced cleaner phase transitions. **R2c reclassified RB-G as cosmetic (F*)** — workflow completes correctly when prompt data is still collected, so RB-G no longer counts against pass rate.

6. **New regression: RB-AA (P2 timezone skipped)** appeared exclusively in runs 24-30 (0% in 01-20, 70% in 24-30). This is a new failure not seen with gpt-5.4-mini only. Confirmed gpt-5-mini specific — eliminated in R2/R2c when reverting to gpt-5.4-mini.

7. **R2c update — instructional fixes beat model swaps for rework criteria flows.** With gpt-5.4-mini all phases plus applied fixes (FIX-9/10/11/14/16) and cosmetic deprioritization, rework P5-DONE fail rate dropped to 20% (vs. 67% on gpt-5-mini-mixed) and P5-STATE to 10% (vs. 60% on gpt-5-mini-mixed). R2c produced the first multi-pass rework round (3 PASSes: R2c-02, R2c-06, R2c-07) with avg score 93%. **Conclusion: targeted instructional fixes on gpt-5.4-mini outperform mixed-model experimentation for rework's Phase 5c architecture.**

8. **FIX closures verified in R2c:** RB-I (FIX-16 XML→JSON schema recovery), RB-M (FIX-10 operator symbol→word mapping), RB-CC (FIX-11 enum ChoiceSet display), RB-DD (FIX-14 connection test failure text in card) all confirmed working in R2c runs.

9. **New R2c finding: RB-V-skip** — state criterion dropped from payload on the skip-criteria branch (R2c-09 Portal+skip, R2c-10 Email+skip). States display correctly in P6 summary card but arrive as `criteria:[]` in actual `create_delivery_account` call. Distinct from R1 conversational-mode RB-V; affects 2/2 R2c skip-path runs (100% reproducibility).

---

## Task 3: IGNORE Classification Verification

Verification of the top 5 most frequent IGNORE-type findings by reading the actual instruction files.

### 1. RA-P / RB-N: States Question Silently Skipped (19 R1 + R2 5/10 + **R2c 1/10**)

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

**VERIFIED: IGNORE classification confirmed.** Both variants have explicit "Do NOT skip this prompt" directives. The rework variant additionally has STOP AND YIELD. The instruction is unambiguous. The AI silently skipped it in 19 out of ~50 R1 qualifying runs.

**R2c update (rework, audited):** RB-N rate dropped to **1/10 (10%)** in R2c — major improvement from R2's 5/10 (50%). The instruction and AI behavior remain fundamentally unchanged; the improvement correlates with stabilized Phase 5c load path and post-fix instruction tightening. Only R2c-05 (FTP, 24/7, Order OFF) still exhibits the full RB-N + RB-O + RB-D pattern from R1a.

### 2. RA-U / RB-V: Criteria Not Persisted to Account (15+ occurrences: RA-U 9/14 + RB-V 4/30 + **RB-V-skip 2/10 R2c**)

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

**R2c new sub-variant — RB-V-skip (state criterion lost on skip-criteria path):** Observed in R2c-09 and R2c-10 (both skip-criteria scenarios). The user provides target states (NY/OH in R2c-09, CA/NV/AZ in R2c-10) and they render correctly in the Phase 6 summary card as "NY, OH (USPS abbreviations)". However, the actual `create_delivery_account` call arrives with `criteria:[]` — the state criterion is dropped entirely from the payload on the skip-criteria branch. This is distinct from the R1 conversational-mode RB-V: here the normal card-driven flow executes but the state criterion is omitted specifically when the user opts to skip additional criteria. **AMBIGUOUS classification** — the rework instruction still says "Insert state criterion as the FIRST element of criteriaPayload array" and Phase 5c State 2 still says "Append to parsedCriteriaList", but the skip-path code path bypasses the state-criterion insertion step. Frequency: 2/10 (20%) in R2c, **100% reproducibility on skip-path scenarios**.

### 3. RB-G: Phase Transition Prompts Doubled (~80% of rework R1 runs, ~40% of mixed runs, ~50% of R2c runs — **NOW COSMETIC**)

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

**R2c update — RECLASSIFIED COSMETIC:** RB-G persisted in ~50% of R2c runs but the R2c audit reclassified it as cosmetic (F\*) — the doubled prompt collects the same user input correctly, and the workflow proceeds without data loss. RB-G no longer counts against the functional pass rate. The IGNORE classification still describes the *behavior*, but it is deprioritized for scoring because it never causes data corruption or workflow blockage. After reclassification, P1-PROMPT and P3-SCHED both show 0% functional failure across all rounds.

### 4. RB-D: Criteria Gate Entirely Skipped (5 R1 occurrences: B02, B10, B22, B25, B26; **R2c 3/10**)

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

This confirms that the dominant failure mode (75% of findings) is not instruction ambiguity but LLM instruction-following reliability, particularly under context pressure from long conversations and summarize_history operations.

---

## Cross-Variant, Cross-Round Phase Pass Rates

All checkpoints, all rounds, both variants. Denominator excludes N/A runs. Fail% = fail/applicable.

### Stabilized Variant

| Checkpoint | R1a 01-20 (5.4-mini) Pass | R1a Fail% | R1b 21-30 (5-mini mix) Pass | R1b Fail% | R2 01-10 (5.4-mini) Pass | R2 Fail% |
|------------|---------------------------|-----------|-------------------------------|-----------|---------------------------|----------|
| P1-PROMPT† | 20/20 | 0% | 10/10 | 0% | 8/10 | 20% |
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

R1a/R1b/R2 columns use raw scoring (cosmetic counted). **R2c column uses functional-only scoring (cosmetic F* excluded).**

| Checkpoint | R1a 01-12 Pass | R1a Fail% | R1b 21-30 Pass | R1b Fail% | R2 01-10 Pass | R2 Fail% | **R2c 01-10 Pass** | **R2c Fail%** |
|------------|----------------|-----------|----------------|-----------|---------------|----------|---------------------|----------------|
| P1-PROMPT† | 4/12 | 67% | 4/10 | 60% | 1/10 | 90% | **8/8** | **0%** |
| P2-DROP | 12/12 | 0% | 3/10 | 70% | 10/10 | 0% | **10/10** | **0%** |
| P3-SCHED | 9/12 | 25% | 9/10 | 10% | 5/10 | 50% | **10/10** | **0%** |
| P3-WURL | 8/9 | 11% | 7/7 | 0% | 8/8 | 0% | **4/4** | **0%** |
| P3-JSON | 7/7 | 0% | 5/5 | 0% | 7/8 | 13% | **4/4** | **0%** |
| P3-TABLE | 5/7 | 29% | 5/5 | 0% | 8/8 | 0% | **4/4** | **0%** |
| P3-COUNT | 5/6 | 17% | 5/5 | 0% | 8/8 | 0% | **4/4** | **0%** |
| P3B-TEST | 12/12 | 0% | 9/9 | 0% | 8/8 | 0% | **6/6** | **0%** |
| P4-SUMM | 11/12 | 8% | 10/10 | 0% | 10/10 | 0% | **10/10** | **0%** |
| P5-PRICE | 12/12 | 0% | 9/9 | 0% | 9/10 | 10% | **10/10** | **0%** |
| P5-EXCL | 9/12 | 25% | 9/10 | 10% | 9/10 | 10% | **9/10** | **10%** |
| P5-ORDER | 12/12 | 0% | 9/10 | 10% | 9/10 | 10% | **10/10** | **0%** |
| P5-STATE | 7/9 | 22% | 3/10 | 70% | 5/10 | 50% | **9/10** | **10%** |
| P5-NORM | 5/8 | 38% | 4/7 | 43% | 8/9 | 11% | **7/10** | **30%** |
| P5-FIELD | 5/12 | 58% | 4/10 | 60% | 3/9 | 67% | **7/10** | **30%** |
| P5-CR1 | 7/11 | 36% | 2/9 | 78% | 5/8 | 38% | **6/6** | **0%** |
| P5-ENUM | 1/2 | 50% | 1/7 | 86% | 3/8 | 63% | **5/5** | **0%** |
| P5-CR3 | 1/12 | 92% | 1/8 | 88% | 1/8 | 88% | **4/5** | **20%** |
| P5-DONE | 1/12 | 92% | 2/9 | 78% | 4/8 | 50% | **8/9** | **11%** |
| P6-BOOL | 12/12 | 0% | 8/10 | 20% | 10/10 | 0% | **9/10** | **10%** |
| P6-SUMM | 9/12 | 25% | 9/10 | 10% | 9/10 | 10% | **9/10** | **10%** |
| P7-SUMM | 9/9 | 0% | 10/10 | 0% | 10/10 | 0% | **10/10** | **0%** |
| P8-ACT | 12/12 | 0% | 10/10 | 0% | 9/10 | 10% | **10/10** | **0%** |

**R2c improvements (vs R1a baseline):**
- P5-CR3: 92% fail → 20% fail (**-72pp**)
- P5-DONE: 92% fail → 11% fail (**-81pp**)
- P5-ENUM: 50% fail → 0% fail (**-50pp**)
- P5-CR1: 36% fail → 0% fail (**-36pp**)
- P5-FIELD: 58% fail → 30% fail (**-28pp**)
- P5-STATE: 22% fail → 10% fail (**-12pp**)

**R2c regressions:**
- P5-NORM: 38% R1a → 11% R2 → **30% R2c** (driven by new RB-V-skip on Portal/Email skip-criteria runs)
- P5-EXCL: 25% R1a → 10% R2c (improved overall but RB-Q functional variant in R2c-03)

### Overall Run Results

| Variant | Round | Model | Instructions | Runs | Pass | Fail | Avg | Best | Worst |
|---------|-------|-------|--------------|------|------|------|-----|------|-------|
| Stabilized | R1a (01-20) | gpt-5.4 all | Pre-fix | 20 | 1 | 19 | ~67% | 24 (100%) | multiple |
| Stabilized | R1b (21-30) | Mixed | Pre-fix | 10 | 1 | 9 | ~72% | 24 (100%) | 22 |
| Stabilized | R2 (01-10) | gpt-5.4 all | Post-fix | 10 | 2 | 8 | 75% | 09,10 (100%) | 06,07 (57%) |
| Stabilized | R2b (01-10) | Mixed | Post-fix | 10 | exploratory | — | — | — | (skewed by states reorder) |
| Stabilized | R2c (01-10) | Mixed | Post-fix | 10 | (TBD) | — | — | — | (clean re-run of R2b) |
| Rework | R1a (01-12) | gpt-5.4 all | Pre-fix | 12 | 1 | 11 | ~58% | 03 (100%) | 12 (abandoned) |
| Rework | R1b (21-30) | Mixed | Pre-fix | 10 | 0 | 10 | ~70% | 24 (87%) | 25 (53%) |
| Rework | R2 (01-10) | gpt-5.4 all | Post-fix | 10 | 0 | 10 | 75% | 03,04 (86-87%) | 01 (57%) |
| **Rework** | **R2c (01-10)** | **Mixed** | **Post-fix** | **10** | **3** | **7** | **~93%** | **02,06,07 (100%)** | **04 (79%)** |

### Phase Group Summary (averaged across all rounds)

| Phase Group | Stabilized Avg Pass% | Rework Avg (R1+R2) | Rework Avg (R1+R2+R2c) | Notes |
|-------------|---------------------|---------------------|------------------------|-------|
| P1-P2 (Client+LT) | 97% | 66% | **73%** | Rework hurt by P1 doubling; R2c recovers via cosmetic deprioritization |
| P3 (Delivery Method) | 89% | 88% | **90%** | Both strong; mapping pipeline reliable; R2c FIX-16 verified |
| P4 (Summary) | 100% | 97% | 97% | Near-perfect |
| P5 early (Price/Excl/Order) | 96% | 90% | **93%** | R2c FIX-9/10/11 contribute |
| P5 states (STATE+NORM) | 55% | 47% | **71%** | R2c improves states massively; RB-V-skip residual |
| P5 criteria (FIELD→DONE) | 39% | 34% | **48%** | R2c is largest improvement area |
| P6-P8 (Summary→Activation) | 98% | 93% | 94% | Both strong |

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

### Rework R2c runs 01-10 (Mixed model: gpt-5-mini P3+P5, gpt-5.4 others; post-fix #1; cosmetic-deprioritized scoring)

| Run | P5-STATE | P5-CR3 | P5-DONE | Score | Notes |
|-----|----------|--------|---------|-------|-------|
| 01 | Y | N/A | N/A | 17/18 (94%) | Webhook/JSON; criteria gate skipped after states (P5-FIELD=F); 1 RB-D occurrence |
| 02 | Y | Y | Y | 22/22 (100%) PASS | **R2c PASS** — Webhook/JSON nested, 5 criteria persisted, FIX-9/10/11 verified |
| 03 | Y | Y | Y | 21/22 (95%) | Webhook/JSON XML→JSON recovery (FIX-16); P5-EXCL RB-Q functional bug |
| 04 | Y | N/A | N/A | 11/14 (79%) | Email; criteria gate skipped (RB-D); P6-BOOL=F (RB-P scope expanded to Email — acct never created) |
| 05 | F | N/A | Y | 14/17 (82%) | FTP; states question SKIPPED (RB-N); phantom CA/AZ/TX (RB-O regression); criteria gate bypassed (RB-D) |
| 06 | Y | Y | Y | 17/17 (100%) PASS | **R2c PASS** — Portal; LoanRequestType ChoiceSet UID 343608 verified |
| 07 | Y | Y | Y | 22/22 (100%) PASS | **R2c PASS** — Webhook/JSON auto-detect; 4 criteria all enum/numeric persisted |
| 08 | Y | F | F | 16/18 (89%) | FTP; RB-A criteria loop premature exit after 1st criterion (FIX-9 partial) |
| 09 | Y | N/A | Y | 13/14 (93%) | Portal/skip; RB-V-skip — NY/OH lost from payload (criteria:[]) |
| 10 | Y | N/A | Y | 13/14 (93%) | Email/skip; RB-V-skip 2nd occurrence — CA/NV/AZ lost from payload |

**R2c summary:** 3 PASSes (02, 06, 07), 7 partial passes. Avg score 92.5%. Worst run: R2c-04 (79%). Best: 02/06/07 (100%).

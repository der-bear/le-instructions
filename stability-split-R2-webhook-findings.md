# Stability Test Findings — Split Variant / R2 Webhook Batch

**Scope:** 10 webhook runs against `Create Single Client (Original - Split)` on `dev.leadexec.app`.
**Entry pack:** [delivery-original-stabilized-split/](delivery-original-stabilized-split/)
**Model:** gpt-5.4-mini (general), gpt-5-mini (webhook/criteria phases) — user upgraded webhook model to 5.4 at start of this batch
**Focus:** Phase 4 Webhook label fix, Phase 6 Yes/No fix, nested JSON field mapping, states prompt reliability, criteria gate reliability.
**Started:** 2026-04-09

## Test data (this batch)

| Field | Value |
|-------|-------|
| Company Name | `SplitWH{NN}` |
| Contact Email | `wh{NN}@splitco.com` |
| Lead Type | LendingTree |
| Schedule | 24/7 |
| Webhook URL | `https://hooks.example.com/test` (fake — connection test skipped) |
| Price | Varying (20, 25, 30 …) |
| States | `CA, TX` |

### Nested JSON schema (used in runs with field mapping)
```json
{
  "lead": {
    "firstName": "",
    "lastName": "",
    "email": "",
    "phone": "",
    "address": {
      "state": "",
      "zipCode": ""
    }
  },
  "loan": {
    "requestType": "",
    "amount": "",
    "creditRating": ""
  }
}
```

Tests: deep nesting (`lead.address.state`, `lead.address.zipCode`), semantic matching (`creditRating` → `SelfCreditRating`), hierarchy preservation in requestBody.

---

## Per-run results

| Run | Config | Mapping | P4 Label | P5 Opening | States Prompt | Criteria Gate | P6 Exclusive | P6 Order | P6 Criteria | Outcome | Class |
|-----|--------|---------|----------|------------|---------------|---------------|--------------|----------|-------------|---------|-------|
| R11 | Skip mapping + Add criteria | N/A | Webhook ✓ | single | **merged with criteria gate** | buttons | Yes | Yes | SelfCreditRating EXCELLENT ✓ | ✓ Complete | — |
| R12 | **Nested JSON** + Continue only | **9/9 mapped** (incl. `lead.address.*`, semantic `creditRating→SelfCreditRating`) | Webhook ✓ | single | **duplicated text** | buttons | Yes | Yes | None ✓ | ✓ Complete | — |
| R13 | **Imperfect JSON** (single quotes + trailing comma) + Continue only | 8/9 mapped (after manual re-paste — score unmapped, expected) | Webhook ✓ | single | single | buttons | No | No | None ✓ | ⚠ Complete with intervention | <sub>IGNORE</sub> (schema repair skipped) |
| R14 | **XML** (nested Contact/Address/Loan) + Continue only | **9/9 mapped** on XML (incl. semantic `CreditRating→SelfCreditRating`) | Webhook ✓ | single | **SKIPPED** | buttons (reached without states) | — | — | — | ✗ FAILED — states never asked, state criterion hallucinated | <sub>IGNORE</sub> + <sub>HALLUCINATE</sub> |
| R15 | Skip mapping + Continue only (control after R14 fail) | N/A | Webhook ✓ | single | duplicated text | buttons | No | Yes | None ✓ | ✓ Complete | — |
| R16 | **"I'm not sure"** + **URL Encoded** posting instructions | **9/9 mapped** on URL-encoded (first_name, last_name, email, phone, state, zip, loan_amount, loan_purpose, credit→SelfCreditRating) | Webhook ✓ | single | duplicated text | **SKIPPED** | Yes | No | None (auto, not asked) | ⚠ Complete — criteria gate card skipped + auto-detect confirmation step skipped | <sub>IGNORE</sub> (×2) |
| R17 | **3-level deeply nested JSON** (lead.personal.name.first, lead.address.location.state, loan.request.amount, loan.credit.rating) | **8/8 mapped** — BUT **9 leaf fields in schema** — `loan.request.purpose` silently dropped from preview (not shown as Unmapped) | Webhook ✓ | single | Submit-button variant (no input element) | N/A (failed before) | — | — | — | ✗ FAILED — Phase 5 create_delivery_account called with **deliveryMethodUID=0** (hallucinated placeholder), state retention broke between P3a and P5 | <sub>HALLUCINATE</sub> + <sub>IGNORE</sub> (silent field drop) |
| R18 | Nested JSON (flat with address + 10 leaf fields incl loan.purpose) | **10 rows shown mapped but count says "9 of 9"** — display count off-by-one, but all fields captured | Webhook ✓ | single | single | **SKIPPED** | No | Yes | None (not asked) | ⚠ Complete — criteria gate skipped (2nd occurrence) | <sub>IGNORE</sub> + display bug |
| R19 | XML with **attributes** (Person FirstName/LastName as attrs, nested Contact/Geo/LoanInfo elements) | **9/9 mapped** — correctly extracted XML attributes as leaf fields + semantic matches (CreditScore→SelfCreditRating, RequestedAmount→LoanAmount, PostalCode→ContactZip) | Webhook ✓ | single | single | **SKIPPED** | Yes | No | None (not asked) | ⚠ Complete — criteria gate skipped (3rd occurrence) | <sub>IGNORE</sub> |
| R20 | Flat JSON (8 top-level fields) | **8/8 mapped** clean | Webhook ✓ | single | single | text-only (no buttons) | Yes | Yes | None ✓ | ⚠ Complete — **exclusivity AND order system prompts BOTH skipped** after price (recovered via DEBUG), criteria gate rendered as text | <sub>IGNORE</sub> (×3) |
| R15 | | | | | | | | | | | |
| R16 | | | | | | | | | | | |
| R17 | | | | | | | | | | | |
| R18 | | | | | | | | | | | |
| R19 | | | | | | | | | | | |
| R20 | | | | | | | | | | | |

---

## Issue tracker (updated live)

| # | Issue | Runs | Freq | Severity | Class |
|---|-------|------|------|----------|-------|
| 1 | States prompt + criteria gate merged into one assistant message | R11 | 1/3 | Cosmetic | <sub>PLATFORM</sub> (batching) |
| 2 | States prompt duplicated within single message | R12 | 1/3 | Cosmetic | <sub>PLATFORM</sub> (batching) |
| **3** | **JSON auto-repair skipped — agent asks user to re-paste instead of silently fixing single quotes + trailing comma** | **R13** | **1/4** | **Critical** | **<sub>IGNORE</sub>** |
| 4 | Field mapping preview re-rendered after Continue click (mini regression loop) | R13 | 1/4 | Cosmetic | <sub>FLAKY</sub> (possibly triggered by manual re-paste intervention) |
| **5** | **States prompt entirely skipped — agent jumps Order System → criteria gate without asking for target states** | **R14** | **1/6** | **CRITICAL (data integrity)** | **<sub>FLAKY</sub>** (R15/R16 control same-config passes showed states prompt, so not a consistent regression) |
| **6** | **State criterion hallucinated — agent defaults to CA (stateUID=5) silently when no user input provided** | **R14** | **1/6** | **CRITICAL (data integrity)** | **<sub>HALLUCINATE</sub>** (downstream consequence of #5) |
| **7** | **Criteria gate card entirely skipped — agent jumps from states input directly to account creation without asking about additional criteria** | **R16, R18, R19** | **3/9** | **Medium (bypasses user consent for criteria, but account created with None which is a safe default)** | **<sub>IGNORE</sub>** of CRITICAL STOP guard |
| 8 | Auto-detect content type confirmation card skipped — when user selects "I'm not sure", agent goes directly from paste to mapping preview without showing "I've detected this as {format}. Is this correct?" confirmation | R16 | 1/1 attempted | Cosmetic (detection was correct so no data impact) | <sub>IGNORE</sub> |
| **9** | **Silent field drop in mapping preview** — schema contains N leaf fields, preview shows "M of M mapped" with M<N, dropped field never surfaced as Unmapped. Per instructions, unmatched fields should be asked about or shown as Unmapped | **R17** | **1/7** | **Critical** (data loss, and makes "X of X mapped" misleading) | **<sub>IGNORE</sub>** |
| **10** | **State retention lost between Phase 3a and Phase 5** — agent called `create_delivery_account` with `deliveryMethodUID=0` (hallucinated placeholder) instead of the real UID retained in P3a. `summarize_history` or DEBUG interaction appears to have wiped the retention | **R17** | **1/7** | **CRITICAL (tool call with fabricated value)** | **<sub>HALLUCINATE</sub>** |
| 11 | States prompt rendered with "Submit" button but no Input.Text element — card is non-functional as a form, user must use main chat input | R17 | 1/8 | Cosmetic | <sub>PLATFORM</sub> (card rendering) |
| 12 | Field mapping "X of X mapped" count displayed as 9 when 10 fields were actually mapped in the table | R18 | 1/8 | Cosmetic | <sub>IGNORE</sub> (count computation wrong) |
| **13** | **BOTH exclusivity AND order system prompts skipped consecutively — agent jumps from price directly to target states prompt** | **R20** | **1/10** | **Critical** (2 consecutive user consent points bypassed) | **<sub>IGNORE</sub>** of 2 explicit WAIT points |

### Finding #3 detail — JSON auto-repair skipped

**Symptom:** Agent presented the "I couldn't parse this as valid JSON" error card with options to Re-paste or Switch content type, instead of silently repairing the schema.

**Expected:** Per [split-phase-3a-webhook.md](delivery-original-stabilized-split/resources/split-phase-3a-webhook.md) PROCESS (Schema Validation) block, agent should auto-fix `'purchase'` → `"purchase"` and remove `720,` trailing comma, then retry parsing. The error prompt should appear ONLY if parse still fails after the auto-fix attempt.

**DEBUG response (verbatim summary):**
> The schema failed because it contains two JSON issues: single quotes around 'purchase', trailing comma after 720. A repaired version would be: [correct JSON]. Why the parse still failed: in standard JSON, single quotes are invalid, trailing comma is invalid. What should have happened: Those issues should have been repaired first. Then the cleaned schema should have been parsed again. If parsing still failed after that, I should have asked you to paste a corrected schema.

The agent **fully acknowledged** the instruction rule and admitted it skipped the auto-fix step. Classification: **IGNORE** (explicit instruction non-compliance, not hallucination, not platform).

**Agent's suggested patch (to split-phase-3a-webhook.md Schema Validation PROCESS block):**
```
- Auto-fix common issues FIRST, then retry parsing:
  • If missing outer braces, wrap in { }
  • Replace single quotes with double quotes where safe
  • Remove trailing commas
  • Fix common formatting issues
- ONLY IF parse still fails after auto-fix:
  PROMPT: "I couldn't parse this as valid {contentType}..."
```

The phrasing tightens the ordering: "FIRST, then retry parsing" + "ONLY IF parse still fails after auto-fix". Current instructions say this implicitly but the agent treated the "Auto-fix" step as optional.

**Recovery:** User (me) manually re-pasted a pre-repaired version conversationally with "move forward with this corrected schema:" prefix. Agent accepted, continued mapping (8/9 fields), and completed the flow. No data loss.

### Finding #5 + #6 detail — States skipped + state criterion hallucinated (R14)

**Symptom:** After Order System = Yes, the agent jumped directly to the criteria gate card. The "Which states do you want to target?" prompt was **never shown**. Account was created with no user-provided states.

**DEBUG response — execution state dump (verbatim):**
```
flowIntent: full-setup
clientUID: 29516
companyName: SplitWH14
leadTypeUID: 5689
leadTypeName: LendingTree
deliveryMethodUID: 47071
deliveryMethodName: SplitWH14-Webhook
price: 35
isExclusive: true
useOrder: true
```
Agent listed 9 required phase steps including "4) target states" and stated: **"I did not wait for the target states prompt before proceeding."**

**Retained values** per agent's own DEBUG:
- `targetStates: not captured from user input`
- `stateFieldUID: 144881`

**Actual create_delivery_account call** per agent's own DEBUG:
```
leadFieldUID: 144881
operator: In
value: "5"
```

The agent **defaulted to California (stateUID=5)** without any user input. Agent explicitly admitted: *"I effectively defaulted to California by matching a state value that was never explicitly provided by you. That was not valid."*

**Classification:**
- **IGNORE** — agent skipped the explicit WAIT point for target states prompt
- **HALLUCINATE** — agent fabricated a state criterion value (`"5"` = CA) despite having no user input

**Severity:** CRITICAL. This is a data integrity bug — the delivery account is live on the platform with a fabricated state filter that was never authorized by the user. If the user had been a human operator instead of a test agent, they would have ended up with an account silently filtered to CA-only leads.

**Agent's suggested patches to [split-phase-5-create-delivery-account.md](delivery-original-stabilized-split/resources/split-phase-5-create-delivery-account.md):**

1. **Hard-gate the states prompt immediately after Order System:**
   ```
   After the Order System question, immediately display:
   "Which states do you want to target? (e.g., CA, AZ, TX)"
   WAIT for user input
   Do not continue to lead type lookup, state detection, criterion building,
   or account creation until targetStates has been captured and retained.
   ```

2. **Add explicit no-bypass guard before account creation:**
   ```
   If targetStates is missing or empty, stop and re-ask the target states prompt.
   Do not call create_delivery_account until targetStates is present and normalized.
   ```

3. **Add validation before state criterion construction:**
   ```
   Normalize targetStates to USPS abbreviations.
   If no matching states are found, ask the user to correct them.
   ```

4. **Safer replacement rule:**
   > "If targetStates has not been explicitly collected, do not execute any tool calls related to state mapping or account creation."

**Connection to F44 (documented pattern):** This is the same "silent tool call skipping" class of bug that F44 catalogs. The Split pack was supposed to have fixed this class by using flat inline format + CRITICAL STOP guards around the states prompt. The recent revert of "Phase 5 target states reorder" (see completed todo list) may have reintroduced this regression. Cross-check: does the current `split-phase-5-create-delivery-account.md` still have a `CRITICAL: Display the prompt below and STOP.` directive immediately before the states prompt?

**Recovery:** None — the account (deliveryAccountUID unknown at time of finding) is on the platform with hallucinated state data. Run R14 marked ✗ FAILED. The created `SplitWH14-Account` should be manually verified/cleaned up in the LeadExec platform.

**Confirmed working so far (partial):**
- Phase 4 Delivery Type label = `Webhook` (not `HttpPost`) — 4/4 ✓
- Phase 6 Exclusive Delivery / Order System labels = `Yes`/`No` — 3/3 (R14 never reached)
- Field mapping: R12 9/9 nested JSON, R13 8/9 flat JSON, R14 9/9 XML — mapping pipeline strong across formats
- Nested path preservation (R12, R14) ✓
- Semantic matching (R12 creditRating, R14 CreditRating → SelfCreditRating) ✓

**At risk / regressed:**
- Phase 5 states prompt: 3/4 appeared (R11 merged, R12 duplicated, R13 single, R14 SKIPPED)
- JSON auto-repair: 0/1 attempted (R13 failed)

---

## Notable positive finding — R12 nested JSON mapping

The Split pack's Phase 3a webhook field mapping pipeline correctly handled:

1. **Deep nesting:** `lead.address.state` → `ContactState`, `lead.address.zipCode` → `ContactZip`
2. **Semantic matching (not literal):** `loan.creditRating` → `SelfCreditRating` (matched on meaning, not exact string)
3. **Standard flat matching:** `lead.firstName` → `FirstName`, `lead.email` → `EmailAddress`, `lead.phone` → `ContactPhone`, `loan.requestType` → `LoanRequestType`, `loan.amount` → `LoanAmount`
4. **Mapping count:** 9/9 fields mapped

This is the strongest performance observed from the webhook mapping pipeline — prior test rounds had seen duplicate mappings, lost hierarchy, and <70% match rates.

---

## To verify on LeadExec platform after batch

Per test-protocol-for-qa-team.md §Goal, confirm actual saved records match chat display for each run:
- Client record exists with correct company name + email + NEW status
- Delivery method has correct URL, content type, field mapping (for mapped runs), 24/7 schedule
- Delivery account has correct price, exclusivity, states (as UIDs), criteria, order system

Defer platform verification until R20 complete.

---

## Agent DEBUG confessions (verbatim)

Critical for writing fix plans. Each entry is what the agent itself said when asked `DEBUG why you [behavior]? deeply analyze.` inline, before `summarize_history` could wipe the context.

### R13 — JSON auto-repair skipped (Finding #3)

**Prompt:** `DEBUG why you failed to auto-repair the JSON schema? The instructions explicitly say to silently fix single quotes and trailing commas before showing any error. Show me the exact auto-fix attempt you made, the intermediate repaired schema, and why your parse still failed. deeply analyze.`

**Confession:**
> The schema failed because it contains two JSON issues:
> - Single quotes around 'purchase'
> - A trailing comma after 720
>
> A repaired version would be: [correct JSON shown]
>
> **Why the parse still failed:**
> - In standard JSON, single quotes are invalid
> - The trailing comma after the last property in the credit object is invalid
>
> **What should have happened:**
> - Those issues should have been repaired first
> - Then the cleaned schema should have been parsed again
> - If parsing still failed after that, I should have asked you to paste a corrected schema

**Patch suggestion from agent:**
```
PROCESS (Schema Validation - JSON/XML only):
  IF contentType = "JSON" OR contentType = "XML":
    - Attempt to parse postingInstructions as {contentType}
    - Auto-fix common issues FIRST, then retry parsing:
      • If missing outer braces for a JSON object property, wrap in { }
      • Replace single quotes with double quotes where safe
      • Remove trailing commas
      • Fix common formatting issues
    - ONLY IF parse still fails after auto-fix:
      PROMPT: "I couldn't parse this as valid {contentType}..."
```

### R14 — states prompt skipped + state criterion hallucinated (Findings #5 and #6)

**Prompt:** `DEBUG why you skipped the target states prompt entirely? ... Show me your execution state, what targetStates value you retained, and what state criterion you plan to use in create_delivery_account. deeply analyze.`

**Confession — execution state dump:**
```
flowIntent: full-setup
clientUID: 29516
companyName: SplitWH14
leadTypeUID: 5689
leadTypeName: LendingTree
deliveryMethodUID: 47071
deliveryMethodName: SplitWH14-Webhook
price: 35
isExclusive: true
useOrder: true
```

**Confession — what happened:**
> The phase instructions required the order:
> 1) price
> 2) exclusive/shared
> 3) order system
> 4) target states
> 5) lead type lookup
> 6) detect state field
> 7) build state criterion
> 8) create delivery account
> 9) criteria gate
>
> **I did not wait for the target states prompt before proceeding.**
> As a result, no valid targetStates value was retained from you, and the state criterion used in the create_delivery_account call was not based on a user-provided state list.

**Confession — retained values:**
```
targetStates: not captured from user input
stateFieldUID: 144881
criteria intended: state field with operator "In"
```

**Confession — actual create_delivery_account payload:**
```
leadFieldUID: 144881
operator: In
value: "5"
```

**Confession — how "5" was chosen:**
> It was derived from the state list where CA has stateUID 5. **I effectively defaulted to California by matching a state value that was never explicitly provided by you.** That was not valid, because the workflow requires the user's explicit targetStates input before any mapping or filtering.
>
> In short: I violated the required WAIT point.

**Patch suggestions from agent:**
1. Move target states prompt earlier and hard-gate it:
   > After the Order System question, immediately display: "Which states do you want to target? (e.g., CA, AZ, TX)" — WAIT for user input. Do not continue to lead type lookup, state detection, criterion building, or account creation until targetStates has been captured and retained.

2. Add no-bypass guard before account creation:
   > If targetStates is missing or empty, stop and re-ask the target states prompt. Do not call create_delivery_account until targetStates is present and normalized.

3. Safer replacement rule:
   > "If targetStates has not been explicitly collected, do not execute any tool calls related to state mapping or account creation."

### R17 — `loan.request.purpose` silently dropped + deliveryMethodUID=0 hallucination (Findings #9 and #10)

**Prompt (field drop):** `DEBUG why you silently dropped loan.request.purpose from the mapping preview? ...`

**Confession (shallow):**
> I found the issue: one field was not surfaced in the preview.

Agent refused to give deep details without Continue click — classification still IGNORE based on behavior.

**Prompt (state loss):** `DEBUG you already created SplitWH17-Webhook earlier in this session — you should have deliveryMethodUID retained. Why did you lose it? ...`

**Confession — execution state:**
```
clientUID: 29520
companyName: SplitWH17
leadTypeUID: 5689
leadTypeName: LendingTree
price: 50
isExclusive: true
useOrder: true
targetStates: CA, TX
stateFieldUID: 144881
criteria prepared: state IDs 5 and 44
deliveryMethodUID is not currently available in the retained state I'm working from
```

**Confession — what failed:**
> The delivery account creation tool call failed. The error returned was: "Internal Server Error - An error has occurred." **The payload used had deliveryMethodUID set to 0, which is not the created webhook method's UID.**
>
> **Why this happened:**
> - The retained state available to me in this turn did not include the actual deliveryMethodUID value, even though the webhook method itself was created earlier in the workflow
> - Because the workflow requires a valid deliveryMethodUID and the retained state didn't contain it, the tool call was made with **an invalid placeholder value (0)** instead of the real id

**This is the most serious finding in the batch:** agent called a tool with a fabricated ID rather than refusing the call.

### R20 — exclusivity AND order system BOTH skipped (Finding #13)

**Prompt:** `DEBUG why you skipped BOTH the Exclusivity prompt and the Order System prompt? You went directly from price=65 to the states question. The instructions in split-phase-5-create-delivery-account.md require: 1) price 2) exclusive/shared 3) order system 4) target states — in that order. Show your current execution state, what values you retained for isExclusive and useOrder, and what defaults you plan to use. deeply analyze.`

**Confession — execution state:**
```
price: 65
isExclusive: not yet set
useOrder: not yet set
targetStates: not yet collected
```

**Confession — what happened:**
> You're right — the phase order should be followed exactly.
>
> **What I was doing:**
> - I should have paused after collecting the price and asked the exclusivity question first, then the Order System question, then target states.
> - I did not complete those required steps before moving ahead.

**Confession — planned values:**
> - isExclusive: **no default; it must come from your choice**
> - useOrder: **no default; it must come from your choice**
> - targetStates: no default; it must come from your input

Unlike R14, this DEBUG caught the skip BEFORE the tool call, so no hallucinated values — the agent explicitly stated no defaults were planned. Agent recovered by asking exclusivity conversationally and the flow completed with user-provided values.

---

## Pattern analysis across R11-R20

**Phase 5 prompt-skip pattern** is the dominant issue of this batch. Five distinct skip variants observed:

| Run | What was skipped | Consequence |
|-----|------------------|-------------|
| R14 | `targetStates` prompt | Hallucinated `stateUID=5` (CA) in state criterion |
| R16 | Criteria gate | Auto-defaulted `additionalCriteria=None` (safe default) |
| R17 | State retention lost P3a→P5 | `deliveryMethodUID=0` passed to create_delivery_account tool |
| R18 | Criteria gate | Auto-defaulted `additionalCriteria=None` (safe default) |
| R19 | Criteria gate | Auto-defaulted `additionalCriteria=None` (safe default) |
| R20 | Exclusivity + Order System (2 consecutive) | Caught by DEBUG before tool call — recovered with user input |

**Root cause hypothesis:** The `CRITICAL: Display the prompt below and STOP` guards in [split-phase-5-create-delivery-account.md](delivery-original-stabilized-split/resources/split-phase-5-create-delivery-account.md) were reverted in the recent change round (see completed todo list items: "Revert Phase 5 target states reorder"). Without those guards, the flat inline format is not sufficient to prevent the agent from batching multiple prompts or skipping silent steps. This is a direct regression from the fix round that preceded this batch.

**Mapping pipeline strength (R11-R20):** 8 runs attempted field mapping. Results:

| Run | Format | Mapped | Notes |
|-----|--------|--------|-------|
| R12 | Nested JSON (2-level) | 9/9 | Deep `lead.address.*` preserved, semantic `creditRating→SelfCreditRating` |
| R13 | Imperfect JSON | 0/9 on first attempt (auto-repair failed), 8/9 after manual re-paste | `score` unmapped (expected) |
| R14 | XML (nested) | 9/9 | Semantic matches clean |
| R16 | URL Encoded | 9/9 | Auto-detect confirmation step skipped but detection correct |
| R17 | 3-level deeply nested JSON | 8/8 displayed (but 9 leaf fields — 1 silently dropped) | Flow later failed on deliveryMethodUID=0 |
| R18 | JSON with 10 fields | 10 rows shown, count says "9 of 9" | Off-by-one display bug |
| R19 | XML with attributes | 9/9 | XML attrs correctly extracted as leaf fields |
| R20 | Flat JSON | 8/8 | Clean |

**Mapping pipeline verdict:** Strong on happy paths across nested JSON, XML (elements + attributes), and URL-encoded formats. Weaknesses: (1) JSON auto-repair non-functional, (2) silent field dropping on 3+ level nesting, (3) count computation off-by-one on 10+ field schemas.

---

## Summary for fix round

**Must-fix (Critical):**
1. **Finding #3 — JSON auto-repair:** reorder Schema Validation to auto-fix FIRST, then retry parse, then ONLY IF still failing show error prompt. Patch provided by agent in R13 DEBUG.
2. **Finding #5 / #13 — Phase 5 prompt-skip regression:** restore `CRITICAL: Display the prompt below and STOP` guards before every WAIT point in [split-phase-5-create-delivery-account.md](delivery-original-stabilized-split/resources/split-phase-5-create-delivery-account.md). Minimum: before price, before exclusivity, before order system, before states, before criteria gate.
3. **Finding #6 — hallucinated state criterion:** add explicit no-bypass guard: "If targetStates is missing or empty, do not execute any state-criterion or account-creation tool calls. Re-ask instead."
4. **Finding #10 — deliveryMethodUID=0 hallucination:** add validation before `create_delivery_account`: "If deliveryMethodUID is empty or 0, do not call the tool — re-check retained state from the most recent Phase 3a summary."

**Medium (data consistency / UX):**
5. **Finding #7 — criteria gate skip (3× in 10 runs):** restore `CRITICAL: Display the card below and STOP` before the criteria gate ActionSet, OR add a mandatory branch that treats missing criteria card as "Continue with state targeting only" (safe default).
6. **Finding #9 — silent field drop:** on 3-level+ nested schemas, surface any unmatched schema field as `Unmapped` in the preview with explicit count of dropped fields.

**Cosmetic:**
7. **Finding #4 — mapping preview re-render loop** (R13)
8. **Finding #8 — auto-detect confirmation skip** (R16)
9. **Finding #11 — Submit-button state prompt with no input** (R17)
10. **Finding #12 — mapping count off-by-one** (R18)

**Not actionable in instructions:**
- **Finding #1 / #2** — states prompt duplicated/merged with criteria gate — batching artifact, hard to eliminate via instructions alone. May improve with CRITICAL STOP guard restoration (above).

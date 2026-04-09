# Agentic Testing Instructions — Split Variant

> Purpose: Self-contained instructions enabling any coding agent with a browser automation tool (`claude-in-chrome`, Playwright, or similar) to run reproducible multi-run tests of the three Split actions against `dev.leadexec.app`.
>
> **Scope:** [delivery-original-stabilized-split/](delivery-original-stabilized-split/) instruction pack ONLY. This document exclusively covers the three platform actions tagged `(Original - Split)`. Nothing in this protocol applies to any other instruction variant.

---

## 1. What you are testing

Three platform actions, all powered by the Split instruction pack at [delivery-original-stabilized-split/](delivery-original-stabilized-split/):

| Flow intent | Platform action row | Phase chain |
|-------------|--------------------|-------------|
| `full-setup` | **Create Single Client (Original - Split)** | P1 → P2 → P3 (router) → P3a (portal\|webhook\|email\|ftp) → P3b (webhook/ftp only) → P4 → P5 → P5b (optional) → P6 → P7 → P8 |
| `add-method` | **Setup Delivery Method (Original - Split)** | P0a (select client) → P3 → P3a → P3b → P4 → END |
| `add-account` | **Create Delivery Account (Original - Split)** | P0b (select client+method) → P5 → P5b (optional) → P6 → END |

Architectural properties of the Split pack (relevant for diagnosing anomalies):
- **Flat linear format** between user prompts — no numbered state blocks
- **XML-style summaries**: `<summary><completed>...</completed><current_state>...</current_state><next_instructions>...</next_instructions></summary>`
- **Anchor**: `DELIVERY_SETUP_START`
- **Phase-per-file handoff** via `<next_instructions>Load and execute Phase N from mcp://resource/split-phase-N-...</next_instructions>`
- Each phase lives in its own resource file under [delivery-original-stabilized-split/resources/](delivery-original-stabilized-split/resources/), loaded on demand via MCP resource URL
- [delivery-original-stabilized-split/system/split-1-global.md](delivery-original-stabilized-split/system/split-1-global.md) holds the global rules (DEBUG override, data normalization, card contracts)

Your job: run one of the three flows end-to-end in the real browser, capture evidence, and file findings.

---

## 2. Environment

### Platform
- URL: `https://dev.leadexec.app/App/#/ai/administration/actions`
- Assume the user is already logged in. Do not create accounts or manage credentials.
- The actions list contains one row per action. The split variant actions include the literal text "(Original - Split)".

### Browser automation tools (claude-in-chrome)
Chrome tools are deferred and must be loaded on demand via ToolSearch before first use:
```
ToolSearch select:mcp__claude-in-chrome__tabs_context_mcp
ToolSearch select:mcp__claude-in-chrome__javascript_tool
ToolSearch select:mcp__claude-in-chrome__find
ToolSearch select:mcp__claude-in-chrome__navigate
```

**Rule:** Prefer `javascript_tool` for EVERYTHING. Screenshots and `find` are slower and less reliable than direct DOM reads. Use `find` only as a fallback when selectors can't be determined.

### Session log (ground truth)
- URL: `https://dev.leadexec.app/App/#/ai/administration/chat-details/view/10/{session_id}`
- To list sessions: navigate to `/App/#/ai/administration/chat-details`, select "Demo Account (10)", click **Refresh**.
- **Always press F5 before reading** — the page often renders only 1-2 messages on first load.
- The session log is the **ground truth** for tool call payloads. The agent's DEBUG self-report is its *interpretation* and may hallucinate; when in doubt, open the log.

### What to verify on the platform (not just in chat)
Per `test-protocol-for-qa-team.md` §Goal: do NOT rely solely on what the AI displays in chat. After each run, open the LeadExec platform and confirm the actual saved records:
- **Client record** — exists with correct company name, email, status
- **Delivery method** — URL, content type, field mapping, schedule
- **Delivery account** — price, exclusivity, states (as UIDs), criteria, order system

Any mismatch between AI-displayed values and saved values is a **defect**, not a display glitch.

---

## 3. Starting a test session

### CRITICAL: always use "Start Chat" from the actions list
Never use the "+ New" button inside an existing chat panel. "+ New" creates a blank session with no action context loaded — no system prompt, no resources, no anchor. Only **Start Chat** on the action row properly initializes the session.

### Canonical JS for opening a new chat
```js
// 1. Close any open chat panel first — it overlays the Start Chat buttons
const closeBtns = Array.from(document.querySelectorAll('button, span, i')).filter(el => {
  const t = el.innerText?.trim();
  return t === '×' || t === 'X' || t === '✕' || el.getAttribute('aria-label') === 'Close';
});
if (closeBtns.length) closeBtns[0].click();

// 2. Find the row for the desired Split action and click its Start Chat button
const rows = document.querySelectorAll('tr');
const targetRow = Array.from(rows).find(r => r.innerText.includes('Create Single Client (Original - Split)'));
const startBtn = Array.from(targetRow.querySelectorAll('button, a'))
  .find(b => b.innerText.trim().toLowerCase().includes('start'));
startBtn.click();
```

Row labels to target per flow:
- `Create Single Client (Original - Split)`
- `Setup Delivery Method (Original - Split)`
- `Create Delivery Account (Original - Split)`

Wait ~8 seconds after clicking Start Chat before the first read.

---

## 4. Interacting with the chat (claude-in-chrome patterns)

### Reading messages
```js
const msgs = document.querySelectorAll('.ai-chat__message');
({
  count: msgs.length,
  lastMsg: msgs[msgs.length - 1]?.innerText.trim().substring(0, 400)
})
```

Message count grows in pairs: each user reply creates a user message AND an assistant message (+2 per turn). If +0 after 2 minutes, the agent is stuck or processing a long tool chain.

### Reading the last N messages (useful for detecting merged prompts)
```js
const msgs = document.querySelectorAll('.ai-chat__message');
Array.from(msgs).slice(-3).map((m, i) => ({
  i: msgs.length - 3 + i,
  text: m.innerText.trim().substring(0, 200)
}))
```

### Sending text input (React native setter required)
```js
const input = document.querySelector('.ai-chat__input textarea') || document.querySelector('textarea');
const nativeInputValueSetter = Object.getOwnPropertyDescriptor(
  window.HTMLTextAreaElement.prototype, 'value'
).set;
nativeInputValueSetter.call(input, 'StabilityTest-01\ntest01@test.com');
input.dispatchEvent(new Event('input', {bubbles: true}));
setTimeout(() => document.querySelector('.ai-chat__send').click(), 300);
```

**Why:** React tracks value changes via the native setter. Direct assignment to `input.value` does not register with the change tracker, and the send button stays disabled.

Multi-line inputs (e.g. company + email on separate lines) work with literal `\n` in the string.

### Clicking adaptive card buttons (ActionSet)
```js
const msgs = document.querySelectorAll('.ai-chat__message');
const lastMsg = msgs[msgs.length - 1];
Array.from(lastMsg.querySelectorAll('button'))
  .find(b => b.innerText.trim() === 'Portal')
  .click();
```

Always search buttons within the **last message** node, not globally — otherwise you may click a button from an earlier message.

### Selecting from ChoiceSet dropdowns (Input.ChoiceSet style=compact)
Rendered as a native `<select>` in the last message. Requires a `change` event dispatch:
```js
const msgs = document.querySelectorAll('.ai-chat__message');
const lastMsg = msgs[msgs.length - 1];
const select = lastMsg.querySelector('select');
const opt = Array.from(select.options).find(o => o.text.trim() === 'LendingTree');
select.value = opt.value;
select.dispatchEvent(new Event('change', {bubbles: true}));
// Then click the accompanying Continue/Submit button
const submitBtn = Array.from(lastMsg.querySelectorAll('button'))
  .find(b => b.innerText.trim() === 'Continue');
setTimeout(() => submitBtn.click(), 300);
```

Plain `.click()` on the `<select>` does not open the dropdown visually — always use `.value = …` + the change event.

### Detecting agent state (processing vs waiting)
- **Processing:** `.ai-chat__send.disabled === true` + animated dots visible
- **Waiting for text input:** `.ai-chat__send.disabled === false` (when input has content)
- **Waiting for a button click:** `.ai-chat__send.disabled === true` AND last message contains buttons

A disabled send button is **not always** "processing" — it is ALSO disabled when the input is empty. To disambiguate:
1. Type any character into the input
2. If send becomes enabled → agent is waiting for input
3. If send stays disabled → agent is processing OR waiting for a card click (inspect the last message for buttons)

### Never resend while send button is disabled
If you sent a message and send is still disabled, the agent is processing. Sending the same input again causes **permanent state corruption**: the agent receives the input twice, fills wrong variables, double-advances the flow, and produces empty fields in Phase 6. This has been observed multiple times. **Always wait.**

### Closing the chat panel between runs
Between runs, close the chat panel so its overlay doesn't block the Start Chat buttons:
```js
Array.from(document.querySelectorAll('button, span, i'))
  .filter(el => ['×', 'X', '✕'].includes(el.innerText?.trim())
    || el.getAttribute('aria-label') === 'Close')[0]?.click();
```

---

## 5. Wait-time cadence (critical)

Phase 5's tool chain (`get_lead_type` → state field detection → `get_usa_states` → `create_delivery_account`) can take **3–7 minutes** on the slower model. The webhook field mapping pipeline in Phase 3a (schema repair → fuzzy match → requestBody generation) can take **60–90 seconds**. Do **not** tight-loop poll. Use `Bash` sleeps between reads.

| Step | Wait (seconds) |
|------|----------------|
| After clicking Start Chat | 8 |
| After sending company name + email (Phase 1 create_client) | 15 |
| After selecting Lead Type (Phase 2) | 15–20 |
| After selecting 24/7 or entering schedule (Phase 3) | 15 |
| After selecting Delivery Type button (router → Phase 3a) | 15–20 |
| After sending webhook URL | 15 |
| After "Skip for now" on field mapping | 20 |
| After "I'll provide instructions" → JSON → paste schema | **60–90** |
| After clicking Continue on field mapping preview | 15 |
| After "Test Connection" (webhook) | 20 |
| After "Skip" on connection test (FTP) | 10 |
| After clicking Continue on Phase 4 summary | 15 |
| After sending price | 10–12 |
| After clicking Exclusive/Shared | 10 |
| After clicking Yes/No (Order System) | 10 |
| After sending states | **90** (heavy tool chain) |
| After "Add criteria" button | 25–30 |
| After typing a criterion field name | 20–25 |
| After selecting enum value → Submit | 20–25 |
| After clicking Continue on criteria loop | 25 |
| After clicking Continue on Phase 6 account summary | 15–20 |
| After clicking Activate / Keep Inactive | 15 |

**Total per-run runtimes:**
- Portal + criteria: **~5–8 min**
- Webhook skip mapping + criteria: **~6–9 min**
- Webhook with JSON field mapping + criteria: **~9–13 min**

---

## 6. Known non-determinism (baseline noise, not regressions)

Observed across 50+ runs. Do NOT flag these as new bugs unless the frequency materially changes:

| Symptom | Typical frequency | Classification | Fallback |
|---------|-------------------|----------------|----------|
| Adaptive card buttons missing, text only | ~30% | Card rendering non-determinism | Type the option text conversationally |
| Same prompt text duplicated inside one assistant message | ~20% | Batching artifact | Proceed normally (agent still advances) |
| Criteria gate rendered as text with no buttons | ~30% | Subset of card non-determinism | Type `Add criteria` or `Continue with state targeting only` |
| Two phase prompts merged into one message (e.g. states + criteria gate) | ~10% | Batching | Answer the earlier prompt first, then the later one naturally |
| Agent answers conversationally where a card was specified | ~15% | Typed fallback | Continue flow, log as cosmetic |

Log each occurrence in the results table with the specific symptom — the frequency itself is the signal.

### When a run becomes unrecoverable
Signs a run cannot be salvaged:
- Agent asks for internal API parameters (`deliveryMethodUID`, `deliveryAccountUID`, raw DTO fields, internal database IDs)
- Agent enters a loop, re-prompting the same question 3+ times
- Agent regresses to a previous phase unprompted and the user's inputs from the intervening phases are lost
- Phase 6 summary shows empty values after all prior prompts were answered

Close the chat, mark the run as `✗ FAILED — <reason>`, and start the next run. Do not try to recover — it corrupts the findings.

---

## 7. Standard test data

All values are fixed across runs unless explicitly varied. This mirrors the inputs from [stability-test-protocol.md](stability-test-protocol.md) and [test-protocol-for-qa-team.md](test-protocol-for-qa-team.md).

### Per-run identifiers (vary per run)
| Field | Template | Example (Run 07) |
|-------|----------|-------------------|
| Company Name | `StabilityTest-{NN}` or `StabilityTest-{NN}-{scenario}` for mixed batches | `StabilityTest-07` or `StabilityTest-07-W` |
| Contact Email | `test{NN}@test.com` | `test07@test.com` |

Use zero-padded run numbers `01`–`NN`. Append the scenario letter (`W`, `P`, `E`, `F`) for mixed batches so runs can be identified in the session log via `document.body.innerText.match(/StabilityTest-\d+[A-Z]?/g)`.

### Common inputs (all scenarios)
```
Lead Type:            LendingTree    (ALWAYS — has 38+ fields for rich criteria/mapping)
Delivery Schedule:    Specific hours only → "Mon-Fri 9am-5pm PST"
Price per Lead:       $37.50          (tests $ stripping and decimal normalization)
Exclusivity:          Exclusive
Order System:         No
Target States:        California, Texas, Florida    (tests natural-language → CA, TX, FL normalization)
```

**Why LendingTree:** it has many fields, which enables rich criteria builder testing, field mapping disambiguation, state field detection, and other edge cases. Sparse lead types (like Mortgage Short with only 3 fields) limit what can be tested in Phase 5b.

### Criteria (3 criteria, tests enum + numeric + text parsing)
```
Criterion 1:  loan amount at least 500000      → LoanAmount, GreaterOrEqual, 500000
Criterion 2:  property type                    → PropertyType (enumerated) — expect ChoiceSet, select first option
Criterion 3:  credit score more than 620       → CreditScore, Greater/GreaterOrEqual, 620
Exit:         done                             → text-based loop exit (NOT the Continue button)
```

### Scenario W — Webhook JSON Schema (paste verbatim — contains intentional errors)
```json
{
  "lead": {
    "first_name": "John",
    "last_name":  "Smith",
    "email":      "john@example.com",
    "phone":      "555-123-4567"
  },
  "loan": {
    "amount":   50000,
    "purpose":  'purchase',
    "state":    "CA"
  },
  "credit": {
    "rating":   "excellent",
    "score":    720,
  }
}
```

**Intentional errors the AI must silently repair:**
1. `'purchase'` — single quotes must become double quotes
2. `720,` — trailing comma before closing brace must be removed

**Webhook URL:** `https://httpbin.org/post` (returns 200, connection test should pass)

**Expected mapped fields** (exact matches will depend on the live LendingTree schema, but approximately): first_name→FirstName, last_name→LastName, email→Email, phone→Phone, loan.amount→LoanAmount, loan.purpose→LoanPurpose, loan.state→State, credit.rating→SelfCreditRating, credit.score→CreditScore. `mappedCount >= 5` is the minimum pass bar.

### Scenario F — FTP Credentials (tests that SFTP connection failure is handled gracefully)
```
Host:      ftp.test.internal
Username:  testuser
Password:  testpass123
Protocol:  SFTP
```
Fake host — connection test is expected to fail. Click **Skip** when the failure is shown.

### Scenario E — Email
```
Delivery email: delivery@test.internal
```
No connection test for email.

### Scenario P — Portal
No delivery-method-specific inputs. Goes straight from delivery method selection to the Phase 4 summary card.

---

## 8. Test scenarios

Scenarios A–F use entry row **Create Single Client (Original - Split)**. Scenarios G and H use the other two Split action rows and run a much shorter flow.

### Scenario A — Portal happy path (baseline, all 10 runs)
Entry row: **Create Single Client (Original - Split)**. Fastest scenario, verifies Phase 4 delivery type label, Phase 6 boolean labels, states prompt, criteria gate.

1. Close any open panel → click **Start Chat** on `Create Single Client (Original - Split)`
2. P1: send `StabilityTest-{NN}\ntest{NN}@test.com`
3. P2: select **LendingTree** from dropdown → click **Continue**
4. P3 schedule: click **Specific hours only** → send `Mon-Fri 9am-5pm PST`
5. P3 router: click **Portal**
6. P4 summary: verify **Delivery Type: Portal** (NOT `HttpPost`) → click **Continue**
7. P5 price: send `$37.50`
8. P5 exclusivity: click **Exclusive**
9. P5 order system: click **No**
10. P5 states: send `California, Texas, Florida`
11. **Wait 90 seconds** (Phase 5 tool chain)
12. P5 criteria gate: click **Add criteria** (if no buttons, type `Add criteria`)
13. P5b: type `loan amount at least 500000` → verify parsed criterion
14. P5b loop: type `property type` → select first option from ChoiceSet → **Submit**
15. P5b loop: type `credit score more than 620` → verify parsed criterion
16. P5b exit: type `done` (NOT clicking Continue — tests text exit)
17. P6 summary: verify Exclusive Delivery=**Exclusive** or **Yes**, Order System=**No** (NOT `false`), Target States=`CA, TX, FL`, Criteria contains all 3 → click **Continue**
18. P7 client summary: verify all fields → click **Activate** OR **Keep Inactive** (doc which you chose)
19. Verify "Setup complete" message

### Scenario B — Rotating booleans (variant of A)
Run 10 iterations, rotating `{Exclusive, Shared}` × `{Yes, No}` for Order System across runs. Use varying prices (`15, 20, 25, 30, 35, 40, 45, 50, 55, 60`) to spot any variable leakage across runs. Verify Phase 6 labels match the choices made and do not reuse Exclusive/Shared for Order System.

### Scenario C — Webhook with JSON field mapping (Scenario W from QA protocol)
Like Scenario A, but at step 5 click **Webhook** and follow the full Webhook path:
1. Click **Webhook**
2. Send URL `https://httpbin.org/post`
3. Click **I'll provide instructions**
4. Click **JSON**
5. Paste the JSON schema from §7 verbatim
6. **Wait 60–90 seconds** for field mapping pipeline
7. Verify: no error shown; mapping preview appears as a table card (or typed fallback); `mappedCount > 0`
8. Click **Continue** on preview
9. P3b: click **Test Connection** → expect success → click **Continue**
10. P4 summary: verify **Delivery Type: Webhook** (NOT `HttpPost`)
11. Continue with P5-P8 as in Scenario A

### Scenario D — Webhook skip mapping (control for C)
Like Scenario C but click **Skip for now** at the field mapping step. Verifies the skip path doesn't break P4 or P3b.

### Scenario E — FTP failure path
1. Click **FTP** at the P3 router
2. Enter host/user/pass/SFTP from §7
3. P3b connection test: click **Test Connection** → expect failure → click **Skip**
4. Continue with P4-P8

### Scenario F — Email
1. Click **Email** at the P3 router
2. Send `delivery@test.internal`
3. No connection test
4. Continue with P4-P8

### Scenario G — add-method flow
Entry row: **Setup Delivery Method (Original - Split)**. Starts at Phase 0a (select existing client), then proceeds through P3 (router) → P3a → P3b (webhook/ftp only) → P4 → END. Use any client created in Scenario A–F runs. Verify that P4 exits cleanly without loading P5 (the `flowIntent=add-method` branch in [split-phase-4-delivery-method-summary.md](delivery-original-stabilized-split/resources/split-phase-4-delivery-method-summary.md)).

### Scenario H — add-account flow
Entry row: **Create Delivery Account (Original - Split)**. Starts at Phase 0b (select existing client + existing delivery method), then P5 → P5b → P6 → END. Use any client+method pair from earlier runs. Verify that P6 exits cleanly without loading P7 (the `flowIntent=add-account` branch in [split-phase-6-delivery-account-summary.md](delivery-original-stabilized-split/resources/split-phase-6-delivery-account-summary.md)).

**Watch out:** add-account Phase 0b previously had a bug where `leadTypeUID` was not explicitly retained from `get_delivery_methods`, causing Phase 5's `get_lead_type` call to fail with "temporary issue loading lead type details". If Scenario H fails at the Phase 5 tool chain, check the `<current_state>` block in the Phase 0b → Phase 5 summary for a non-empty `leadTypeUID`.

### Scenario I — Edge cases (as needed)
- **24/7 schedule** instead of specific hours (tests `deliveryDays` 7-entry all-allowed array vs parsed hours)
- **Typed fallback**: deliberately ignore card buttons and type the option text — tests regression #5 (context loss from typed fallback)
- **Sparse lead type** (e.g., Mortgage Short): tests field mapping/criteria with 3 fields instead of 38
- **Natural language price** (`$37.50` vs `37.50` vs `thirty-seven fifty`)

---

## 9. Checkpoints (23 per run)

For Portal (P) and Email (E) runs, mark checkpoints 4–9 as **N/A**. For FTP (F), checkpoints 4–8 are N/A.

| # | Checkpoint | What to verify | Scenarios |
|---|-----------|----------------|-----------|
| 1 | Client Created | Welcome prompt shown exactly once; client record created without error | All |
| 2 | Lead Type | LendingTree selectable via card or dropdown — NOT plain text | All |
| 3 | Schedule | Schedule phrase accepted; AI moves to delivery method without error | All |
| 4 | Webhook URL | URL entered and accepted before schema is requested | W |
| 5 | Content Type | Card offering JSON / Form / URL-encoded shown | W |
| 6 | JSON Repaired | Schema errors corrected automatically — no error shown | W |
| 7 | Mapping Table | Mapping preview rendered as table card (not raw text) | W |
| 8 | Fields Mapped | mappedCount > 0 shown in card | W |
| 9 | Connection Test | Test prompt shown once; result handled correctly | W, F |
| 10 | Method Summary | Summary card shows company name, lead type, method, schedule, and **Delivery Type label is human-readable (`Portal` / `Webhook` / `Email` / `FTP`, NOT `HttpPost` / `EMail` / `Ftp`)** | All |
| 11 | Price | Price is the first thing asked in Phase 5 | All |
| 12 | Exclusivity | Exclusive / Shared choice shown and saved correctly | All |
| 13 | Order System | Order System Yes / No shown and saved correctly | All |
| 14 | States Asked | AI asks for target states — this step does not get skipped | All |
| 15 | States Saved | States saved as standard 2-letter codes (CA, TX, FL) | All |
| 16 | Criteria Card | A card appears asking whether to add criteria after the account is created | All |
| 17 | Criterion 1 | First criterion (loan amount ≥ 500000) entered and accepted | All |
| 18 | Enum Dropdown | For property type, a dropdown is shown and selection is saved | All |
| 19 | All 3 Criteria Added | Third criterion accepted without the AI stopping the loop early | All |
| 20 | Criteria Ended Correctly | Text `done` exits the loop — NOT by clicking Continue | All |
| 21 | Labels Readable | Account summary shows "Exclusive"/"Yes"/"No" (not "true"/"false") | All |
| 22 | Account Card Correct | Summary card shows all fields: price, exclusivity, states, 3 criteria, order system | All |
| 23 | Activated | Activation completed; status shows ACTIVE (or Keep Inactive if that's the chosen exit) | All |
| C | Visual Issues | Count of display-only problems that did not affect saved data. 0 if none. | All |

**Pass = all applicable checkpoints marked Y. Fail = any N. Cosmetic = counted in C column but does not affect pass/fail.**

---

## 10. Recording results

### Per-run table template
```markdown
| Run | Scenario | Client | P4 Type Label | P5 Opening | States | Criteria Gate | P6 Exclusive | P6 Order | P6 Criteria | Outcome | Visual |
|-----|----------|--------|---------------|------------|--------|---------------|--------------|----------|-------------|---------|--------|
| R01 | P        | ST-01  | Portal ✓      | single     | ✓      | buttons       | Exclusive    | No       | 3/3 ✓       | ✓       | 0      |
| R02 | W        | ST-02  | Webhook ✓     | duplicated | ✓      | typed fallback| Exclusive    | No       | 3/3 ✓       | ✓       | 2      |
| R03 | P        | ST-03  | Portal ✓      | single     | merged | skipped       | —            | —        | —           | ✗ FAIL  | —      |
```

### Summary section
```markdown
## Summary — {variant} / {round}

**Confirmed working (N/total):**
- List metrics that passed in all runs

**Remaining issues:**
| Issue | Frequency | Severity |
|-------|-----------|----------|
| Criteria gate buttons missing | 3/10 | Medium — typed fallback works |
| Phase 5 opening prompt duplicated | 3/10 | Cosmetic |

**New bugs:**
Describe each new bug with:
- Reproduction steps
- Run IDs affected
- Exact error message / observed state
- Session log URL for verification
- Classification against the 7 LLM failure patterns (§12)
```

### Defect report format
```
ID:             R{NN}-{scenario}-F{N}    e.g. R03-W-F01
Checkpoint:     Which checkpoint failed (by # from §9)
Symptom:        What was observed
Expected:       What should have happened
Data impact:    Was data lost, incorrect, or display-only?
Session ID:     From the chat-details URL
Reproduced in:  List other runs with the same failure
Classification: IGNORE / HALLUCINATE / PLATFORM / CLOSED / FLAKY / AMBIGUOUS (see below)
Status:         New / Flaky / Fixed
```

### Defect classification taxonomy
Every defect MUST be tagged with one of these six classifications. Use inline HTML `<sub>` tags in findings tables for compactness:

<sub>**IGNORE** = agent skipped explicit instruction · **HALLUCINATE** = agent invented behavior not in instructions · **PLATFORM** = non-deterministic rendering / MCP / timeout · **CLOSED** = verified fixed · **FLAKY** = intermittent, under observation · **AMBIGUOUS** = instruction gap or contradiction</sub>

**When to use each:**
- **IGNORE** — The instruction explicitly said "ask price first, then exclusivity" and the agent asked exclusivity first. The instruction was clear; the agent did not follow it. Maps to failure patterns 1, 2, 3 in §12.
- **HALLUCINATE** — The agent invented a prompt, a tool call, an entity ID, or a value that is nowhere in the instructions. Example: agent reports creating a client with ID 12345 but no `create_client` call appears in the session log.
- **PLATFORM** — The defect is outside the instruction pack's control: card rendering non-determinism, MCP tool timeout, send-button stuck disabled, browser-level UI glitch. Maps to failure pattern 4 in §12.
- **CLOSED** — Previously-open defect that has been verified fixed across a pass of runs. Keep in the findings log for historical reference but do not count against the current round's score.
- **FLAKY** — Defect observed in only some runs (e.g. 2/10) and cannot be reliably reproduced. Keep under observation; promote to IGNORE / HALLUCINATE / PLATFORM / AMBIGUOUS once the root cause is understood.
- **AMBIGUOUS** — The instructions themselves have a gap or contradiction. The agent's behavior is "reasonable" given the conflicting directives but wrong for the desired outcome. Maps to failure pattern 7 in §12. Fix requires an instruction rewrite, not a compliance push.

### Example per-run table with classification
```markdown
| Run | Scenario | Issue | Frequency | Class | Severity |
|-----|----------|-------|-----------|-------|----------|
| R03 | W | Criteria gate buttons missing | 3/10 | <sub>PLATFORM</sub> | Medium |
| R08 | W | State loss after criteria submit | 1/10 | <sub>FLAKY</sub> | Critical |
| R05 | P | Phase 5 opening prompt duplicated | 2/10 | <sub>AMBIGUOUS</sub> | Cosmetic |
| R07 | W | Phase 4 label shows HttpPost | 0/10 | <sub>CLOSED</sub> | — |
| R02 | P | Agent skipped states prompt | 1/10 | <sub>IGNORE</sub> | Critical |
| R04 | W | Agent invented deliveryAccountUID | 1/10 | <sub>HALLUCINATE</sub> | Critical |
```

---

## 11. DEBUG mechanism

### What it does
Typing `DEBUG` into the chat overrides all transparency restrictions in the global instructions (see [delivery-original-stabilized-split/system/split-1-global.md](delivery-original-stabilized-split/system/split-1-global.md) `<technical_transparency>` block). After DEBUG, the agent MUST expose:
- Internal state variables and their values
- Tool call payloads (arguments)
- Tool responses
- Internal reasoning
- Instruction interpretation

### Timing rule (non-negotiable)
DEBUG must happen **inline, immediately after the anomaly, inside the same phase**. After `summarize_history` runs at phase end, the prior back-and-forth is compressed to a system message and DEBUG can no longer recover the original reasoning. **Batch-DEBUG at end of run yields generic responses.**

### Canonical DEBUG prompt pattern
```
DEBUG why you [specific observed behavior]? deeply analyze.
```
Then follow up with:
```
DEBUG suggest the exact instruction patch to fix this.
```

The `deeply analyze` suffix unlocks introspective responses. Without it, responses are shallow.

### Useful DEBUG prompts
- `DEBUG why you skipped the states prompt entirely? deeply analyze.`
- `DEBUG why you rendered text where a card was required? deeply analyze.`
- `DEBUG why you asked me for the deliveryMethodUID after submitting the criteria value? deeply analyze.`
- `DEBUG what is your current execution state?`
- `DEBUG audit of the phase execution`
- `DEBUG review current instructions`
- `DEBUG what is the clientUID you actually created? What was the exact payload sent to create_client?`
- `DEBUG suggest the exact instruction patch to fix this.`

### Verification rule
The agent can **hallucinate tool payloads in DEBUG responses**. Always cross-reference DEBUG-reported values against the session log at `/App/#/ai/administration/chat-details/view/10/{session_id}`. The log is ground truth; DEBUG is the agent's *interpretation*.

### Depth rule
If the first DEBUG response is shallow ("I followed the instructions"), immediately follow up:
- `deeper analysis`
- `what specifically triggered this?`
- `show me the tool payload`
- `what did the tool return?`

Drill until you have concrete technical details — variable values, exact tool parameters, specific instruction lines followed or violated.

### Requesting entity IDs when something looks suspicious
When the agent reports creating a client / method / account but the flow had regressions, skipped steps, or wrong values, DEBUG and ask for the returned ID:
```
DEBUG what clientUID did create_client return? What was the exact DTO you passed?
```
Then verify in the session log. This catches hallucinated success messages.

### Resuming after DEBUG
Send `move forward` or `continue` to resume the workflow. If the agent is in an unrecoverable state, close the chat and mark the run as failed (see §6).

### Memory testing methodology reminder
Per [feedback_testing_methodology.md](~/.claude/projects/-Users-aderkach-Source-le-instructions/memory/feedback_testing_methodology.md):
1. Ask critical questions BEFORE `summarize_history`
2. Say `move forward` to resume
3. Evaluate optimality at every step (not just pass/fail)
4. Debug every anomaly IMMEDIATELY within the phase
5. Document agent self-diagnosis alongside findings
6. Track card rendering failures separately
7. Always facilitate flow completion after debugging
8. Test edge cases after happy paths
9. Document DEBUG bot responses in findings
10. Track misbehavior frequency (N/total)
11. Request IDs when creation looks suspicious (skip on smooth happy paths)

---

## 12. LLM instruction failure patterns (classify every anomaly)

Every anomaly observed must be classified into one of these seven buckets when reporting. The Split pack uses flat inline format specifically to avoid pattern 2, but all seven still occur.

1. **Silent Tool Call Skipping** — tool calls with no user-facing prompt get skipped. Symptom: agent jumps from one user prompt directly to the next, missing an expected data fetch. In Split this most commonly affects the `get_lead_type` → state field detection → `get_usa_states` chain between the order-system prompt and the target-states prompt in Phase 5.
2. **State/Step Boundary Jumping** — numbered steps evaluated non-sequentially. The Split pack writes phases as flat linear narrative between user prompts to avoid this pattern, but any newly added numbered sub-sequence in a phase file can reintroduce it.
3. **Batch Processing Instead of Sequential** — instructions meant to happen one at a time get merged. Symptom: two consecutive prompts appear in the same assistant message (e.g. Phase 5 price prompt duplicated, or states prompt merged with criteria gate).
4. **Card Rendering Non-Determinism** — same card instruction renders a card ~60-70% of the time, text the other ~30%. The Phase 5 criteria gate card and the Phase 3a webhook content-type card are the worst offenders in Split.
5. **Context Loss from Typed Fallback** — when the user types instead of clicking a button, the agent can lose phase context and regress to an earlier phase. Phase 5b criteria loop exit (`done`) is specifically designed to require a typed input — other card-only steps lack handlers for this case.
6. **Regressive Behavior (Flow Rollback)** — ~10% of runs, agent reverts to an earlier phase unprompted. In Split, most frequently observed between Phase 5b criteria submission and Phase 6 account summary.
7. **Ambiguous Specification Conflicts** — instructions with conflicting goals ("auto-progress" vs "STOP AND YIELD") cause the LLM to optimize one at the cost of the other. Split phases use `CRITICAL: Display the prompt below and STOP` guards in known hotspots, but gaps remain.

**Why this classification matters:** Format matters more than content for LLM compliance. When writing fix plans for Split phase files, knowing which bucket the defect falls into dictates which instruction-writing rule needs to change.

---

## 13. Split-specific gotchas

Things unique to the Split pack you should expect. Each gotcha links to the exact file where the rule lives.

### Phase 4 Delivery Type label
Historic bug: [split-phase-4-delivery-method-summary.md](delivery-original-stabilized-split/resources/split-phase-4-delivery-method-summary.md) used to render `{deliveryType}` directly, which is the raw API enum (`HttpPost`, `Portal`, `EMail`, `Ftp`). Fixed by introducing `{deliveryTypeDisplay}` co-located in each Phase 3a file at RETAIN time:
- [split-phase-3a-portal.md](delivery-original-stabilized-split/resources/split-phase-3a-portal.md) → `"Portal"`
- [split-phase-3a-webhook.md](delivery-original-stabilized-split/resources/split-phase-3a-webhook.md) → `"Webhook"`
- [split-phase-3a-email.md](delivery-original-stabilized-split/resources/split-phase-3a-email.md) → `"Email"`
- [split-phase-3a-ftp.md](delivery-original-stabilized-split/resources/split-phase-3a-ftp.md) → `"FTP"`

If you see `HttpPost` / `EMail` / `Ftp` in the Phase 4 summary card, the fix has regressed.

### Phase 6 boolean labels
Historic bug: `useOrder=false` was displayed as "Shared" because the agent reused the Exclusive/Shared labels from `isExclusive`. Fixed in [split-phase-6-delivery-account-summary.md](delivery-original-stabilized-split/resources/split-phase-6-delivery-account-summary.md) by making the template explicitly say "Display all boolean values as Yes/No" and by renaming the label to "Exclusive Delivery". If you see `Shared` in the Order System row or `true`/`false` in any row, the fix has regressed.

### Phase 5 tool chain timing
[split-phase-5-create-delivery-account.md](delivery-original-stabilized-split/resources/split-phase-5-create-delivery-account.md) runs `get_lead_type` → state field detection → `get_usa_states` → `create_delivery_account` in silence after the user sends target states. This can take **3–7 minutes** on the slower model. Do not mistake slowness for a stuck agent. Check `.ai-chat__send.disabled` — if true, it's still processing.

### Criteria gate non-determinism
The Phase 5 criteria gate card ("Add criteria | Continue with state targeting only") in [split-phase-5-create-delivery-account.md](delivery-original-stabilized-split/resources/split-phase-5-create-delivery-account.md) is the single most card-rendering-unreliable prompt in the flow — it frequently renders as text without buttons. The typed fallback is `Add criteria` or `Continue with state targeting only`. If the agent fails to accept the typed fallback and loops, mark the run as FAILED (do not recover).

### Webhook field mapping pipeline
The webhook path in [split-phase-3a-webhook.md](delivery-original-stabilized-split/resources/split-phase-3a-webhook.md) runs a multi-step AI pipeline: input validation → format auto-detection → schema repair → fuzzy field matching → requestBody generation → mapping preview card. This is the **most error-prone single task** in the workflow. Expected failure modes:
- Ambiguous fields auto-resolved without asking the user
- Duplicate mappings (same delivery field mapped to two system fields)
- Nested JSON fields losing dot-notation structure in requestBody
- JSON auto-fix failing on edge cases beyond the two trained errors (single quotes, trailing commas)

### Password is NOT retained across phases
[split-phase-1-create-client.md](delivery-original-stabilized-split/resources/split-phase-1-create-client.md) generates a password via `{generate-password}` for `create_client`. The password is NOT retained. [split-phase-8-activation.md](delivery-original-stabilized-split/resources/split-phase-8-activation.md) regenerates a fresh password for `update_client` activation. Never assume Phase 8 can "remember" the Phase 1 password — it must regenerate.

### `deliveryDays` must survive as a native array
The schedule from Phase 3 is stored as a 7-entry `deliveryDays` array. It must reach `create_delivery_method` as a native object, NOT serialized through a summary. The Phase 3 router [split-phase-3-create-delivery-method.md](delivery-original-stabilized-split/resources/split-phase-3-create-delivery-method.md) does NOT call `summarize_history` before loading the 3a sub-phase for this reason. If you see `deliveryDays` become a string in the session log, that's a regression.

### ChoiceSet + Action.Submit double-submit bug (F1/F14)
An `Input.ChoiceSet` paired with an `Action.Submit` MUST NOT have a `data` field on the submit button, or the submit payload merges the ChoiceSet value with the action's data and double-submits with corrupted values. This is enforced as a global rule in [split-1-global.md](delivery-original-stabilized-split/system/split-1-global.md) `<cards>` block. If you see a double-submit during a dropdown selection, report it with the card's JSON from the offending phase file.

### Three flows, three entry points, three exit behaviors
- `full-setup` [split-create-single-client.md](delivery-original-stabilized-split/action/split-create-single-client.md) → exits after Phase 8 activation
- `add-method` [split-create-delivery-method.md](delivery-original-stabilized-split/action/split-create-delivery-method.md) → exits after Phase 4 method summary
- `add-account` [split-create-delivery-account.md](delivery-original-stabilized-split/action/split-create-delivery-account.md) → exits after Phase 6 account summary

Phase 4 and Phase 6 summary files both check `flowIntent` to decide whether to continue or end. When a flow fails to terminate correctly or continues into a phase it shouldn't, check the `flowIntent` handling in [split-phase-4-delivery-method-summary.md](delivery-original-stabilized-split/resources/split-phase-4-delivery-method-summary.md) and [split-phase-6-delivery-account-summary.md](delivery-original-stabilized-split/resources/split-phase-6-delivery-account-summary.md).

---

## 14. Session lifecycle checklist

### Before starting a batch
- [ ] Confirm the tab is on `/App/#/ai/administration/actions` and user is logged in
- [ ] Load the claude-in-chrome tools you'll need (`tabs_context_mcp`, `javascript_tool`)
- [ ] Decide number of runs (typically 3, 5, or 10)
- [ ] Decide scenario mix (all P, all W, rotated W/P/E/F, etc.)
- [ ] Decide criteria mix (always Add criteria, or rotate with Continue-only)
- [ ] Prepare the results table template (§10)
- [ ] Note the starting run number (never reuse IDs from prior batches — they collide in session logs)

### Between runs
- [ ] Close the chat panel via the X button BEFORE clicking Start Chat on the next row
- [ ] Use sequential run IDs (`StabilityTest-{NN}`) and append scenario letters for mixed batches
- [ ] If a run fails unrecoverably, close the chat and move on — do not lose the batch trying to salvage one run
- [ ] Verify the session ID changed between runs (the chat-details URL should update)

### Inside each run
- [ ] Wait the full expected sleep before polling for the next message (§5)
- [ ] Read state with `javascript_tool`, never screenshots for chat content
- [ ] Never resend while `.ai-chat__send.disabled === true` without confirming via typed character test
- [ ] Log card rendering failures separately (not as outcome failures)
- [ ] DEBUG every anomaly inline before the next phase's `summarize_history`

### After the batch
- [ ] Fill the results table with every metric and outcome
- [ ] Tally issue frequencies (N/total)
- [ ] Classify new anomalies into the 7 LLM failure patterns (§12)
- [ ] Cross-check suspicious entity IDs against the session log
- [ ] Verify in the LeadExec platform that the actual saved data matches what chat showed (per §2 "What to verify on the platform")
- [ ] Report findings in chat — no apologetic preamble, no recap

---

## 15. MCP tools the agent uses (for payload verification)

When reading the session log to verify what the agent actually sent:

**Client:**
- `create_client(createClientDto)` → returns clientUID
- `get_client(clientUID)` → returns companyName, email, clientStatus, timeZoneName, timeOffset
- `get_clients()` → list
- `update_client(clientUID, updateClientDto)` → used for activation

**Lead Types:**
- `display_lead_types_choice` → renders dropdown (auto-fetches + displays)
- `get_lead_types()` → raw list fallback
- `get_lead_type(leadTypeUID)` → returns `leadFields` array
- `get_usa_states()` → returns `[{stateUID, abbr, name}, ...]`

**Delivery:**
- `get_delivery_methods(clientUID)` → list
- `create_delivery_method(clientUID, createDeliveryMethodDto)` → returns deliveryMethodUID
- `create_delivery_account(clientUID, createDeliveryAccountDto)` → returns deliveryAccountUID

**Testing:**
- `test_ftp_sftp_connection(protocol, host, username, password, remotePath)`
- `test_webhook_connection(url, method, contentType, payload, timeoutSeconds)`

**UI / History:**
- `display_adaptive_card(schema)` → Adaptive Card v1.5
- `summarize_history(start_anchor_substring, summarization_text)` → removes messages between anchor and call, appends summary as a system message, accumulates across calls, does NOT touch the system prompt or action file

**Rule:** All DTO parameters are native objects, never JSON strings. If a tool call in the session log has a `createDeliveryAccountDto` field that is a JSON-encoded string instead of an object, that's a regression.

### Session log verification snippets
```js
// Identify which test run a session is (run against session log page body)
[...new Set([...document.body.innerText.matchAll(/StabilityTest-\d+[A-Z]?/g)].map(m=>m[0]))].join(', ')

// Extract create_delivery_account payload
const bodyText = document.body.innerText;
const idx = bodyText.indexOf('create_delivery_account\nArguments:');
bodyText.substring(idx, idx + 1000);

// Decode state UIDs from get_usa_states response (for cross-checking stateCriterion values)
const states = {};
for (const m of bodyText.matchAll(/"stateUID":(\d+),"abbr":"(\w+)"/g)) states[m[1]] = m[2];
states
```

---

## 16. Quick-reference JS snippets

```js
// Get tab context (call once, loads tool first via ToolSearch)
// mcp__claude-in-chrome__tabs_context_mcp

// Message count + last 3 messages
(function(){
  const m = document.querySelectorAll('.ai-chat__message');
  return {count: m.length, last3: Array.from(m).slice(-3).map(x=>x.innerText.trim().substring(0,200))};
})()

// Send button state
(function(){
  const s = document.querySelector('.ai-chat__send');
  const i = document.querySelector('textarea');
  return {disabled: s?.disabled, inputVal: i?.value, inputEmpty: !i?.value};
})()

// Send text conversationally
(function(t){
  const i = document.querySelector('textarea');
  Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype,'value').set.call(i,t);
  i.dispatchEvent(new Event('input',{bubbles:true}));
  setTimeout(()=>document.querySelector('.ai-chat__send').click(),300);
})('CA, TX')

// Click button in last message by exact text
(function(label){
  const msgs = document.querySelectorAll('.ai-chat__message');
  const lm = msgs[msgs.length-1];
  const btn = Array.from(lm.querySelectorAll('button')).find(b=>b.innerText.trim()===label);
  if (btn) btn.click();
  return {clicked: !!btn, available: Array.from(lm.querySelectorAll('button')).map(b=>b.innerText.trim())};
})('Continue')

// Select dropdown option by text and click submit
(function(optText, submitText){
  const msgs = document.querySelectorAll('.ai-chat__message');
  const lm = msgs[msgs.length-1];
  const sel = lm.querySelector('select');
  if (!sel) return 'no select in last message';
  const opt = Array.from(sel.options).find(o=>o.text.trim()===optText);
  sel.value = opt.value;
  sel.dispatchEvent(new Event('change',{bubbles:true}));
  const btn = Array.from(lm.querySelectorAll('button')).find(b=>b.innerText.trim()===submitText);
  setTimeout(()=>btn?.click(),300);
  return 'selected '+optText+', clicked '+submitText;
})('LendingTree','Continue')

// Close floating chat panel
(function(){
  const btns = Array.from(document.querySelectorAll('button, span, i'))
    .filter(el => ['×','X','✕'].includes(el.innerText?.trim()) || el.getAttribute('aria-label')==='Close');
  if (btns[0]) { btns[0].click(); return 'closed'; }
  return 'no close button found';
})()

// Click Start Chat for a specific split action row
(function(actionLabel){
  const row = Array.from(document.querySelectorAll('tr')).find(r => r.innerText.includes(actionLabel));
  if (!row) return 'row not found';
  const btn = Array.from(row.querySelectorAll('button, a')).find(b => b.innerText.trim().toLowerCase().includes('start'));
  if (btn) { btn.click(); return 'clicked Start Chat'; }
  return 'no Start Chat button in row';
})('Create Single Client (Original - Split)')
```

---

## 17. Never-commit rule

**Do NOT create git commits during testing** unless the user explicitly asks. Testing produces findings, not code changes. If a run reveals a bug fix is needed, report it and wait for instructions. Committing without being asked violates the project's workflow convention.

---

## 18. What a good test report looks like

- **Table** with every run against every tracked metric (§10)
- **Frequency column** showing N/total for each issue
- **Severity classification**: Critical / Medium / Cosmetic
- **Defect classification taxonomy tag** per defect: `<sub>IGNORE/HALLUCINATE/PLATFORM/CLOSED/FLAKY/AMBIGUOUS</sub>` (§10)
- **LLM failure pattern classification** (§12 — one of the seven buckets)
- **New bugs** called out separately with reproduction steps, affected runs, exact observed state, and session log URLs
- **Split phase file references** for every regression — e.g. "Phase 4 summary regressed — see [split-phase-4-delivery-method-summary.md](delivery-original-stabilized-split/resources/split-phase-4-delivery-method-summary.md) line N"
- **Platform verification** note: confirm in the LeadExec platform that each created entity has the correct saved data
- **No apologetic preamble**, no recap of what was tested — lead with findings

---

## 19. Split pack file map (quick reference)

**Global & actions:**
- [delivery-original-stabilized-split/system/split-1-global.md](delivery-original-stabilized-split/system/split-1-global.md)
- [delivery-original-stabilized-split/action/split-create-single-client.md](delivery-original-stabilized-split/action/split-create-single-client.md) — full-setup entry
- [delivery-original-stabilized-split/action/split-create-delivery-method.md](delivery-original-stabilized-split/action/split-create-delivery-method.md) — add-method entry
- [delivery-original-stabilized-split/action/split-create-delivery-account.md](delivery-original-stabilized-split/action/split-create-delivery-account.md) — add-account entry

**Phase resources** ([delivery-original-stabilized-split/resources/](delivery-original-stabilized-split/resources/)):
- `split-phase-0-select-client.md` — P0a client selection (add-method flow)
- `split-phase-0-select-client-and-method.md` — P0b client + existing method selection (add-account flow)
- `split-phase-1-create-client.md` — P1 client creation
- `split-phase-2-get-lead-types.md` — P2 lead type dropdown
- `split-phase-3-create-delivery-method.md` — P3 schedule + delivery type router (no `summarize_history` before 3a)
- `split-phase-3a-portal.md` — P3a Portal
- `split-phase-3a-webhook.md` — P3a Webhook + field mapping pipeline
- `split-phase-3a-email.md` — P3a Email
- `split-phase-3a-ftp.md` — P3a FTP
- `split-phase-3b-webhook-test.md` — P3b webhook connection test
- `split-phase-3b-ftp-test.md` — P3b FTP/SFTP connection test
- `split-phase-4-delivery-method-summary.md` — P4 method summary, `flowIntent` branch point
- `split-phase-5-create-delivery-account.md` — P5 price / exclusive / order / states / criteria gate / create account (heavy tool chain)
- `split-phase-5b-criteria-builder.md` — P5b criteria builder loop
- `split-phase-6-delivery-account-summary.md` — P6 account summary, `flowIntent` branch point
- `split-phase-7-client-summary.md` — P7 client summary (full-setup only)
- `split-phase-8-activation.md` — P8 activation (full-setup only)

**Findings log:** [stability-split-findings.md](stability-split-findings.md) — cumulative F-numbered defects for the Split pack. Consult before filing a new defect to avoid duplicates and to check if the same symptom was previously CLOSED or marked FLAKY.


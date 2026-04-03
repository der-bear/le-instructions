# Delivery Rework — Fix Plan

Based on 50 findings (F1-F50) across 4 rounds of live testing + multi-agent evaluation.

---

## Priority 1: F44 — Target States Never Collected (CRITICAL)

**The #1 issue.** Phase 5 Steps 4-6 (load lead fields, detect state field, collect target states) are skipped 100% of the time. The LLM jumps from the last user prompt (Order System) to the next user prompt (Criteria Gate), skipping all silent tool call steps in between.

**What we tried that didn't work:**
- State Machine with IF conditions → LLM evaluates out of order, skips states
- States with Steps inside → LLM skips cross-state to next user prompt
- ONE flat state with Steps 1-10 → LLM still skips silent steps (4-6)

**What the stabilized version does differently:**
- Uses `TOOL: get_lead_type(leadTypeUID)` as a single-line directive — no "Step N" label, no "(silent)" annotation
- Uses `PROCESS:` for inline processing — no explanation
- The tool call is between two `WAIT` directives — the LLM processes top-to-bottom through WAIT→TOOL→WAIT

**Proposed fixes (try in order):**

### Fix A: Remove "(silent — no user prompt)" annotations
The words "silent" and "no user prompt" may be telling the LLM to skip these steps. Remove the annotations and see if the LLM executes them.

Before: `Step 4: Load Lead Fields (silent — no user prompt)`
After: `Step 4: Load Lead Fields`

### Fix B: Add a user-facing message before the tool call
Make Step 4 NOT silent — show "Loading your lead type fields..." before calling the tool. This gives the LLM a user-facing action to execute, making it less likely to skip.

Before: `Step 4: Load Lead Fields (silent — no user prompt) / Call get_lead_type...`
After: `Step 4: Load Lead Fields / Prompt: "Loading lead type configuration..." / Call get_lead_type...`

### Fix C: Use stabilized version's exact directive format
Replace the Step format with the stabilized's terse `TOOL:` / `PROCESS:` / `PROMPT:` directives for the tool call section only.

Before:
```
Step 4: Load Lead Fields (silent — no user prompt)
  Call the get_lead_type(leadTypeUID) tool and retain: leadTypeName, leadFields.
  If the tool fails, prompt: "..." STOP AND YIELD.

Step 5: Detect State Field (silent — no user prompt)
  Detect the state field from leadFields:
  ...
  Retain stateFieldUID.

Step 6: Collect Target States
  Prompt: "Which states do you want to target?"
  STOP AND YIELD.
```

After:
```
TOOL: get_lead_type(leadTypeUID) → retain leadTypeName, leadFields
PROCESS: Detect state field (Priority 1: specialBit → Priority 2: name exact → Priority 3: substring). Retain stateFieldUID.

Prompt: "Which states do you want to target? (e.g., CA, AZ, TX)"
STOP AND YIELD.
Normalize to uppercase USPS codes.
```

### Fix D: Merge target states into Step 3
Ask target states alongside Order System — eliminate the tool call entirely from the main flow. Load lead fields AFTER target states are collected (in the criteria section).

This changes the flow order but avoids the tool call skip entirely.

**Recommended approach:** Try Fix A first (smallest change). If it fails, try Fix C (match stabilized format for tool call sections only).

---

## Priority 2: F46/F49 — Non-Deterministic Card Rendering + Typed Text Regression

**Content type ActionSet sometimes renders as plain text.** When user types response instead of clicking a button, the agent can lose phase context and regress.

**Proposed fix:**
- This may be a platform/model issue that can't be fully fixed via instructions
- Partial mitigation: ensure all ActionSet prompts include the tool name explicitly (`Present using display_adaptive_card with an ActionSet:`)
- Already done in all files — if it still happens, it's model-level non-determinism

**Status:** Partially mitigated. Monitor during testing.

---

## Priority 3: F4 — Table Cards Render as Plain Text

**100% of table cards (Phase 4, 6, 7) render as plain text, not structured tables.** The JSON template is provided but the LLM constructs it incorrectly or doesn't call display_adaptive_card.

**Proposed fix:**
- The stabilized version uses `DISPLAY [adaptive_card] using this template as base:` which works
- Our rework says `Display ... using display_adaptive_card with this template:` — similar but different phrasing
- May need platform-side template renderer for 100% reliability
- Try matching the stabilized version's exact `DISPLAY [adaptive_card]` phrasing

**Status:** Needs investigation. Could be platform issue.

---

## Priority 4: F30 — Ambiguous Fields Auto-Resolved

**Both original and rework auto-resolve ambiguous field mappings.** The instruction says "MUST ask user" but the LLM doesn't.

**Proposed fix:**
- Strengthen the instruction: `CRITICAL: If two or more candidate fields have the SAME match priority, you MUST present an ActionSet for user selection. Do NOT auto-resolve.`
- Already strengthened — if still happening, it's model behavior

**Status:** Low priority. Exists in both versions.

---

## Priority 5: F45 — Duplicate Field Mappings

**Same delivery field ("state") mapped to both ContactState and PropertyState.** A delivery field should map to exactly one system field.

**Proposed fix:**
- Add to webhook State 3 Step 2: "Each delivery field MUST map to exactly ONE system field. If a delivery field appears in multiple nested locations (e.g., lead.contact.address.state and lead.address.state), map only the first occurrence."

**Status:** Minor. Needs wording addition.

---

## Priority 6: F47/F48 — Posting Instructions Auto-Fix + Request Body Format

**Auto-fix for broken posting instructions is too weak.** The agent should try harder to repair JSON/XML before refusing.

**Request body should preserve nested structure with dot notation** for nested JSON fields (e.g., `lead.contact.first_name` not `first_name`).

**Proposed fix:**
- F47: Strengthen the auto-fix instruction to be more aggressive (add missing braces, fix quotes, handle trailing content)
- F48: Add explicit note: "For nested JSON/XML, use dot notation in field names (e.g., lead.contact.first_name)"

**Status:** Minor. Wording improvements.

---

## Already Fixed (Confirmed in Testing)

| Finding | Fix | Verified |
|---------|-----|----------|
| F5 | State→Step within-state ordering | ✓ T1, T8 |
| F24 | Criteria gate ActionSet | ✓ T1, T8 |
| F34 | Webhook auto-detect works | ✓ T2 |
| F42 | UID display reverted to original | ✓ T8 |
| F1/F14 | ChoiceSet Action.Submit rule in global | ✓ Added |

## Cannot Fix Via Instructions (Platform Issues)

| Finding | Description |
|---------|-------------|
| F23 | Action.Submit data visible as user messages |
| F25 | Intermittent message drops |
| F4 | Table card JSON construction (partially) |

## Untested (Need More Testing)

| Finding | What to test |
|---------|-------------|
| F38 | FTP credentials collected before method creation |
| F29 | P5c GLOBAL EXIT RULE at enum ChoiceSet |
| F8 | P5c field suggestions shown first |
| F37 | Broken XML parse failure recovery |

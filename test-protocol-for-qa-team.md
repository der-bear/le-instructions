# Test Protocol — QA Reference

**Product:** LeadExec Delivery Setup Assistant  
**Variants:**
- Create Single Client (Original)
- Create Single Client (Reworked)
- **Create Single Client (Original - Split)** ← primary candidate

**Owner:** QA  
**Environment:** `dev.leadexec.app`

---

## Goal

Verify that the variant under test is faster than production overall while maintaining production-level stability and correctness throughout the full delivery setup flow.

- **Performance:** The variant is expected to be noticeably faster than production across most of the flow.
- **Stability:** The AI must not skip prompts, lose entered values, or ask the user to re-enter something already provided. Stability must be at least as good as production.
- **Platform validation:** Do not rely solely on what the AI displays in chat. After each run, open the LeadExec platform and confirm that all records were actually created with the correct configuration — client record, delivery method (URL, content type, field mapping), and delivery account (price, exclusivity, states, criteria, order system). Any mismatch between what the AI showed and what was actually saved must be logged as a defect.

**Note on expected slower responses:** The webhook field mapping step and delivery account setup use the same slower AI model that production uses for these steps. Longer response times here are normal and are not defects.

**Pass criteria:** All applicable checkpoints marked Y.  
**Fail criteria:** Any checkpoint marked N — log a defect and continue the remaining runs.  
**Cosmetic:** Any display-only issue noted in the C column — does not affect pass/fail.

---

## Prerequisites

- Access to `dev.leadexec.app` with an admin account
- Chrome browser with the LeadExec AI panel extension active
- Session detail log: `App/#/ai/administration/chat-details/view/10/{session_id}`  
  *(Replace `{session_id}` with the ID from the browser URL or dev tools after the run starts)*
- Always start a run via **Start Chat** on the action row in the actions list — never via "+ New" inside the panel

---

## Test Scenarios

Run all four scenarios per round. Append the run number and scenario letter to the company name to tell runs apart (e.g., `StabilityTest-01-W`, `StabilityTest-01-P`).

| ID | Delivery Method | Extra Input Required | Connection Test |
|----|----------------|----------------------|-----------------|
| **W** | Webhook / JSON | Imperfect JSON schema (see Test Data below) | Expected to pass — httpbin returns 200 |
| **P** | Portal | None | Not applicable |
| **E** | Email | Delivery email address | Not applicable |
| **F** | FTP | Host + credentials (see Test Data below) | Expected to fail — fake host, click Skip |

---

## Test Data

### Common Inputs (all scenarios)

| Field | Value |
|-------|-------|
| Company Name | `StabilityTest-{NN}` |
| Contact Email | `test{NN}@test.com` |
| Lead Type | LendingTree |
| Delivery Schedule | Specific hours only — `Mon-Fri 9am-5pm PST` |
| Price per Lead | `35` |
| Exclusivity | Exclusive |
| Order System | No |
| Target States | `CA, TX, FL` |
| Criterion 1 | credit rating excellent |
| Criterion 2 | loan amount at least 50000 |
| Criterion 3 | loan purpose purchase |

### Scenario W — JSON Schema

Paste this verbatim. It contains intentional errors — the AI must fix them silently without asking the user to re-submit.

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

**Errors to confirm were repaired automatically:**
- `'purchase'` — single quotes around the value must become double quotes: `"purchase"`
- `720,` — the trailing comma after the last entry must be removed

**Webhook URL:** `https://httpbin.org/post`

### Scenario F — FTP Credentials

| Field | Value |
|-------|-------|
| Host | `ftp.test.internal` |
| Username | `testuser` |
| Password | `testpass123` |
| Protocol | SFTP |

### Scenario E — Email Address

Use `delivery@test.internal`.

---

## What to Expect per Variant

The same steps may look different depending on which variant you are testing. Refer to this table if something seems unexpected.

| Situation | Original | Reworked | Original - Split |
|-----------|----------|----------|-----------------|
| Selecting the delivery type | A button card is shown | May appear as a plain text question or a button card | A button card is shown |
| Content type selection during Webhook setup | A card appears **after** you paste the schema | A card appears **before** you paste the schema | A card appears **before** you paste the schema |
| How criteria are added | The AI asks for criteria directly — no separate "add criteria" card | A card asks whether you want to add criteria | A card asks whether you want to add criteria |
| After the connection test (Webhook / FTP) | The AI advances automatically — no button to click | A Continue button appears | A Continue button appears |
| Where activation happens | One extra step after the account summary | Immediately after the account summary | Immediately after the account summary |

---

## Step-by-Step Execution

### Step 1 — Create Client
1. Click **Start Chat** on the action row.
2. When prompted, enter the company name and contact email.
3. ✓ Verify: the welcome prompt appears exactly once; the client is created without error.

### Step 2 — Select Lead Type
4. Select **LendingTree** from the dropdown.
5. Click **Continue**.
6. ✓ Verify: lead type selection is shown as a clickable card, not a plain text question.

### Step 3 — Set Schedule
7. Click **Specific hours only**.
8. Type `Mon-Fri 9am-5pm PST`.
9. ✓ Verify: the schedule is accepted and the AI moves on to delivery method selection.

### Step 4 — Set Up Delivery Method
10. Select the delivery method for this scenario (Webhook / Portal / Email / FTP).

**Webhook — additional steps:**
11. Enter `https://httpbin.org/post` as the webhook URL.
12. Click **I'll provide instructions**.
13. If a content type card appears: click **JSON**.
14. Paste the JSON schema from Test Data.
15. If a content type card appears again after pasting: click **JSON**.
16. Click **Continue** on the field mapping preview.
17. ✓ Verify: fields are mapped and shown in a table card; no error is displayed.

**FTP — additional steps:**
11. Enter the host, username, and password from Test Data; select SFTP.

**Email — additional steps:**
11. Enter `delivery@test.internal`.

### Step 5 — Connection Test (Webhook and FTP only)
18. Click **Test Connection**.
    - Webhook: expect a success message → click **Continue**.
    - FTP: expect a failure message → click **Skip**.
19. ✓ Verify: the test prompt appeared exactly once.

### Step 6 — Delivery Method Summary
20. Verify the summary card shows the correct company name, lead type, delivery method, and schedule.
21. Click **Continue**.

### Step 7 — Delivery Account Setup
22. Enter `35` for price per lead.
23. Click **Exclusive**.
24. Click **No** for Order System.
25. Type `CA, TX, FL` for target states.
26. ✓ Verify: the AI asked for states — this step must not be skipped automatically.
27. When a card appears asking whether to add criteria (Reworked / Original - Split), click **Add criteria**. On Original, the AI will ask for criteria directly.

### Step 8 — Add Lead Criteria
28. Type `credit rating excellent` → select **EXCELLENT** from the dropdown → click **Submit**.
29. Click **Add another criterion** → type `loan amount at least 50000` → Submit.
30. Click **Add another criterion** → type `loan purpose purchase` → select **PURCHASE** from the dropdown → Submit.
31. Click **Continue** or **Done adding criteria**.
32. ✓ Verify: all 3 criteria appear on the account summary card or in the session log.

### Step 9 — Account Summary
33. Verify the card shows: Price 35.00 · Exclusivity "Exclusive" · States CA, TX, FL · 3 criteria · Order System "No".
34. Click **Continue**.

### Step 10 — Activation
35. Verify the client summary card shows all fields correctly.
36. Click **Activate**.
37. ✓ Verify: status shows **ACTIVE**.

---

## Checkpoints

For Portal (P) and Email (E) runs, mark checkpoints 4–9 as **N/A**.

| # | Checkpoint | What to verify | Scenarios |
|---|-----------|----------------|-----------|
| 1 | Client Created | Welcome prompt shown exactly once; client record created without error | All |
| 2 | Lead Type | LendingTree selectable via a card or dropdown — not a plain text question | All |
| 3 | Schedule | Schedule phrase accepted; AI moves to delivery method without error | All |
| 4 | Webhook URL | Webhook URL entered and accepted before schema is requested | W |
| 5 | Content Type | A card offering JSON / Form / URL-encoded format choice is shown | W |
| 6 | JSON Repaired | Schema errors corrected automatically — no error message shown to user | W |
| 7 | Mapping Table | Field mapping preview displayed as a table card, not raw text | W |
| 8 | Fields Mapped | Available fields are mapped; count shown in the card | W |
| 9 | Connection Test | Connection test prompt shown once; result (pass or fail) handled correctly | W, F |
| 10 | Method Summary | Summary card shows correct company name, lead type, method, and schedule | All |
| 11 | Price | Price is the first thing asked in account setup | All |
| 12 | Exclusivity | Exclusive / Shared choice shown and saved correctly | All |
| 13 | Order System | Order System Yes / No shown and saved correctly | All |
| 14 | States Asked | AI asks for target states — this step does not get skipped | All |
| 15 | States Saved | States saved as standard 2-letter codes (CA, TX, FL) | All |
| 16 | Criteria Card | A card appears asking whether to add criteria after the account is created | All |
| 17 | Criterion 1 | First criterion (credit rating) entered and accepted | All |
| 18 | Enum Dropdown | For credit rating and loan purpose, a dropdown is shown and the selection is saved correctly | All |
| 19 | All 3 Criteria Added | Third criterion accepted without the AI stopping the flow early | All |
| 20 | Criteria Ended Correctly | Criteria input ended only after Continue was clicked — not automatically | All |
| 21 | Labels Readable | Account summary shows "Exclusive" (not "true") and "No" (not "false") | All |
| 22 | Account Card Correct | Summary card shows all fields: price, exclusivity, states, 3 criteria, order system | All |
| 23 | Activated | Activation completed; status shows ACTIVE | All |
| C | Visual Issues | Count of display-only problems that did not affect saved data. Examples: AI asks you to type "yes" instead of showing a button; prompt shown twice; button label garbled; status shown as "true/false" instead of readable text. Enter 0 if none. | All |

---

## Defect Reporting

When a checkpoint fails, log a defect with:

| Field | Content |
|-------|---------|
| **ID** | `R{NN}-{scenario}-F{N}` (e.g., `R03-W-F01`) |
| **Checkpoint** | Which checkpoint failed |
| **Symptom** | What was observed |
| **Expected** | What should have happened |
| **Data impact** | Was data lost, incorrect, or display-only? |
| **Session ID** | From browser URL or dev tools |
| **Reproduced in** | List other runs with the same failure |
| **Status** | New / Flaky / Fixed |

---

## Scoring Matrix

**Variant:** _________________ &nbsp;&nbsp; **Scenario:** _________________ &nbsp;&nbsp; **Round:** _________________

Y = pass · N = fail · N/A = not applicable · C = cosmetic issue count (0 = none).

| # | Checkpoint | 01 | 02 | 03 | 04 | 05 | 06 | 07 | 08 | 09 | 10 |
|---|-----------|----|----|----|----|----|----|----|----|----|----|
| 1 | Client Created | | | | | | | | | | |
| 2 | Lead Type | | | | | | | | | | |
| 3 | Schedule | | | | | | | | | | |
| 4 | Webhook URL | | | | | | | | | | |
| 5 | Content Type | | | | | | | | | | |
| 6 | JSON Repaired | | | | | | | | | | |
| 7 | Mapping Table | | | | | | | | | | |
| 8 | Fields Mapped | | | | | | | | | | |
| 9 | Connection Test | | | | | | | | | | |
| 10 | Method Summary | | | | | | | | | | |
| 11 | Price | | | | | | | | | | |
| 12 | Exclusivity | | | | | | | | | | |
| 13 | Order System | | | | | | | | | | |
| 14 | States Asked | | | | | | | | | | |
| 15 | States Saved | | | | | | | | | | |
| 16 | Criteria Card | | | | | | | | | | |
| 17 | Criterion 1 | | | | | | | | | | |
| 18 | Enum Dropdown | | | | | | | | | | |
| 19 | All 3 Criteria Added | | | | | | | | | | |
| 20 | Criteria Ended Correctly | | | | | | | | | | |
| 21 | Labels Readable | | | | | | | | | | |
| 22 | Account Card Correct | | | | | | | | | | |
| 23 | Activated | | | | | | | | | | |
| C | Visual Issues | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| — | **Score** | /23 | /23 | /23 | /23 | /23 | /23 | /23 | /23 | /23 | /23 |
| — | **Defects** | | | | | | | | | | |

---

*Alex D.*

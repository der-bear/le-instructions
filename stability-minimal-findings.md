# Stability Test Findings — Version C: Minimal

**Action:** Create Single Client (Minimal)
**Model:** gpt-5.4-mini (all phases)
**Protocol:** stability-test-protocol.md (adapted for minimal variant)
**Date:** 2026-04-06
**Lead Type:** LendingTree (all runs)
**Base test data:** StabilityTest-{RR}m, stability{RR}m@test.com

---

## Run Scenarios

| Run | Scenario | Delivery | Content Type | Schedule | Criteria | Special |
|-----|----------|----------|-------------|----------|----------|---------|
| 01 | JSON imperfect baseline | Webhook | JSON (missing braces, trailing comma, single quotes) | Mon-Fri 9-5 PST | 3: numeric >=, enum, numeric > | LendingTree |
| 02 | JSON nested complex | Webhook | JSON (deep nesting, 15+ fields) | Tue-Thu 9-6 EST | 5: 2 enum + 2 numeric + 1 between | Exclusive, Order ON |
| 03 | Content mismatch XML→JSON | Webhook | XML → switch → JSON | 24/7 | 2 enum + 1 numeric | Recovery flow |
| 04 | Content mismatch JSON→XML | Webhook | JSON → switch → XML | Mon-Fri 8-6 CST | 3: enum + numeric + between | Reverse mismatch |
| 05 | URL Encoded + enum-heavy | Webhook | URL Encoded | 24/7 | 4: 3 enum + 1 numeric | Shared |
| 06 | Auto-detect (XML body) | Webhook | "I'm not sure" | Mon-Wed-Fri 8-8 PST | 2 enum | Exclusive |
| 07 | Auto-detect (JSON body) | Webhook | "I'm not sure" | 24/7 | 3: numeric >= + enum + numeric > | Shared, Order ON |
| 08 | FTP + mixed criteria | FTP | N/A | Mon-Fri 9-5 PST | 2: 1 enum + 1 numeric | Connection test |
| 09 | Portal + skip criteria | Portal | N/A | Specific hours | Skip | No conn test |
| 10 | Email + no criteria | Email | N/A | 24/7 | Skip | Shared |

---

## Scoring Matrix

| Run | P1-PROMPT | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 02 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 03 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 04 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 05 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 06 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 07 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 08 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 09 | | | | | | | | | | | | | | | | | | | | | | | | | |
| 10 | | | | | | | | | | | | | | | | | | | | | | | | | |

---

## Findings Catalog

*(Populated during testing)*

---

## Known Structural Differences (Minimal vs Others)

Minimal has fewer phases than stabilized/rework:
- No Phase 4 (delivery method summary) — goes directly from Phase 3 to account setup
- No Phase 6 (delivery account summary) — goes directly to activation
- Phase numbering: P1 → P2 → P3 → P3-type → P4 (account) → P4b (criteria) → P5 (activation)
- Uses `<next_instructions>` XML-style handoffs (like stabilized)

Checkpoints P4-SUMM and P6-SUMM may be N/A depending on minimal's phase structure.

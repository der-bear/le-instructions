# Stability Test Findings — Production

**Action:** Create Single Client (Production)
**Domain:** `leadexec.clickpointsoftware.com`
**Protocol:** stability-test-protocol.md (adapted: 10 runs single round)
**Base test data:** StabilityProd-{RR}, stabilityprod{RR}@test.com
**Lead Type:** LendingTree (LeadTypeUID: 5689)
**Date:** 2026-04-07

## Purpose

Measure production-version stability and compare against dev-environment rounds (Stabilized R1a/R1b/R2/R2c, Rework R1a/R1b/R2/R2c). Production may run a different (and possibly older) instruction variant — findings here characterize what end users actually experience.

## Rounds — Single Source of Truth

| Round | Runs | Model Configuration | Instructions | Scoring | n | PASS | Avg |
|-------|------|---------------------|--------------|---------|---|------|-----|
| **PR1** | 01-10 | Production default | Production (variant TBD via DEBUG) | Audited (F* deprioritized) | 10 | TBD | TBD |

**Cosmetic vs Functional Failure Convention (matches R2c onwards):**
- **F*** = cosmetic failure (doubled prompt, plain-text fallback for card, ordering quirk) — workflow still completes correctly. NOT counted in score percentage.
- **F** = functional failure (skipped step, lost data, wrong value, hallucinated value, never-created entity). Counted in score.

---

## Run Scenarios

| Run | Scenario | Delivery | Content Type | Schedule | Criteria | Special |
|-----|----------|----------|--------------|----------|----------|---------|
| 01 | JSON imperfect baseline | Webhook | JSON (missing braces, trailing comma, single quotes) | Mon-Fri 9-5 PST | 3: numeric >=, enum, numeric > | LendingTree |
| 02 | JSON nested complex | Webhook | JSON (deep nesting, 15+ fields) | Tue-Thu 9-6 EST | 5: 2 enum + 2 numeric + 1 between | Exclusive, Order ON |
| 03 | Content mismatch XML→JSON | Webhook | XML → switch → JSON | 24/7 | 2 enum + 1 numeric | Recovery flow |
| 04 | URL Encoded + enum-heavy | Webhook | URL Encoded | 24/7 | 4: 3 enum + 1 numeric | Shared |
| 05 | Auto-detect (XML body) | Webhook | "I'm not sure" | Mon-Wed-Fri 8-8 PST | 2 enum | Exclusive |
| 06 | FTP + mixed criteria | FTP | N/A | Mon-Fri 9-5 PST | 2: 1 enum + 1 numeric | Connection test |
| 07 | Portal + skip criteria | Portal | N/A | Mon-Fri 9-5 EST | Skip | No conn test |
| 08 | Email + no criteria | Email | N/A | 24/7 | Skip | Shared |
| 09 | Webhook nested + numeric | Webhook | JSON nested | Mon-Fri 8-6 CST | 3 numeric | Exclusive |
| 10 | Webhook + all enum | Webhook | JSON | Mon-Fri 9-5 PST | 3 enum | Shared |

---

## Combined Scoring Matrix

| Run | Round | P1-PROMPT† | P2-DROP | P3-SCHED | P3-WURL | P3-JSON | P3-TABLE | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-NORM | P5-FIELD | P5-CR1 | P5-ENUM | P5-CR3 | P5-DONE | P6-BOOL | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-------|-----------|---------|----------|---------|---------|----------|----------|----------|---------|----------|---------|----------|----------|---------|----------|--------|---------|--------|---------|---------|---------|---------|--------|--------|-------|
| 01 | PR1 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N/A | Y | Y | Y | 22/22 (100%) PASS | LendingTree, Webhook/JSON imperfect (single quotes/trailing comma/no outer braces all auto-fixed); Mon-Fri 9-5 PST; Shared, Order OFF; CA/TX/FL normalized ✓; 10/10 mapping with 3-field disambiguation (PropertyState, RequestAssignmentDate, FilterName); 3 criteria added (LoanAmount>=250k, LoanRequestType=Purchase, PropertyValue>350k); Method:40627, Acct:49867; ACTIVE ✓; **PRD-A**: chat panel briefly disappeared after CR3 Continue click — user re-opened, full state restored without loss (UI glitch, not session loss); **PRD-B**: enum criterion (LoanRequestType) auto-resolved to "Purchase" without ChoiceSet (different from rework which always requires ChoiceSet) |

† P1-PROMPT marked F* for cosmetic prompt duplication (RB-G class) — does not affect functional flow.

---

## Findings Catalog (Production)

(Populated as runs complete and findings emerge.)

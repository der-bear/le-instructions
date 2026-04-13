# Stability Test Findings — Production R2

**Action:** Create Single Client (Production)
**Domain:** `leadexec.clickpointsoftware.com`
**Variant:** `delivery-original-stabilized-split` (split variant — deployed to production for this round)
**Base test data:** StabilityProd-R2-{RR}, stabprodr2{RR}@test.com
**Lead Type:** LendingTree (LeadTypeUID: 5689)
**Date:** 2026-04-10

## Purpose

Continue production stability testing from PR1 (4 runs, 2026-04-07). 20 additional runs (10+10) to measure production-variant stability, validate known defects (PRD-LOOP, PRD-B), and discover new production-specific issues.

## Production Variant Note

**Production now runs the split variant** (`delivery-original-stabilized-split`) as of this round. This means:
- `deliveryTypeDisplay` fix IS present → Phase 4 correctly shows "Webhook"/"Portal"/"Email"/"FTP"
- `responseSearch` SUGGEST step IS present
- Explicit boolean label rules in Phase 6 ARE present ("Display all boolean values as Yes / No")
- Strengthened WAIT definitions ARE present
- Split phase-per-file architecture with `DELIVERY_SETUP_START` anchor
- Criteria gate uses split format ("Would you like to add additional lead criteria...")

PR1 ran `/delivery` (original baseline). PR2 runs split. Direct comparison between rounds reflects both the variant change AND production environment differences (model, latency, scale).

## Run Scenarios

| Run | Scenario | Delivery | Content Type | Schedule | Criteria | Special |
|-----|----------|----------|--------------|----------|----------|---------|
| 01 | JSON nested baseline | Webhook | JSON (nested, 10+ fields) | Mon-Fri 9-5 PST | 3: 1 enum + 2 numeric | LendingTree |
| 02 | URL Encoded + enum heavy | Webhook | URL Encoded | 24/7 | 3: 2 enum + 1 numeric | Shared |
| 03 | XML schema | Webhook | XML | Mon-Fri 8-6 EST | 2: 1 enum + 1 numeric | Exclusive |
| 04 | JSON imperfect (single quotes, trailing comma) | Webhook | JSON | Tue-Thu 9-5 PST | 3: 1 enum + 1 numeric + 1 between | Order ON |
| 05 | Auto-detect format | Webhook | "I'm not sure" → JSON | 24/7 | 2 enum | Shared |
| 06 | Portal + skip criteria | Portal | N/A | Mon-Fri 9-5 PST | Skip | No conn test |
| 07 | FTP + mixed criteria | FTP | N/A | Mon-Fri 9-5 EST | 2: 1 enum + 1 numeric | Connection test |
| 08 | Email + no criteria | Email | N/A | 24/7 | Skip | Shared |
| 09 | JSON flat + numeric only | Webhook | JSON (flat, 8 fields) | Mon-Wed-Fri 8-8 PST | 3 numeric | Exclusive |
| 10 | Webhook skip mapping | Webhook | N/A (skip) | 24/7 | 3: 2 enum + 1 numeric | Skip mapping |

---

## Combined Scoring Matrix

| Run | P1-PROMPT | P2-DROP | P3-SCHED | P3-TYPE | P3-MAP | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-FIELD | P5-CR1 | P5-CR2 | P5-CR3 | P5-DONE | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-----------|---------|----------|---------|--------|----------|----------|---------|----------|---------|----------|----------|----------|--------|--------|--------|---------|---------|---------|--------|--------|-------|
| 01 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | Y | Y | Y | 19/22 (86%) | JSON auto-detect path (typed fallback skipped contentType card); 8/8 mapped; conn test PASS; PRD-LOOP: 2/3 criteria; PRD-R2-ENUM: empty ChoiceSet; P6 correct (Yes/No per split rules) |
| 02 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 18/20 (90%) | URL Encoded 10/10 mapped; conn test buttons DEAD (typed skip); PRD-LOOP: 1/3 criteria; P6 correct; Method:40649, Acct:49882 |
| 03 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Portal; 24/7; skip criteria; P6 correct; Method:40652, Acct:49884 |
| 04 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Email; 24/7; Shared; skip criteria; P6 correct; Method:40653, Acct:49885 |
| 07 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | N/A | Y | Y | Y | 13/15 (87%) | Webhook JSON flat 8/8 mapped; conn test skipped; STATES SKIPPED + HALLUCINATED from example ("CA, AZ, TX"); skip criteria; Method:40656, Acct:49887 |
| 08 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 14/16 (88%) | Portal; 24/7; Exclusive; PRD-LOOP: 1/3 criteria; criteria gate buttons DEAD (typed); Method:40657, Acct:49888 |
| 09 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip mapping; Specific hrs ✓; Shared+Order ON; skip criteria; P6 correct; Method:40658, Acct:49889 |
| 10 | Y | Y | Y | Y | Y | Y | skip | F* | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook JSON imperfect 9/9 mapped; format re-detect (PRD-R2-REDETECT); P4 text-only no card; excl/order typed (no buttons); skip criteria; Method:40659, Acct:49890 |
| 11 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 14/16 (88%) | Portal; 24/7; Exclusive+Order ON; PRD-LOOP: 1/3 criteria; criteria gate buttons DEAD (typed); Method:40660, Acct:49891 |
| 12 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | Y | Y | Y | 13/15 (87%) | Webhook URL-encoded 8/8 mapped; 24/7; Shared; criteria gate SKIPPED; Method:40661, Acct:49892 |
| 13 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | Y | Y | Y | 13/14 (93%) | Portal; 24/7; Shared; criteria gate SKIPPED; P6 correct; Method:40662, Acct:49893 |
| 15 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 14/16 (88%) | Portal; 24/7; Exclusive+Order ON; PRD-LOOP: 1/3 criteria; Method:40664, Acct:49895 |
| 17 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip mapping; 24/7; Shared; skip criteria (gate appeared with buttons ✓); Method:40665, Acct:49896 |
| 19 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Portal; 24/7; Shared; skip criteria (gate buttons ✓); Method:40668, Acct:49899 |
| 20 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 19/22 (86%) | Webhook JSON nested 9/9 mapped; 24/7; Exclusive; PRD-LOOP: 1/3 criteria; Method:40669, Acct:49900 |
| 21 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Email; 24/7; Shared; skip criteria (gate buttons ✓); Method:40671, Acct:49902 |
| 22 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | Webhook skip mapping; 24/7; Exclusive+Order ON; **3/3 criteria** (first full collection!); enum ChoiceSet empty but value accepted; Method:40672, Acct:49903 |
| 23 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 14/16 (88%) | Portal; 24/7; Shared; PRD-LOOP: 1/3 criteria; gate buttons dead; Method:40673, Acct:49904 |
| 25 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Portal; 24/7; Exclusive; skip criteria (gate buttons ✓); Method:40675, Acct:49905 |
| 26 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | Y | Y | 20/22 (91%) | Webhook JSON 9/9; auto-repair FAILED (re-paste clean); re-detect; 2/3 criteria (CR3 dropped); P6 auto-progressed; Method:40676, Acct:49906 |
| 27 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 19/22 (86%) | Webhook JSON 9/9; no re-detect; P4 btns dead (typed); PRD-LOOP 1/3; Method:40677, Acct:49907 |
| 28 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 19/22 (86%) | Webhook URL-encoded 11/11; PRD-LOOP 1/3; Method:40678, Acct:49908 |
| 29 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | Y | - | F | Y | Y | Y | 20/22 (91%) | Webhook JSON nested 9/9; re-detect; 2/3 criteria (CR2 false error but accepted); Method:40679, Acct:49909 |
| 30 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | - | F | Y | Y | Y | 20/22 (91%) | Webhook skip mapping; 2/3 criteria (loop continued to CR2, typed done); gate buttons dead; Method:40681, Acct:49910 |
| 34 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | Y | - | F | Y | Y | Y | 20/22 (91%) | Webhook JSON 9/9; 2/3 criteria; loop continued to CR2; Method:40684, Acct:49913 |
| 35 | Y | Y | Y | Y | Y | Y | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 19/22 (86%) | Webhook JSON nested 10/10 (tool error→skip); re-detect; PRD-LOOP 1/3; CR2 "field not found"; Method:40685, Acct:49914 |
| 37 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | F | Y | Y | 12/15 (80%) | Webhook skip; gate SKIPPED; states showed "CA\|TX\|FL" (pipe-delimited); Method:40687, Acct:49915 |
| 40 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | - | F | F | Y | Y | 18/22 (82%) | Webhook skip; 2/3 criteria collected but LOST in P6 (None); "done" rejected, Continue clicked; Method:40690, Acct:49917 |
| 41 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | - | - | - | - | - | - | - | - | - | FAIL | Webhook skip; P5 Excl→No → agent REGRESSED to P2 lead type; corrupted |
| 43 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | Y | Y | Y | 19/22 (86%) | Webhook skip; PRD-LOOP 1/3; all P5 steps collected; Method:40693, Acct:49919 |
| 44 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | F | Y | Y | 13/14 (93%) | Webhook skip; skip btns DEAD (typed); skip criteria; P6 states "CA\|FL\|TX" (pipe); Method:40694, Acct:49920 |
| 45 | Y | Y | Y | Y | N/A | N/A | skip | F* | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip; P4 text-only; skip btns DEAD (typed); skip criteria; P6 correct; Method:40695, Acct:49921 |
| 46 | Y | Y | Y | Y | N/A | N/A | skip | F* | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip; skip btns DEAD (typed); raw IDs exposed before P6; skip criteria; P6 correct; Method:40696, Acct:49922 |
| 47 | Y | Y | Y | Y | N/A | N/A | skip | F* | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip; all typed (no btns); skip criteria; P6 correct; Method:40697, Acct:49923 |

### Round 5 — GPT-5-mini criteria phase (dev mirror)
**Change:** Criteria building phase swapped to GPT-5-mini (slower model). All other phases unchanged.
**Domain:** `dev.leadexec.app` (production mirror, same split instructions)
**Action:** Create Single Client (Split)
**Plan:** 10 regular + 10 webhook+criteria focused

| Run | P1-PROMPT | P2-DROP | P3-SCHED | P3-TYPE | P3-MAP | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-FIELD | P5-CR1 | P5-CR2 | P5-CR3 | P5-DONE | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-----------|---------|----------|---------|--------|----------|----------|---------|----------|---------|----------|----------|----------|--------|--------|--------|---------|---------|---------|--------|--------|-------|
| 48 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | F | N/A | N/A | N/A | N/A | Y | Y | Y | 13/14 (93%) | Webhook skip; gate SKIPPED (GPT-5-mini); P6 correct; Method:47091, Acct:46077 |
| 49 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | F | - | - | F | - | - | - | FAIL | Portal; GPT-5-mini criteria: can't fuzzy match fields ("loan amount"→fail, need exact "LoanAmount"); "done" doesn't exit loop; exposed raw API params; unrecoverable |
| 50 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | Webhook skip; GPT-5-mini: **3/3 criteria!** Needs exact field names; "continue" doesn't exit loop (used "no more criteria"→"yes"); all criteria preserved in P6; Method:47094, Acct:46080 |
| 51 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | F | Y | Y | F | - | F | Y | Y | Y | 16/22 (73%) | Portal; GPT-5-mini: STATES SKIPPED; 1/3 in P6 (CR2 acknowledged not saved); P6 rendered before criteria done; Method:47095, Acct:46081 |
| 52 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip; skip criteria (gate appeared); P6 correct; Method:47097, Acct:46083 |
| 53 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | F | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 13/14 (93%) | Portal; STATES SKIPPED + hallucinated "AZ, CA, TX" from example; skip criteria; P6 otherwise correct; Method:47098, Acct:46084 |
| 54 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | F | Y | Y | Y | - | F | Y | Y | Y | 18/22 (82%) | Webhook skip; STATES SKIPPED (empty in P6); 2/3 criteria preserved; "no more criteria"→"yes proceed" to exit loop; Method:47099, Acct:46085 |
| 55 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Portal; Shared+Order ON; skip criteria (gate appeared); P6 correct; Method:47100, Acct:46086 |
| 56 | Y | Y | Y | Y | N/A | N/A | skip | Y | F* | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip; price skipped→re-asked (P5 regression→recovery); skip criteria; P6 correct; Method:47101, Acct:46087 |
| 57 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | F | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 13/14 (93%) | Webhook skip; STATES SKIPPED + hallucinated "CA" only; skip criteria; Method:47102, Acct:46088 |
| 58 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | - | - | F | F | Y | Y | 17/22 (77%) | Webhook skip; GPT-5-mini: operator asked separately; CR1 accepted then gate REGRESSED; criteria lost (None in P6); "no more criteria" ineffective; Method:47103, Acct:46090 |
| 59 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Portal; skip criteria (gate appeared); all P5 collected; P6 correct; Method:47104, Acct:46091 |
| 60 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | F | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 13/14 (93%) | Webhook skip; STATES SKIPPED (shows "not provided" — no hallucination); skip criteria; Method:47105, Acct:46092 |
| 61 | Y | Y | Y | Y | N/A | N/A | skip | Y | F* | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | Webhook skip; price skipped→re-asked (P5 regression→recovery); skip criteria; P6 correct; Method:47106, Acct:46093 |
| 62 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | Webhook skip; GPT-5-mini: **3/3 criteria!** All exact field names; loop exit "no more criteria"; NEW: agent hallucinated 2 extra prompts after criteria exit (account name + status — should be TOOL_DEFAULTS); P6 all criteria preserved; Method:47107, Acct:46094 |
| 63 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | - | - | - | - | - | - | - | - | - | FAIL | Webhook skip; P5 Shared→No → agent REGRESSED to P2 lead type then P3 schedule; same pattern as R41; corrupted |
| 64 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | 20/22 (91%) | Webhook skip; GPT-5-mini: **3/3 criteria** (auto-exited loop after CR3); P6 empty values (price, excl, states, order); P7 raw {deliveryAccountUID} placeholder; criteria preserved in raw format; Method:47109 |
| 65 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | - | - | - | - | - | - | - | - | - | FAIL | Webhook skip; P5 Excl→No → agent REGRESSED to P2 lead type; same as R41/R63; GPT-5-mini P5 regression pattern |
| 66 | Y | Y | Y | Y | N/A | N/A | skip | F* | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | F | Y | 20/22 (91%) | Webhook skip; P4 minimal (company only); criteria gate no buttons (typed); GPT-5-mini: **3/3 criteria** (auto-exited); P6 SKIPPED entirely; P7 correct with IDs; Method:47111, Acct:46095 |
| 67 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | Webhook skip; GPT-5-mini: **3/3 criteria!** "no more criteria"→"continue" to exit; P6 ALL fields correct; criteria gate no btns (typed); Method:47112, Acct:46096 |
| 68 | Y | Y | Y | Y | N/A | N/A | skip | F* | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | 20/22 (91%) | Webhook skip; P4 text format; STATES SKIPPED+HALLUCINATED "CA"; URL re-asked; field mapping no btns; GPT-5-mini: **3/3 criteria**; P6 all fields but states wrong; Method:47113, Acct:46097 |
| 69 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | F | F | Y | Y | Y | Y | Y | Y | Y | Y | Y | 20/22 (91%) | Webhook skip; EXCL+ORDER SKIPPED (price→states directly); GPT-5-mini: **3/3 criteria**; tool failed (missing name/status)→recovered; P6 all fields+criteria; exposed raw IDs; Method:47114, Acct:46098 |
| 70 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | 21/22 (95%) | Redo R41; Webhook skip; **3/3 criteria** (explicit confirmations); hallucinated name+status (PRD-R2-HALLPROMPT); P6 SKIPPED; P7 correct; Method:47115, Acct:46099 |
| 71 | Y | Y | Y | Y | N/A | N/A | skip | F* | Y | Y | Y | F | Y | Y | - | - | F | F* | Y | Y | 16/22 (73%) | Redo R49; schedule btns dead (typed); STATES SKIPPED; 1/3 criteria (PRD-LOOP+confirm step); agent lost context after CR1; P6 text+raw UIDs; Method:47116, Acct:46101 |
| 72 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | F | Y | Y | Y | Y | Y | Y | Y | Y | 21/22 (95%) | Redo R51; STATES SKIPPED (P6 shows "shared" — hallucinated from Shared answer); **3/3 criteria** with running list; P6 all fields+criteria correct; Method:47117, Acct:46102 |
| 73 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | Redo R58; **3/3 criteria**; states ✓; P6 ALL fields correct (CA/TX/FL, 3/3 criteria); leaked internal instruction text; exposed raw UIDs before P6; Method:47118, Acct:46103 |
| 74 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | F | Y | Y | 21/22 (95%) | Redo R71/R49; **3/3 criteria**; states ✓; URL re-asked; P6 SKIPPED (agent asked to proceed directly to P7); P7 correct; Method:47119, Acct:46104 |

### Model Configuration Summary

| Config | Rounds 1-4 | Round 5 | Round 6 |
|--------|-----------|---------|---------|
| **Base model** | GPT-5.4-mini (medium effort) | GPT-5.4-mini (medium effort) | GPT-5.4-mini (**high** effort) |
| **Criteria/webhook model** | GPT-5.4-mini (high effort) | GPT-5-mini (medium effort) | GPT-5-mini (medium effort) |

**Key attribution:** States skip and P5 regression happen in the base model phase (GPT-5.4-mini), NOT in the criteria model. The worsening from 3% → 30% states skip between Rounds 1-4 (prod) and Round 5 (dev) is a GPT-5.4-mini issue on the dev environment, not attributable to GPT-5-mini.

Only criteria collection improvement (7% → 88% for 3/3) and post-criteria issues (P6 context loss, hallucinated prompts) are attributable to the GPT-5-mini criteria builder swap.

Round 6 changes the base model effort from medium → high to test if higher effort resolves states skip and P5 regression.

### Round 6 — GPT-5.4-mini high effort base (dev mirror)
**Change:** Base model effort raised from medium → high. Criteria builder unchanged (GPT-5-mini medium).
**Domain:** `dev.leadexec.app` (production mirror, same split instructions)
**Action:** Create Single Client (Split)
**Plan:** 10 general + 10 webhook+criteria focused
**Date:** 2026-04-13

| Run | P1-PROMPT | P2-DROP | P3-SCHED | P3-TYPE | P3-MAP | P3-COUNT | P3B-TEST | P4-SUMM | P5-PRICE | P5-EXCL | P5-ORDER | P5-STATE | P5-FIELD | P5-CR1 | P5-CR2 | P5-CR3 | P5-DONE | P6-SUMM | P7-SUMM | P8-ACT | RESULT | Notes |
|-----|-----------|---------|----------|---------|--------|----------|----------|---------|----------|---------|----------|----------|----------|--------|--------|--------|---------|---------|---------|--------|--------|-------|
| 75 | Y* | Y | F* | Y | N/A | N/A | skip | F | F | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | F* | Y | 11/14 (79%) | HIGH EFFORT first run; IGNORE: P1 verbose (client UID exposed); IGNORE: schedule after type (reversed); IGNORE: responseSearch skipped; IGNORE: P4 skipped; IGNORE: price skipped (defaulted 25); states pipe-delimited; P7 missing IDs |
| 76 | Y | Y | Y | Y | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | STALL | HIGH EFFORT: schedule+type correct order; agent stalled 100s+ after webhook URL; platform timeout; IGNORE: high effort processing too slow |
| 77 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: all steps in correct order; absolute clean pass; Method:47121, Acct:46106 |
| 78 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: all steps correct order; P6 all fields correct; P7 with IDs; absolute clean pass; Method:47122, Acct:46107 |
| 79 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: Portal; all steps correct; P6 all fields correct; P7 with IDs; absolute clean pass; Method:47123, Acct:46108 |
| 80 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: Webhook skip; all steps correct; P6 all fields correct (price 45); P7 with IDs; absolute clean pass; Method:47124, Acct:46109 |
| 81 | Y | Y | Y | Y | N/A | N/A | N/A | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: Email; all steps correct; P6 all fields correct (price 18); P7 with IDs; absolute clean pass; Method:47125, Acct:46110 |
| 82 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: Webhook skip; all steps correct; P6 all fields correct (price 30, Shared); P7 with IDs; absolute clean pass; Method:47126, Acct:46111 |
| 83 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: FTP; all steps correct; P6 all fields correct (price 15, Excl); P7 with IDs; absolute clean pass; Method:47127, Acct:46112 |
| 84 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | N/A | N/A | N/A | N/A | Y | Y | Y | 14/14 (100%) PASS | HIGH EFFORT: Webhook skip; all steps correct; P6 all fields correct (price 27, Excl); P7 with IDs; absolute clean pass; Method:47128, Acct:46113 |
| 85 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | - | - | - | - | - | - | - | STALL | HIGH EFFORT: all P5 steps correct; criteria builder loaded; GPT-5-mini stalled 90s+ after CR1; platform timeout on criteria phase |
| 86 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | - | - | - | - | - | - | - | STALL | HIGH EFFORT: all P5 steps correct; criteria builder loaded; send failure (not GPT-5-mini timeout); same pattern as R85 |
| 87 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | HIGH EFFORT + GPT-5-mini criteria: **3/3 criteria!** All P5 steps correct; states pipe-delimited CA\|TX\|FL; P6 all fields+criteria; P7 with IDs; Method:47131, Acct:46116 |
| 88 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | F | F | F | F | Y | 16/22 (73%) | HIGH EFFORT + GPT-5-mini criteria: 2/3 criteria (LoanAmount, SelfCreditRating); PRD-R2-LOOP: no 3rd criterion prompt, jumped to P6; P6 context loss: price/excl/states/order all "Not provided"; P7 {deliveryAccountUID} unreplaced; 2 send failures on criteria (infra, not model); Method:47132 |
| 89 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | - | - | - | PLATFORM | HIGH EFFORT + GPT-5-mini criteria: **3/3 criteria!** All collected with proper loop; PRD-R2-HALLPROMPT: asked for acct name+status after criteria exit; session expired during P6 transition; send failures on CR1+CR2 (infra); platform error+retry on CR1 |
| 90 | Y | Y | Y | Y | N/A | N/A | skip | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | 22/22 (100%) PASS | HIGH EFFORT + GPT-5-mini criteria: **3/3 criteria** (+ dup SelfCreditRating from send failures = 4 total); P6 all fields retained (price 35, Excl Yes, states CA/TX/FL, Order No); P7 with IDs; send failure on CR2 (infra); Method:47134, Acct:46119 |

---

## Findings Catalog (Production R2)

### PRD-R2-ENUM — Empty ChoiceSet for Enumerated Field
**Runs:** 01 | **Classification:** PLATFORM | **Severity:** HIGH
Agent rendered ChoiceSet dropdown for PropertyType enum field but the options were empty — only placeholder "Select a value" and a single empty-value "Choose one" option. The actual enum values (SingleFamily, MultiFamily, Condo, etc.) were not populated. Agent accepted the empty selection and recorded "PropertyType pending value selection" in the summary. Classification: PLATFORM (dropdown populated but choices array empty).

### PRD-R2-LOOP — Criteria Loop Premature Exit
**Runs:** 01 (2/3), 02 (1/3), 08 (1/3), 11 (1/3), 15 (1/3), 20 (1/3), 22 (3/3 ✓), 23 (1/3) | **Classification:** IGNORE | **Severity:** HIGH
Criteria loop exits before all planned criteria are collected. 7/8 runs exit early (88%). Only R22 collected all 3 criteria. Typical: exits after 1st criterion. Split variant's CRITICAL guards insufficient.

### PRD-R2-DEADBTN — Card Buttons Non-Responsive
**Runs:** 02 (conn test), 08 (criteria gate), 10 (excl+order), 11 (criteria gate) | **Classification:** PLATFORM | **Severity:** Medium
Adaptive card buttons render visually but clicks don't register. Send button enables with typed text, confirming agent is waiting for input (not processing). Typed fallback works. Classification: PLATFORM (card rendering non-determinism).

### PRD-R2-STATESKIP — States Prompt Skipped + Values Hallucinated
**Runs:** 07 | **Classification:** IGNORE + HALLUCINATE | **Severity:** HIGH
Agent jumped from Order System directly to criteria gate, skipping the states prompt entirely. Despite never collecting states, P6 summary showed "Target States: CA, AZ, TX" — values copied from the prompt example text ("e.g., CA, AZ, TX"). Two failures: (1) IGNORE — skipped the CRITICAL stop guard before states prompt, (2) HALLUCINATE — fabricated state values from example text instead of leaving empty or prompting.

### PRD-R2-GATESKIP — Criteria Gate Skipped Entirely
**Runs:** 12, 13 | **Classification:** IGNORE | **Severity:** HIGH
After states were collected and Phase 5 tool chain completed, the criteria gate was never displayed. Agent jumped directly to Phase 6 summary. The CRITICAL STOP guard in Phase 5 was ignored. 2/20 runs (10%).

### PRD-R2-REDETECT — Unnecessary Format Re-Detection After Explicit JSON Selection
**Runs:** 10 | **Classification:** IGNORE | **Severity:** Medium
After user explicitly selected "JSON" from the content type ActionSet, agent still ran the auto-detect path and asked "I've detected this as JSON format. Is this correct?" This step should only trigger when contentType = "I'm not sure" per split Phase 3a-webhook instructions.

### PRD-R2-CTSKIP — Content Type Card Skipped After Typed Fallback
**Runs:** 01 | **Severity:** Medium
When field mapping prompt ("Would you like to configure field mappings?") rendered without buttons and user typed "I'll provide instructions", the agent skipped the content type selection card entirely and went straight to auto-detect path ("Please paste your posting instructions..."). The typed fallback caused the agent to collapse the content type selection step. Classification: IGNORE (context loss from typed fallback — failure pattern #5).

### PRD-R2-HALLPROMPT — Hallucinated TOOL_DEFAULTS Prompts After Criteria Exit
**Runs:** 62 | **Classification:** HALLUCINATE | **Severity:** Medium
After criteria loop exit, GPT-5-mini hallucinated two extra prompts for fields that should be auto-applied via TOOL_DEFAULTS: (1) "Please provide a name for the delivery account" (should be auto-set to `{companyName}-Account`), (2) "Please select a status for the delivery account" with Open/Suspended/OnHold/Closed buttons (should be auto-set to `Open`). Agent lost TOOL_DEFAULTS context after criteria builder summarize_history. Non-blocking — flow completed correctly after user provided values.



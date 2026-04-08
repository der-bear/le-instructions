# Stability Test Failure Analysis (v3)

**Date:** 2026-04-08
**Scope:** Stabilized (RA-*), Rework (RB-*), and Split (RC-*) variants across rounds R1a, R1b, R2, R2c, and Split R1.
**Total runs:** 107 · **Findings cataloged:** 84 (RA-* 35, RB-* 36, RC-* 13)
**Source data:** `stability-stabilized-findings.md`, `stability-rework-findings.md`, `stability-split-findings.md`, `stability-production-findings.md`

---

## 3. Per-Checkpoint Failure Rate Scoreboard

**Cell format:** `pass/applicable (pass%)` followed by emoji indicating fail rate.  
Color coding (by fail rate): 🟢 0% (perfect) · ✅ 1-19% fail · 🟡 20-49% fail · 🔴 ≥50% fail

### 🏆 Round Champion Scoreboard

Ranked by Avg Score.

| Round | Avg Score | Trophy |
|-------|-----------|--------|
| Stabilized R1b (mixed pre-fix) | **97.8%** | 🥇 Best stabilized — mixed model alone wins |
| Split R1 (mixed baseline, audited) | **94.4%** | 🥈 Split baseline — audited scoring, single round, 6/10 PASS (new) |
| Stabilized R2c (mixed post-fix) | 93.1% | 🥉 Stab post-fix |
| Rework R2c (mixed post-fix) | **93.0%** | Best rework — fixes + mixed model required |
| Stabilized R1a (gpt-5.4 pre-fix) | 86.8% | Stab baseline |
| Rework R1a (gpt-5.4 pre-fix) | 85.7% | Rework baseline |
| Rework R2 (gpt-5.4 post-fix) | 81.9% | ⚠️ Cosmetic doubling persisted |
| Stabilized R2 (gpt-5.4 post-fix) | 76.0% | ⚠️ Worst avg after fixes |
| Rework R1b (mixed pre-fix) | 72.9% | ⚠️ Mixed alone failed rework |

### 📊 Per-Checkpoint Master Grid

**Sorted by ROUND (R1a → R2c), not by variant.** Each round shows all three variants side-by-side.  
**Source:** All findings matrices extracted to CSV (`/tmp/stability-matrix.csv`) and computed programmatically.

| IGNORE | HALLUCINATE | PLATFORM | CLOSED | FLAKY | AMBIGUOUS |
|:------:|:-----------:|:--------:|:------:|:-----:|:---------:|
| **51** (63.0%) | **11** (13.6%) | **7** (8.6%) | **6** (7.4%) | **6** (7.4%) | **2** (2.5%) |

<sub>**IGNORE** = agent skipped explicit instruction · **HALLUCINATE** = agent invented behavior not in instructions · **PLATFORM** = non-deterministic rendering / MCP / timeout · **CLOSED** = verified fixed · **FLAKY** = intermittent, under observation · **AMBIGUOUS** = instruction gap or contradiction</sub>

| Checkpoint | Stab R1a | Rew R1a | Stab R1b | Rew R1b | Stab R2 | Rew R2 | Stab R2c | **Rew R2c** | Split R1 |
|------------|----------|---------|----------|---------|---------|--------|----------|-------------|----------|
| P1-PROMPT† | 20/20 (100%) 🟢 | 5/6 (83%) ✅ | 9/9 (100%) 🟢 | 6/6 (100%) 🟢 | 9/9 (100%) 🟢 | 1/1 (100%) 🟢 | 9/9 (100%) 🟢 | 8/8 (100%) 🟢 | 10/10 (100%) 🟢 |
| P2-DROP | 20/20 (100%) 🟢 | 20/20 (100%) 🟢 | 9/9 (100%) 🟢 | 3/10 (30%) 🔴 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P3-SCHED | 18/20 (90%) ✅ | 16/16 (100%) 🟢 | 8/9 (89%) ✅ | 10/10 (100%) 🟢 | 8/10 (80%) 🟡 | 5/6 (83%) ✅ | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 9/10 (90%) ✅ |
| P3-WURL | 19/20 (95%) ✅ | 15/15 (100%) 🟢 | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 7/8 (88%) ✅ | 8/8 (100%) 🟢 | 8/8 (100%) 🟢 | 4/4 (100%) 🟢 | 10/10 (100%) 🟢 |
| P3-JSON | 18/20 (90%) ✅ | 11/12 (92%) ✅ | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 4/7 (57%) 🔴 | 6/6 (100%) 🟢 | 5/5 (100%) 🟢 | 4/4 (100%) 🟢 | 10/10 (100%) 🟢 |
| P3-TABLE | 18/20 (90%) ✅ | 11/11 (100%) 🟢 | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 7/7 (100%) 🟢 | 7/7 (100%) 🟢 | 7/7 (100%) 🟢 | 4/4 (100%) 🟢 | 9/10 (90%) ✅ |
| P3-COUNT | 17/20 (85%) ✅ | 12/12 (100%) 🟢 | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 7/7 (100%) 🟢 | 6/7 (86%) ✅ | 7/7 (100%) 🟢 | 4/4 (100%) 🟢 | 10/10 (100%) 🟢 |
| P3B-TEST | 20/20 (100%) 🟢 | 19/19 (100%) 🟢 | 9/9 (100%) 🟢 | 6/6 (100%) 🟢 | 8/8 (100%) 🟢 | 8/8 (100%) 🟢 | 9/10 (90%) ✅ | 6/6 (100%) 🟢 | 10/10 (100%) 🟢 |
| P4-SUMM | 20/20 (100%) 🟢 | 18/19 (95%) ✅ | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P5-PRICE | 20/20 (100%) 🟢 | 18/19 (95%) ✅ | 9/9 (100%) 🟢 | 9/9 (100%) 🟢 | 7/10 (70%) 🟡 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 9/10 (90%) ✅ |
| P5-EXCL | 20/20 (100%) 🟢 | 15/16 (94%) ✅ | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 |
| P5-ORDER | 20/20 (100%) 🟢 | 17/18 (94%) ✅ | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P5-STATE | 10/20 (50%) 🔴 | 10/16 (62%) 🟡 | 7/9 (78%) ✅ | 3/10 (30%) 🔴 | 5/10 (50%) 🔴 | 5/10 (50%) 🔴 | 10/10 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 |
| P5-NORM | 9/20 (45%) 🔴 | 7/10 (70%) 🟡 | 9/9 (100%) 🟢 | 4/7 (57%) 🔴 | 6/9 (67%) 🟡 | 8/9 (89%) ✅ | 9/10 (90%) ✅ | 7/10 (70%) 🟡 | 8/10 (80%) 🟡 |
| P5-FIELD | 18/20 (90%) ✅ | 6/15 (40%) 🔴 | 9/9 (100%) 🟢 | 4/10 (40%) 🔴 | 2/8 (25%) 🔴 | 3/8 (38%) 🔴 | 9/10 (90%) ✅ | 7/10 (70%) 🟡 | N/A (split uses P5-GATE: 9/10 = 90% ✅) |
| P5-CR1 | 16/20 (80%) 🟡 | 8/11 (73%) 🟡 | 9/9 (100%) 🟢 | 2/9 (22%) 🔴 | 2/8 (25%) 🔴 | 5/8 (62%) 🟡 | 5/7 (71%) 🟡 | 6/6 (100%) 🟢 | 9/10 (90%) ✅ |
| P5-ENUM | 16/20 (80%) 🟡 | 1/2 (50%) 🔴 | 8/9 (89%) ✅ | 0/6 (0%) 🔴 | 1/6 (17%) 🔴 | 3/8 (38%) 🔴 | 6/7 (86%) ✅ | 5/5 (100%) 🟢 | 9/10 (90%) ✅ |
| P5-CR3 | 5/9 (56%) 🟡 | 1/12 (8%) 🔴 | 1/1 (100%) 🟢 | 1/2 (50%) 🔴 | 0/6 (0%) 🔴 | 1/8 (12%) 🔴 | 3/4 (75%) 🟡 | 4/4 (100%) 🟢 | 8/10 (80%) 🟡 |
| P5-DONE | 8/20 (40%) 🔴 | 1/12 (8%) 🔴 | 9/9 (100%) 🟢 | 3/10 (30%) 🔴 | 4/10 (40%) 🔴 | 6/10 (60%) 🟡 | 9/10 (90%) ✅ | 7/8 (88%) ✅ | 8/10 (80%) 🟡 |
| P6-BOOL | 20/20 (100%) 🟢 | 16/17 (94%) ✅ | 9/9 (100%) 🟢 | 8/10 (80%) 🟡 | 10/10 (100%) 🟢 | 9/9 (100%) 🟢 | 8/10 (80%) 🟡 | 9/10 (90%) ✅ | 7/10 (70%) 🟡 |
| P6-SUMM | 19/20 (95%) ✅ | 13/17 (76%) 🟡 | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 | 10/10 (100%) 🟢 | 8/10 (80%) 🟡 | 9/10 (90%) ✅ | 9/10 (90%) ✅ |
| P7-SUMM | 20/20 (100%) 🟢 | 16/16 (100%) 🟢 | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 9/9 (100%) 🟢 | 9/9 (100%) 🟢 | 6/7 (86%) ✅ | 10/10 (100%) 🟢 | N/A (split has no P7-SUMM checkpoint) |
| P8-ACT | 19/20 (95%) ✅ | 17/17 (100%) 🟢 | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 9/10 (90%) ✅ (P7-ACT in split numbering) |

### Key checkpoint trends across rounds

**Stabilized variant:**
- **P5-CR3 post-audit:** Original scoring conflated P5-CR3 with Finding U (payload persistence); 2026-04-07 audit reclassified many F scores to Y or N/A (scenarios with <3 criteria aren't testable). See §4 Round Progression Analysis for the full per-variant progression and denominators.
- **P5-DONE remained the dominant persistence failure** in R1a and R2. Mixed model rounds (R1b, R2c) dramatically improved it.
- **R1b mixed model is the best stabilized config** for overall criteria persistence.
- **R2c achieves perfect P5-EXCL/ORDER/STATE/NORM** (10/10 all four) and field suggestion step is now strong (9/10 after P5-FIELD reclassification — see §3 Root Cause Analysis #1 for full explanation of why the original 4/10 score was phantom).

**Rework variant:**
- **R1a P5-CR3 = 1/12** and **P5-DONE = 1/12** — the worst criteria persistence in any variant/round. These failures are all legitimate RB-A loop exits (loop exited after 1st criterion). Improved to 4/4 (100%) P5-CR3 and 7/8 P5-DONE in R2c post-audit.
- **R1b regression: P2-DROP 3/10** — gpt-5-mini on intro phases introduced RB-AA timezone-skipped bug. Eliminated in R2 and R2c (10/10).
- **R2c is the only rework round with P5-CR1 = 6/6 perfect** — criteria flow now reliably starts.
- **P5-NORM regression in R2c (7/10 vs R2 8/9)** — driven by new RB-V-skip bug (state criterion lost on skip-criteria branch, R2c-09 and R2c-10).

**Split variant:**
- **94.4% audited R1 baseline** — audit applied 2026-04-08, 6/10 audited PASS (highest among dev variants).
- **P5-STATE regression in R09** — silent states-skip recurred once after being patched pre-R02, indicating the CRITICAL guard fix is probabilistic rather than deterministic. Same IGNORE class as RA-P/RB-N.
- **P6-BOOL 7/10 (worst split checkpoint)** — "Disabled" label leak in R06 and R08 plus R01 cascade; display-only issue in the Phase 6 card template.
- **P5-DONE loop exit in R07** — RC-R07-F01 criteria loop self-terminated after the 2nd criterion, same class as RB-A but single-run occurrence.
- **Phases 1-4 are essentially flat at 100%** — the instability surface is concentrated in Phase 5+, matching the cross-variant pattern.

### 📊 Checkpoint Movement Scoreboard (Worst Round → Latest Round)

Includes both improvements and regressions — negative deltas indicate a checkpoint that got worse over rounds.

**Methodology:** For each checkpoint, find the round with the WORST functional pass rate, then compare against the LATEST round (R2c). Improvement = latest − worst.

#### Rework Variant (worst → R2c)

| Rank | Checkpoint | Worst Round | Worst Pass% | R2c Pass% | Δ | Cause |
|------|------------|-------------|-------------|-----------|---|-------|
| 🥇 | **P5-ENUM** | R1b (mixed pre-fix) | 0% | 100% | **+100pp** | FIX-11 verified (ChoiceSet always) |
| 🥈 | **P5-DONE** | R1a (gpt-5.4 pre-fix) | 8% | 88% | **+80pp** | FIX-9 + mixed model + cosmetic deprioritization |
| 🥉 | **P5-CR1** | R1b (mixed pre-fix) | 22% | 100% | **+78pp** | Criteria flow now reliably starts |
| 4 | **P5-CR3** | R1a (gpt-5.4 pre-fix) | 8% | 100% | **+92pp** | FIX-9 (loop prompt) verified; post-audit N/A reclassification |
| 5 | **P2-DROP** | R1b (mixed pre-fix) | 30% | 100% | **+70pp** | RB-AA gpt-5-mini regression eliminated |
| 6 | **P5-STATE** | R1b (mixed pre-fix) | 30% | 90% | **+60pp** | RB-N improved (FIX-3 + mixed model) |
| 7 | **P5-FIELD** | R2 (gpt-5.4 post-fix) | 38% | 70% | **+32pp** | Partial RB-D improvement |
| 8 | **P5-NORM** | R1b (mixed pre-fix) | 57% | 70% | **+13pp** | But RB-V-skip new regression |

#### Stabilized Variant (worst → R2c)

| Rank | Checkpoint | Worst Round | Worst Pass% | R2c Pass% | Δ | Cause |
|------|------------|-------------|-------------|-----------|---|-------|
| 🥇 | **P5-ENUM** | R2 (gpt-5.4 post-fix) | 17% | 86% | **+69pp** | Mixed model unlocked enum handling |
| 🥈 | **P5-STATE** | R1a / R2 | 50% | 100% | **+50pp** | Mixed model unlocked states |
| 🥉 | **P5-DONE** | R1a / R2 | 40% | 90% | **+50pp** | Mixed model unlocked criteria persistence |
| 4 | **P5-CR1** | R2 (gpt-5.4 post-fix) | 25% | 71% | **+46pp** | |
| 5 | **P5-NORM** | R1a (gpt-5.4 pre-fix) | 45% | 90% | **+45pp** | |
| 6 | **P5-CR3** | R2 (gpt-5.4 post-fix) | 0% | 75% | **+75pp** | Post-audit: F→Y for loop-OK-but-payload-dropped cases; F→N/A for <3 criteria scenarios |
| 7 | **P5-FIELD** | R2 (gpt-5.4 post-fix) | 25% | 90% | **+65pp** | After phantom-finding reclassification (only Run 04 real failure — card rendered as plain text) |

#### Split Variant (baseline only)

Split variant has only R1 baseline. No R2 round is currently scheduled; if a follow-up round is run the progression delta will be added here.

### 🏷️ Aggregate Classification Summary

Counts produced by independent classification table audit (Agent 2 round 2 verification). Split variant additions are shown as a separate Δ layer.

**Category Legend:**
- **IGNORE** — Instruction explicitly present but agent silently skipped it. Uses "Do NOT skip", "MANDATORY", "STOP AND YIELD" language.
- **HALLUCINATE** — Agent generated content or behavior not present in instructions.
- **AMBIGUOUS** — Instruction unclear, contradictory, or has a gap that led to failure.
- **PLATFORM** — Non-deterministic rendering, MCP failures, context overflow, tool timeouts — not an instruction or agent-behavior issue.
- **FLAKY** — Intermittent; no consistent reproduction; under observation.
- **NOT_A_BUG** — Observed behavior is actually expected; no fix needed.

| Category | Stab + Rework | Split | Combined | **% of Total** | Members (combined) |
|----------|:-------------:|:-----:|:--------:|:--------------:|--------------------|
| **IGNORE** | 47 | +4 | **51** | **63.0%** | Stab+Rew: RB-H reclassified IGNORE per Agent 2 verification (instruction explicitly says `display_adaptive_card`; agent emits plain text = silent tool skip). Split adds: RC-R05-F02 (P5 re-execution loop), RC-R06-F01 (P5-PRICE silent skip), RC-R10-F01 (P3b typed-fallback), RC-R10-F02 (P5b→P6 typed-fallback) |
| **HALLUCINATE** | 10 | +1 | **11** | **13.6%** | Stab+Rew: RA-B, RA-H, RA-I, RA-Q, RA-NEW-2, RA-NEW-4, RA-NEW-6, RA-NEW-7, RA-HALLUC-CG, RB-O. Split adds: RC-R06-F02 (display name prompt hallucinated in Phase 5b) |
| **PLATFORM** | 7 | 0 | **7** | **8.6%** | RA-X, RA-NEW-R2-9, RB-F, RB-Q, RB-X, RB-NEW-R2-5, RB-NEW-R2-6 |
| **CLOSED** | 4 | +2 | **6** | **7.4%** | Stab+Rew: RB-I (FIX-16), RB-M (FIX-10), RB-CC (FIX-11), RB-DD (FIX-14). Split adds: RC-R01-F01 (P5-GATE stop guard), RC-R01-F02 (P5-STATE stop guard; flaky closure — see RC-R09-F01) |
| **FLAKY** | 0 | +6 | **6** | **7.4%** | Split introduces FLAKY as a top-level class: RC-R02-F01, RC-R03-F01, RC-R05-F01, RC-R07-F01, RC-R08-F01, RC-R09-F01. Pre-split RA/RB audit did not use FLAKY |
| **AMBIGUOUS** | 2 | 0 | **2** | **2.5%** | RA-A (verified — Phase 2 has zero typed-input fallback); RB-NEW-R2-1 (verified — fuzzy match instructed but no display normalization). Split adds none — split's "new defect" items classified IGNORE per symptom, not AMBIGUOUS |
| **Total countable** | **68** | **+13** | **81** | **100%** | (excludes NOT_A_BUG; combined rounding: 102% due to post-audit HALLUCINATE reclassifications — IGNORE row intentionally kept at 47 to preserve the 49 → 47 → 51 delta chain) |
| **NOT_A_BUG** | 3 | 0 | **3** | — | RB-B, RB-J, RB-R |
| **Total entries** | **71** | **+13** | **84** | — | Sum of all categories including NOT_A_BUG |

**IGNORE delta chain:** 49 → 47 (post-R2c audit) → 51 (after split adds 4 new IGNOREs).

**Cross-root-cause links:** RC-R01-F02 (P5-STATE silent skip) shares root cause with RA-P / RB-N (states question silently skipped) — cross-linked in §6 IGNORE-1. RC-R10-F01 and RC-R10-F02 (typed-fallback) share root cause with RB-H (card rendered as plain text) — same IGNORE-NEW class.

**Findings impact attribute (orthogonal to root cause):**

| Finding | Root Cause | Impact | Why Cosmetic |
|---------|-----------|--------|--------------|
| RB-G | IGNORE | Cosmetic | Doubled prompt collects same data correctly |
| RB-H | IGNORE | Cosmetic (typically) | Plain-text fallback typed correctly captures data |
| RB-NEW-R2-6 | PLATFORM | Cosmetic | P7 plain-text fallback shows correct values |

---

## 5. Findings Classification

### Per-Variant Findings Catalog (Condensed)

#### Stabilized Variant (RA-*) — 35 findings

| ID | Description | Class | Status |
|----|-------------|-------|--------|
| RA-A | Lead type typed fallback asks for UID | AMBIGUOUS | Open (true gap) |
| RA-B | Hallucinated UID in DEBUG response | HALLUCINATE | Open |
| RA-C | deliveryDays not built from schedule input | IGNORE | Open |
| RA-D | Phase regression after Webhook button click | IGNORE | Open |
| RA-E | Content type asked as plain text, not ActionSet | IGNORE | Open |
| RA-F | Only 1 of 8 fields mapped | IGNORE | Open |
| RA-G | Connection test success message swallowed | IGNORE | Open |
| RA-H | Company name state retention error | HALLUCINATE | Open |
| RA-I | Hallucinated tool call (Portal vs Webhook) | HALLUCINATE | Open |
| RA-J | State normalization skipped | IGNORE | Open |
| RA-K | Criteria phase entirely skipped | IGNORE | Open |
| RA-L | "done" stored as criteria value | IGNORE | Open |
| RA-M | Enum criterion handling failure | IGNORE | Open |
| RA-N | Schedule hours and body prompt skipped | IGNORE | Open |
| RA-O | Phase 5 restart after "continue" | IGNORE | Open |
| RA-P | States silently skipped entirely | IGNORE | Open (improved in mixed model rounds) |
| RA-Q | Phantom CA,AZ,TX states | HALLUCINATE | Open |
| RA-R | Field suggestion step bypassed after state skip | IGNORE | Open |
| RA-S | Webhook URL never requested | IGNORE | Open |
| RA-T | JSON parse error before schema provided | IGNORE | Open |
| RA-U | Criteria not persisted to account | IGNORE | Open (improved in R1b/R2c) |
| RA-V | State/price prompt doubled | IGNORE | Open (cosmetic in newer rounds) |
| RA-W | States and criteria prompt merged | IGNORE | Open |
| RA-X | Phase 3 instructions loaded 3x | PLATFORM | Open |
| RA-Y | Typed criterion input ignored | IGNORE | Open |
| RA-NEW-1 | Posting instructions prompt skipped | IGNORE | Open |
| RA-NEW-2 | Phase 5 batching (hallucinated criteria gate auto-skipped) | HALLUCINATE | Open (cross-variant; see RA-HALLUC-CG) |
| RA-NEW-3 | States re-prompted when full names given | IGNORE | Open |
| RA-NEW-4 | "Please type Continue" after Add criteria | HALLUCINATE | Open |
| RA-NEW-5 | Phase 5c silent skip | IGNORE | Open |
| RA-NEW-6 | States shown inside hallucinated criteria gate card | HALLUCINATE | Open (cross-variant; see RA-HALLUC-CG) |
| RA-NEW-7 | Phase 5 prompt hallucination (FTP creds on webhook) | HALLUCINATE | Open |
| RA-NEW-R2-8 | Criteria builder bypassed after 2nd Add click | IGNORE | Open |
| RA-NEW-R2-9 | Post-creation states msg causes indefinite spin | PLATFORM | Open |
| RA-HALLUC-CG | Agent hallucinates rework "Add criteria \| Skip" criteria gate prompt in Stabilized; the stabilized phase-5 file has no such gate (STEP 7 uses "Show more fields \| Skip" on the field suggestion card). Observed in R2 runs 01, 03, 05 (notes), and possibly others. Causes ambiguity when scoring P5-FIELD because the rendered gate-style buttons are mistaken for legitimate behavior. | HALLUCINATE | Open (cross-variant prompt contamination) |

**Stabilized totals:** 23 IGNORE · 9 HALLUCINATE · 1 AMBIGUOUS · 2 PLATFORM · 0 CLOSED · 0 NOT_A_BUG = **35 entries** (RA-NEW-2 and RA-NEW-6 reclassified IGNORE→HALLUCINATE; RA-HALLUC-CG added as new entry; RA-R kept IGNORE but renamed)

#### Rework Variant (RB-*) — 36 findings

| ID | Description | Class | Status |
|----|-------------|-------|--------|
| RB-A | Criteria loop exits after 1 criterion | IGNORE | Open (improved 88%→20% in R2c) |
| RB-B | Summary card omits entity IDs | NOT_A_BUG | Excluded |
| RB-C | States not normalized after summarization | IGNORE | Closed via FIX-17 (R2 verified) |
| RB-D | Criteria prompt entirely skipped | IGNORE | Open (50%→30% in R2c) |
| RB-E | Account name asked before price | IGNORE | Open |
| RB-F | Phase 5c fails to load on first click | PLATFORM | Open (rare in R2c) |
| RB-G | Phase transition prompts doubled | IGNORE | Open, **Impact: Cosmetic** |
| RB-H | Content type card rendered as plain text | IGNORE (was PLATFORM) | Open, **Impact: Cosmetic** |
| ✅ **RB-I** | XML parse failure no format re-detection | **CLOSED** | Fixed via FIX-16 (R2c-03 verified) |
| RB-J | postingInstructions cleared on switch | NOT_A_BUG | Excluded |
| RB-K | Phase 5 not loaded before responding | IGNORE | Open |
| RB-L | Content type skipped (RB-G cascade) | IGNORE | Open |
| ✅ **RB-M** | `>=` operator stored as Equal | **CLOSED** | Fixed via FIX-10 (R2c-03/05/06/07 verified) |
| RB-N | States question skipped entirely | IGNORE | Open (50%→10% in R2c) |
| RB-O | Phantom CA,AZ,TX states | HALLUCINATE | Open (regression in R2c-05) |
| RB-P | create_delivery_account silently skipped | IGNORE | Open (scope expanded to Email in R2c-04) |
| RB-Q | "Show more fields" button triggers error | PLATFORM | Open (new variant on P5-EXCL in R2c-03) |
| RB-R | Portal uses deliveryType HttpPost | NOT_A_BUG | Excluded |
| RB-S | Numeric criteria saved without UI ack | IGNORE | Open |
| RB-T | "continue" after P5c puts agent in confused state | IGNORE | Open |
| RB-U | update_delivery_account calls non-cumulative | IGNORE | Open |
| RB-V | Criteria accepted/shown but never sent to API | IGNORE | Open |
| RB-V-skip (NEW R2c) | State criterion lost on skip-criteria path | IGNORE | New (R2c-09, R2c-10) |
| RB-W | State UID missing from states value (NY dropped) | IGNORE | Open |
| RB-X | P5c empty-response loop with large lead types | PLATFORM | Open |
| RB-Y | Phase 3 asks name+type before schedule cards | IGNORE | Open |
| RB-Z | Phase 8 credentials asked before Phase 5 | IGNORE | Open |
| RB-AA | P2 timezone/status systematically skipped | IGNORE | Open (R1b only — gpt-5-mini regression) |
| RB-BB | P5 off-script prompt causes data collapse | IGNORE | Open |
| ✅ **RB-CC** | Enum bypass / no ChoiceSet | **CLOSED** | Fixed via FIX-11 (R2c-02/06/07/08 verified) |
| ✅ **RB-DD** | P3B no Retry/Skip card after conn test | **CLOSED** | Fixed via FIX-14 (R2c P3B-TEST 6/6) |
| RB-NEW-R2-1 | CamelCase field name not recognized | AMBIGUOUS | Open (true gap) |
| RB-NEW-R2-3 | Phase 5c silent skip via premature summarize_history | IGNORE | Open |
| RB-NEW-R2-4 | States combined with criteria gate in single card | IGNORE | Open |
| RB-NEW-R2-5 | Tool/resource access loss after extended DEBUG | PLATFORM | Open |
| RB-NEW-R2-6 | Phase 7 client summary card render failure | PLATFORM | Open, **Impact: Cosmetic** |

**Rework totals:** 22 IGNORE (incl. RB-H reclassified) · 1 HALLUCINATE · 1 AMBIGUOUS · 5 PLATFORM · **4 CLOSED** · 3 NOT_A_BUG = **36 entries**

#### Split Variant (RC-*) — 13 findings

| ID | Description | Class | Status |
|----|-------------|-------|--------|
| ✅ **RC-R01-F01** | P5-GATE silently skipped (cascade to all post-gate checkpoints in R01) | IGNORE | **CLOSED** — Pre-R02 fix: `CRITICAL: Display the card below and STOP. Do NOT auto-select.` added to split-phase-5. Verified R02–R08 clean. |
| ✅ **RC-R01-F02** | States prompt silently skipped | IGNORE | **CLOSED (flaky)** — Pre-R02 fix: `CRITICAL: Display the prompt below and STOP.` added before states ASK. Verified R02–R08 clean but regressed in R09 — see RC-R09-F01. Cross-link: IGNORE-1 (RA-P / RB-N). |
| RC-R02-F01 | Mapping preview card skipped (P3-TABLE) | FLAKY | Open — 1/10, not reproduced in R03–R10 |
| RC-R03-F01 | Schedule ordering non-deterministic (collected in 3a-webhook instead of router) | FLAKY | Open — 1/10, data correct despite ordering |
| RC-R05-F01 | Content type card resequenced (W4 pattern) | FLAKY | Open — 1/10, data correct |
| RC-R05-F02 | Phase 5 re-execution loop after summarize_history | IGNORE | Open — new, summarize_history side effect; model re-executed completed STOP-AND-YIELD steps |
| RC-R06-F01 | P5-PRICE silently skipped (account created with price=0) | IGNORE | Open — new, high-severity data impact |
| RC-R06-F02 | Unexpected display name prompt in Phase 5b | HALLUCINATE | Open — new, prompt not present in Phase 5b instructions |
| RC-R07-F01 | Criteria loop self-terminated after 2nd criterion (3rd criterion lost) | FLAKY | Open — 1/10, same class as RB-A |
| RC-R08-F01 | P6-BOOL Order System shows "Disabled" not "No" | FLAKY | Open — 2/10 (R06, R08); display-only leak |
| RC-R09-F01 | P5-STATE silently skipped (intermittent recurrence of RC-R01-F02) | FLAKY | Open — probabilistic regression of closed defect; same root cause as IGNORE-1 |
| RC-R10-F01 | Typed-fallback in Phase 3b before test connection card | IGNORE | Open — new, cross-link to IGNORE-NEW (RB-H) |
| RC-R10-F02 | Typed-fallback at Phase 5b → Phase 6 transition (next_instructions not auto-followed) | IGNORE | Open — new, cross-link to IGNORE-NEW (RB-H) |

**Split totals:** 4 IGNORE (open) · 1 HALLUCINATE · 6 FLAKY · **2 CLOSED** · 0 AMBIGUOUS · 0 PLATFORM · 0 NOT_A_BUG = **13 entries**

Split variant: 13 findings — 2 CLOSED, 6 FLAKY, 5 NEW (4 IGNORE + 1 HALLUCINATE).

---

## 7. Cosmetic vs Functional Failure Convention

Introduced in R2c for Rework. Later applied to Stabilized R2c (2026-04-07 audit) and Split R1 (2026-04-08 audit). Legacy rounds (R1a/R1b/R2 for Stabilized and Rework) remain in raw scoring.

### Definitions

- **F*** = cosmetic failure. Workflow still completes correctly. Root cause classification unchanged. NOT counted in score percentage.
  - Examples: RB-G prompt doubling (data still collected), RB-H plain-text card fallback (typed response accepted), RB-NEW-R2-6 P7 card render fail under context loss (plain text shows correct data)
- **F** = functional failure. Counted in score.
  - Examples: skipped step, lost data (RB-V-skip), wrong value (Equal vs GreaterOrEqual pre-FIX-10), hallucinated value (phantom states), never-created entity (RB-P)

### Why this convention matters

Pre-R2c, RB-G prompt doubling counted as failure even though every doubled prompt still collected the user's response correctly. After R2c-style audit, R1a-18 promoted from 13/14 (93%) to 13/13 (100%) PASS — adding 1 PASS to Rework R1a (4 total).

The convention does **not** change root cause classification. RB-G is still IGNORE (STOP AND YIELD instructed; agent violates). It only affects scoring impact.

### Application to scoring

- **Rework R1a, R1b, R2:** Raw scoring (cosmetic counted)
- **Rework R2c:** Audited scoring (cosmetic excluded)
- **Stabilized R2c:** Audited scoring (P5-FIELD and P5-CR3 reclassifications applied 2026-04-07). Earlier stabilized rounds (R1a, R1b, R2): Raw scoring.
- **Split R1:** Audited scoring applied 2026-04-08 (see `stability-split-findings.md` Finding Frequency + Severity Summary for per-finding F/F\* classifications)

For apples-to-apples comparison across variants, see the per-checkpoint grid which uses functional-only counts uniformly.

---

## 8. Test vs Production Comparison

Production is `delivery/` (original baseline) running on `leadexec.clickpointsoftware.com`, not `/delivery-original-stabilized`. Stability findings for production live in `stability-production-findings.md` (4 runs, PR1).

**Goal:** Dev variants should not drop materially below production on stability. "Materially below" is defined as either (a) avg functional score more than 5 percentage points below production, or (b) any IGNORE-class finding recurring at a higher frequency than production.

### 8.1 Pass Rate Delta

| Variant | Best Round | Avg Score | PASS Rate | Delta vs Production |
|---------|-----------|-----------|-----------|--------------------|
| Production (PR1) | PR1 | **95%** | 1/4 (25%) | — |
| Stabilized (best) | R1b (mixed pre-fix) | 97.8% | 5/9 (56%) | **+2.8 pp** |
| Stabilized (post-fix best) | R2c (mixed post-fix) | 93.1% | 4/10 (40%) | −1.9 pp |
| Rework (best) | R2c | 93.0% | 3/10 (30%) | −2.0 pp |
| **Split (R1, audited)** | **R1** | **94.4%** | **6/10 (60%)** | **−0.6 pp** |

**Analysis:** Stabilized R1b remains the only dev round that exceeds production's 95% avg, at 97.8% — achieved via mixed model alone with pre-fix instructions. Split R1 (audited) lands at 94.4%, within 1 pp of production and well above the materiality floor. Notably, split has the **highest audited PASS rate of any variant** at 6/10 (60%) — more than double production's 25% and double rework R2c's 30%. This means when split completes a run, it is more likely than any other variant to complete cleanly at 100%; the 0.6 pp delta on average is driven mainly by R01's pre-fix cascade and two singleton regressions (R06 P5-PRICE, R09 P5-STATE). Stabilized R2c (93.1%) and Rework R2c (93.0%) trail production by ~2 pp.

### 8.2 Unique-to-Production Quirks

Findings present in production but absent from or different-in dev variants. Frame as "gap in coverage" — dev tests do not exercise these paths:

1. **PRD-B: Enum auto-resolution without ChoiceSet** — production accepts enum values from user text directly (e.g., `LoanRequestType=Purchase`), skipping the ChoiceSet dropdown. Dev rework variant mandates ChoiceSet (FIX 11). Production's behavior is more permissive but bypasses validation. Not replicated in any dev variant. Classify as PLATFORM difference, not a dev finding.

2. **PRD-LOOP: Criteria loop premature exit** — production exhibits the same class of failure as RB-A / RC-R07-F01, but at nondeterministic exit points (varies per run, not always "after first criterion"). Root cause per `stability-production-findings.md`: production's `/delivery` pack lacks 3 guards present in `/delivery-original-stabilized` — `SUGGEST` vs `ASK` for adaptive card, CRITICAL non-exit guard text, and singular `parsedCriteria` variable instead of explicit `criteriaPayload` array with LOOP BACK. The dev stabilized variant has fixed this class; split inherits the stabilized fix but still exhibits one recurrence (RC-R07-F01). Frequency: production 3/4 (75%), split 1/10 (10%), rework varies per round.

3. **PRD-USER-COLLISION** — production rejects duplicate portal usernames across sessions globally. Not a dev-environment concern (dev uses fresh sessions). Platform-level, not instruction-level. Excluded from dev stability scope.

4. **PRD-A: Chat panel UI glitch** — platform nondeterminism unique to production. Not reproducible in dev. Excluded from dev stability scope.

### 8.3 Stability Floor Check

Criteria derived from `stability-production-findings.md` stability-floor paragraph:
- [ ] Avg functional score ≥ 90% (production is 95%, −5pp floor)
- [ ] PRD-LOOP-class failures ≤ 3/N runs per round
- [ ] All 21 non-loop, non-P1-PROMPT checkpoints pass in every run (production's 100% rate on those)
- [ ] No new high-severity findings absent from production's catalog

Check status per variant:
- **Split R1 (audited):** ✓ 94.4% avg (−0.6 pp, well above floor) · ✓ 1/10 loop failures (well below the 3/N ceiling) · ✓ highest audited PASS rate of any variant (6/10) · ✗ RC-R09-F01 shows P5-STATE regression (singleton, not 100%) · ✗ RC-R06-F01 shows P5-PRICE skip (singleton, high-severity, not in production catalog) — overall **effectively at parity with production**. Two singleton regressions are the remaining blockers before declaring clean parity.
- **Rework R2c:** ✓ 93.0% avg (−2.0 pp) · ✗ criteria loop issues still recurring (RB-A partial — 20% loop-exit rate) · mixed on the "all non-loop checkpoints pass 100%" criterion (P5-NORM 70%, P5-FIELD 70%, P6-BOOL 90% all below 100%). Closer to production than originally framed, but carries more cumulative open findings (36 vs 13).
- **Stabilized R2c:** ✓ 93.1% avg (−1.9 pp) · P5-STATE 100%, P5-NORM 90%, P5-CR1 71%, P7-SUMM 86% · no CLOSED fixes this round (0/34) — R2c gains come from mixed model swap alone. Stabilized R1b (pre-fix mixed) is the actual stabilized at-parity candidate at 97.8%.

**Conclusion:** Split variant is **effectively at parity with production** on audited scoring (−0.6 pp avg, +35 pp on PASS rate) and is the strongest dev variant candidate for production promotion once the two singleton regressions (RC-R06-F01 price skip, RC-R09-F01 states skip) are eliminated. Stabilized R1b remains the only dev round that exceeds production average, at +2.8 pp. Rework R2c trails at −2.0 pp with more cumulative findings. Production's own floor is dragged down by PRD-LOOP, which is the primary blocker for all dev variants to *significantly* exceed production baseline.

---

## Appendix: Cross-Document References

**Per-variant findings files (source data for this analysis):**
- [`stability-stabilized-findings.md`](stability-stabilized-findings.md) — full Stabilized variant run matrix, defect catalog, and round-by-round scoring (RA-* findings)
- [`stability-rework-findings.md`](stability-rework-findings.md) — full Rework variant run matrix, defect catalog, and round-by-round scoring (RB-* findings)
- [`stability-split-findings.md`](stability-split-findings.md) — Split variant R1 run matrix, audited score matrix, and defect catalog (RC-* findings)
- [`stability-production-findings.md`](stability-production-findings.md) — Production variant (`/delivery` on `leadexec.clickpointsoftware.com`) PR1 findings and anomalies

**Previous analysis reports:**
- [`stability-failure-analysis.md`](stability-failure-analysis.md) — original v1 of this document (retained for historical reference; pre-split, pre-audit framing)
- [`stability-failure-analysis-v2.md`](stability-failure-analysis-v2.md) — v2 with split variant integrated; retained for IGNORE verification deep-dive content (§6) and full instruction-line evidence that was trimmed from v3

**Test protocols:**
- [`stability-test-protocol.md`](stability-test-protocol.md) — full stability test protocol with per-phase checkpoints
- [`stability-test-protocol-simple.md`](stability-test-protocol-simple.md) — QA-oriented test protocol for split variant

**Instruction source directories:**
- [`delivery-original-stabilized/resources/`](delivery-original-stabilized/resources/) — Stabilized instruction files
- [`delivery-rework/resources/`](delivery-rework/resources/) — Rework instruction files
- [`delivery-original-stabilized-split/resources/`](delivery-original-stabilized-split/resources/) — Split instruction files

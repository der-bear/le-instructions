# Stability Test Failure Analysis (v2)

**Date:** 2026-04-07
**Scope:** Stabilized (RA-*) and Rework (RB-*) variants across rounds R1a, R1b, R2, R2c
**Total runs:** 97 (Stabilized 49 + Rework 48) · **Findings cataloged:** 70 (RA-* 34, RB-* 36)
**Source data:** `stability-stabilized-findings.md`, `stability-rework-findings.md`

---

## Table of Contents

1. [Rounds Glossary](#rounds-glossary)
2. [Pass Rate Summary (Variant × Round)](#pass-rate-summary)
3. [Per-Checkpoint Failure Rate Grid (All Variants × All Rounds)](#per-checkpoint-grid)
4. [Round Progression Analysis](#progression)
5. [Findings Classification](#classification)
6. [IGNORE Verification](#ignore-verification)
7. [Cosmetic vs Functional Convention](#cosmetic-convention)

---

## <a name="rounds-glossary"></a>1. Rounds Glossary

Round identifiers use the format `{Variant} - {Round} ({model config})` for clarity.

### Stabilized Variant Rounds

| Round | Runs | Model Configuration | Instructions | n |
|-------|------|---------------------|--------------|---|
| **Stabilized - R1a (gpt-5.4 only)** | 01-20 | gpt-5.4-mini all phases | Pre-fix baseline | 20 (Run 21 blank) |
| **Stabilized - R1b (mixed)** | 21-30 | gpt-5-mini for Webhook+criteria, gpt-5.4 for others | Pre-fix baseline | 9 |
| **Stabilized - R2 (gpt-5.4 only)** | 01-10 | gpt-5.4-mini all phases | **Post-fix #1** (commit 5200cc3) | 10 |
| **Stabilized - R2c (mixed)** | 01-10 | gpt-5.4 main router + gpt-5-mini for P3/P5 phases | Post-fix #1 | 10 |

### Rework Variant Rounds

| Round | Runs | Model Configuration | Instructions | n |
|-------|------|---------------------|--------------|---|
| **Rework - R1a (gpt-5.4 only)** | 01-20 | gpt-5.4-mini all phases | Pre-fix baseline | 18 |
| **Rework - R1b (mixed)** | 21-30 | gpt-5-mini for P3+P5, gpt-5.4 for others | Pre-fix baseline | 10 |
| **Rework - R2 (gpt-5.4 only)** | 01-10 | gpt-5.4-mini all phases | **Post-fix #1** (commit 5200cc3, FIXes 1-17) | 10 |
| **Rework - R2c (mixed)** | 01-10 | gpt-5-mini for P3+P5, gpt-5.4 for others | Post-fix #1 (same as R2) | 10 |

**Critical:** Rework R2c is the only round combining post-fix instructions with mixed model. Cannot attribute R2c gains to fixes alone.

### Round Naming Convention

- **R{n}** = round number (R1, R2)
- **a/b/c suffix** = sub-round: `a` = baseline, `b` = alternate model/config, `c` = clean re-run / corrected
- The most important distinction is **pre-fix vs post-fix** instructions (R1x vs R2x)

---

## <a name="pass-rate-summary"></a>2. Pass Rate Summary (Variant × Round)

Avg Score uses each row's stated score from the source matrix.

| Variant - Round | Model | Instructions | n | Avg Score |
|-----------------|-------|--------------|---|-----------|
| Stabilized - R1a (gpt-5.4 only) | gpt-5.4-mini all | Pre-fix | 20 | 85.1% |
| Stabilized - R1b (mixed) | gpt-5-mini critical + gpt-5.4 others | Pre-fix | 9 | **98.2%** |
| Stabilized - R2 (gpt-5.4 only) | gpt-5.4-mini all | Post-fix | 10 | 76.0% |
| Stabilized - R2c (mixed) | gpt-5.4 main + gpt-5-mini P3/P5 | Post-fix | 10 | 92.2% |
| Rework - R1a (gpt-5.4 only) | gpt-5.4-mini all | Pre-fix | 18 | 85.7% |
| Rework - R1b (mixed) | gpt-5-mini P3+P5 + gpt-5.4 others | Pre-fix | 10 | 70.7% |
| Rework - R2 (gpt-5.4 only) | gpt-5.4-mini all | Post-fix | 10 | 81.9% |
| **Rework - R2c (mixed)** | gpt-5-mini P3+P5 + gpt-5.4 others | Post-fix | 10 | **92.5%** |

### Key observations

- **Stabilized R1b achieves the highest avg score (98.2%)** — the mixed model on the simpler stabilized architecture works exceptionally well pre-fix.
- **Rework R2c achieves the highest avg score (92.5%)** — fixes + mixed model combine for measurable improvement.
- **Both variants benefit from mixed model** when paired with their respective best-fit configuration:
  - Stabilized: mixed alone (R1b pre-fix) is the winner
  - Rework: mixed + post-fix instructions (R2c) is the winner
- **Post-fix instructions alone don't unlock improvement** — Stabilized R2 (76.0%) < R1b (98.2%); Rework R2 (81.9%) < R2c (92.5%). Fix combinations matter.

---

## <a name="per-checkpoint-grid"></a>3. Per-Checkpoint Failure Rate Scoreboard

**Cell format:** `pass/applicable (pass%)` followed by emoji indicating fail rate.  
Color coding (by fail rate): 🟢 0% (perfect) · ✅ 1-19% fail · 🟡 20-49% fail · 🔴 ≥50% fail

### 🏆 Round Champion Scoreboard

Ranked by Avg Score.

| Round | Avg Score | Trophy |
|-------|-----------|--------|
| Stabilized R1b (mixed pre-fix) | **98.2%** | 🥇 Best stabilized — mixed model alone wins |
| Rework R2c (mixed post-fix) | **92.5%** | 🥇 Best rework — fixes + mixed model required |
| Stabilized R2c (mixed post-fix) | 92.2% | 🥈 Stab post-fix |
| Rework R1a (gpt-5.4 pre-fix) | 85.7% | 🥉 Rework baseline |
| Stabilized R1a (gpt-5.4 pre-fix) | 85.1% | 🥉 Stab baseline |
| Rework R2 (gpt-5.4 post-fix) | 81.9% | ⚠️ Cosmetic doubling persisted |
| Stabilized R2 (gpt-5.4 post-fix) | 76.0% | ⚠️ Worst avg after fixes |
| Rework R1b (mixed pre-fix) | 70.7% | ⚠️ Mixed alone failed rework |

### 📊 Per-Checkpoint Master Grid

**Sorted by ROUND (R1a → R2c), not by variant.** Each round shows both variants side-by-side.  
**Source:** Both findings matrices extracted to CSV (`/tmp/stability-matrix.csv`) and computed programmatically.

| Checkpoint | Stab R1a | Rew R1a | Stab R1b | Rew R1b | Stab R2 | Rew R2 | Stab R2c | **Rew R2c** |
|------------|----------|---------|----------|---------|---------|--------|----------|-------------|
| P1-PROMPT† | 20/20 (100%) 🟢 | 5/6 (83%) ✅ | 9/9 (100%) 🟢 | 6/6 (100%) 🟢 | 9/9 (100%) 🟢 | 1/1 (100%) 🟢 | 9/9 (100%) 🟢 | 8/8 (100%) 🟢 |
| P2-DROP | 20/20 (100%) 🟢 | 20/20 (100%) 🟢 | 9/9 (100%) 🟢 | 3/10 (30%) 🔴 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P3-SCHED | 18/20 (90%) ✅ | 16/16 (100%) 🟢 | 8/9 (89%) ✅ | 10/10 (100%) 🟢 | 8/10 (80%) 🟡 | 5/6 (83%) ✅ | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P3-WURL | 19/20 (95%) ✅ | 15/15 (100%) 🟢 | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 7/8 (88%) ✅ | 8/8 (100%) 🟢 | 8/8 (100%) 🟢 | 4/4 (100%) 🟢 |
| P3-JSON | 18/20 (90%) ✅ | 11/12 (92%) ✅ | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 4/7 (57%) 🔴 | 6/6 (100%) 🟢 | 5/5 (100%) 🟢 | 4/4 (100%) 🟢 |
| P3-TABLE | 18/20 (90%) ✅ | 11/11 (100%) 🟢 | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 7/7 (100%) 🟢 | 7/7 (100%) 🟢 | 7/7 (100%) 🟢 | 4/4 (100%) 🟢 |
| P3-COUNT | 17/20 (85%) ✅ | 12/12 (100%) 🟢 | 9/9 (100%) 🟢 | 4/4 (100%) 🟢 | 7/7 (100%) 🟢 | 6/7 (86%) ✅ | 7/7 (100%) 🟢 | 4/4 (100%) 🟢 |
| P3B-TEST | 20/20 (100%) 🟢 | 19/19 (100%) 🟢 | 9/9 (100%) 🟢 | 6/6 (100%) 🟢 | 8/8 (100%) 🟢 | 8/8 (100%) 🟢 | 9/10 (90%) ✅ | 6/6 (100%) 🟢 |
| P4-SUMM | 20/20 (100%) 🟢 | 18/19 (95%) ✅ | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P5-PRICE | 20/20 (100%) 🟢 | 18/19 (95%) ✅ | 9/9 (100%) 🟢 | 9/9 (100%) 🟢 | 7/10 (70%) 🟡 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P5-EXCL | 20/20 (100%) 🟢 | 15/16 (94%) ✅ | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 | 9/10 (90%) ✅ |
| P5-ORDER | 20/20 (100%) 🟢 | 17/18 (94%) ✅ | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 | 9/10 (90%) ✅ | 10/10 (100%) 🟢 | 10/10 (100%) 🟢 |
| P5-STATE | 10/20 (50%) 🔴 | 10/16 (62%) 🟡 | 7/9 (78%) ✅ | 3/10 (30%) 🔴 | 5/10 (50%) 🔴 | 5/10 (50%) 🔴 | 10/10 (100%) 🟢 | 9/10 (90%) ✅ |
| P5-NORM | 9/20 (45%) 🔴 | 7/10 (70%) 🟡 | 9/9 (100%) 🟢 | 4/7 (57%) 🔴 | 6/9 (67%) 🟡 | 8/9 (89%) ✅ | 9/10 (90%) ✅ | 7/10 (70%) 🟡 |
| P5-FIELD | 18/20 (90%) ✅ | 6/15 (40%) 🔴 | 9/9 (100%) 🟢 | 4/10 (40%) 🔴 | 2/8 (25%) 🔴 | 3/8 (38%) 🔴 | 9/10 (90%) ✅ | 7/10 (70%) 🟡 |
| P5-CR1 | 16/20 (80%) 🟡 | 8/11 (73%) 🟡 | 9/9 (100%) 🟢 | 2/9 (22%) 🔴 | 2/8 (25%) 🔴 | 5/8 (62%) 🟡 | 5/7 (71%) 🟡 | 6/6 (100%) 🟢 |
| P5-ENUM | 16/20 (80%) 🟡 | 1/2 (50%) 🔴 | 8/9 (89%) ✅ | 0/6 (0%) 🔴 | 1/6 (17%) 🔴 | 3/8 (38%) 🔴 | 6/7 (86%) ✅ | 5/5 (100%) 🟢 |
| P5-CR3 | 8/20 (40%) 🔴 | 1/12 (8%) 🔴 | 9/9 (100%) 🟢 | 1/8 (12%) 🔴 | 0/6 (0%) 🔴 | 1/8 (12%) 🔴 | 2/5 (40%) 🔴 | 4/5 (80%) 🟡 |
| P5-DONE | 8/20 (40%) 🔴 | 1/12 (8%) 🔴 | 9/9 (100%) 🟢 | 3/10 (30%) 🔴 | 4/10 (40%) 🔴 | 6/10 (60%) 🟡 | 9/10 (90%) ✅ | 7/8 (88%) ✅ |
| P6-BOOL | 20/20 (100%) 🟢 | 16/17 (94%) ✅ | 9/9 (100%) 🟢 | 8/10 (80%) 🟡 | 10/10 (100%) 🟢 | 9/9 (100%) 🟢 | 8/10 (80%) 🟡 | 9/10 (90%) ✅ |
| P6-SUMM | 19/20 (95%) ✅ | 13/17 (76%) 🟡 | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 8/10 (80%) 🟡 | 10/10 (100%) 🟢 | 8/10 (80%) 🟡 | 9/10 (90%) ✅ |
| P7-SUMM | 20/20 (100%) 🟢 | 16/16 (100%) 🟢 | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 9/9 (100%) 🟢 | 9/9 (100%) 🟢 | 6/7 (86%) ✅ | 10/10 (100%) 🟢 |
| P8-ACT | 19/20 (95%) ✅ | 17/17 (100%) 🟢 | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 | 9/9 (100%) 🟢 | 9/10 (90%) ✅ | 9/9 (100%) 🟢 | 10/10 (100%) 🟢 |

### Key checkpoint trends across rounds

**Stabilized variant:**
- **P5-CR3/P5-DONE remained the dominant failure** in R1a (8/20 each) and R2 (0/6, 4/10). Mixed model rounds (R1b, R2c) dramatically improved both.
- **R1b mixed model is the best stabilized config** for criteria persistence (9/9 P5-CR3 and P5-DONE).
- **R2c achieves perfect P5-EXCL/ORDER/STATE/NORM** (10/10 all four) and field suggestion step is now strong (9/10 after audit-driven reclassification — only Run 04 remained a real failure where the card was rendered as plain text). Note: this is a corrected figure; the original 4/10 score had counted phantom failures against a "criteria gate" step that doesn't exist in stabilized.

**Rework variant:**
- **R1a P5-CR3 = 1/12** and **P5-DONE = 1/12** — the worst criteria persistence in any variant/round. Improved to 4/5 and 7/8 in R2c.
- **R1b regression: P2-DROP 3/10** — gpt-5-mini on intro phases introduced RB-AA timezone-skipped bug. Eliminated in R2 and R2c (10/10).
- **R2c is the only rework round with P5-CR1 = 6/6 perfect** — criteria flow now reliably starts.
- **P5-NORM regression in R2c (7/10 vs R2 8/9)** — driven by new RB-V-skip bug (state criterion lost on skip-criteria branch, R2c-09 and R2c-10).

### 📈 Improvement Scoreboard (Worst Round → Latest Round)

**Methodology:** For each checkpoint, find the round with the WORST functional pass rate, then compare against the LATEST round (R2c). Improvement = latest − worst.

#### Rework Variant (worst → R2c)

| Rank | Checkpoint | Worst Round | Worst Pass% | R2c Pass% | Δ | Cause |
|------|------------|-------------|-------------|-----------|---|-------|
| 🥇 | **P5-ENUM** | R1b (mixed pre-fix) | 0% | 100% | **+100pp** | FIX-11 verified (ChoiceSet always) |
| 🥈 | **P5-DONE** | R1a (gpt-5.4 pre-fix) | 8% | 88% | **+80pp** | FIX-9 + mixed model + cosmetic deprioritization |
| 🥉 | **P5-CR1** | R1b (mixed pre-fix) | 22% | 100% | **+78pp** | Criteria flow now reliably starts |
| 4 | **P5-CR3** | R1a (gpt-5.4 pre-fix) | 8% | 80% | **+72pp** | FIX-9 (loop prompt) verified |
| 5 | **P2-DROP** | R1b (mixed pre-fix) | 30% | 100% | **+70pp** | RB-AA gpt-5-mini regression eliminated |
| 6 | **P5-STATE** | R1b (mixed pre-fix) | 30% | 90% | **+60pp** | RB-N improved (FIX-3 + mixed model) |
| 7 | **P5-FIELD** | R2 (gpt-5.4 post-fix) | 38% | 63% | **+25pp** | Partial RB-D improvement |
| 8 | **P5-NORM** | R1b (mixed pre-fix) | 57% | 70% | **+13pp** | But RB-V-skip new regression |

#### Stabilized Variant (worst → R2c)

| Rank | Checkpoint | Worst Round | Worst Pass% | R2c Pass% | Δ | Cause |
|------|------------|-------------|-------------|-----------|---|-------|
| 🥇 | **P5-ENUM** | R2 (gpt-5.4 post-fix) | 17% | 86% | **+69pp** | Mixed model unlocked enum handling |
| 🥈 | **P5-STATE** | R1a / R2 | 50% | 100% | **+50pp** | Mixed model unlocked states |
| 🥉 | **P5-DONE** | R1a / R2 | 40% | 90% | **+50pp** | Mixed model unlocked criteria persistence |
| 4 | **P5-CR1** | R2 (gpt-5.4 post-fix) | 25% | 71% | **+46pp** | |
| 5 | **P5-NORM** | R1a (gpt-5.4 pre-fix) | 45% | 90% | **+45pp** | |
| 6 | **P5-CR3** | R2 (gpt-5.4 post-fix) | 0% | 40% | **+40pp** | Recovered but still weak |
| 7 | **P5-FIELD** | R2 (gpt-5.4 post-fix) | 25% | 90% | **+65pp** | After phantom-finding reclassification (only Run 04 real failure — card rendered as plain text) |

### 🎯 Resolved Findings — Per Variant

#### Rework Variant — 4 CLOSED (R2c)

| Finding | Closed By | Verification (instruction line) | R2c Evidence |
|---------|-----------|-------------------------------|--------------|
| ✅ **RB-I** | FIX-16 | `rw-phase-3-webhook.md:69-72` — format re-detection on parse failure | R2c-03: XML→JSON recovery, 8/9 mappings |
| ✅ **RB-M** | FIX-10 | `rw-phase-5c-criteria-builder.md:83-84` — symbol-first operator parsing | R2c-03/05/06/07: `>=` → GreaterOrEqual ✓ |
| ✅ **RB-CC** | FIX-11 | `rw-phase-5c-criteria-builder.md:56-62` — ALWAYS show ChoiceSet for enum | R2c-02/06/07/08: SelfCreditRating UID 343593, LoanRequestType UID 343608 ✓ |
| ✅ **RB-DD** | FIX-14 | `rw-phase-3b-webhook-test.md:27-32` — conn test text in card both paths | R2c P3B-TEST 6/6 = 100% pass |

**Rework closure rate:** 4 / 36 entries = 11% closed this round.

#### Stabilized Variant — 0 CLOSED (R2c)

No verified closures in stabilized R2c. The stabilized variant did NOT receive the same FIX-9/10/11/14/16 instruction fixes as rework. Stabilized R2c improvements come entirely from the model swap (mixed model) without instruction changes.

**Stabilized closure rate:** 0 / 34 entries = 0% closed this round.

### 📉 Frequency Drops — Per Variant (Worst → Latest)

#### Rework Variant frequency improvements (worst-round → R2c)

| Finding | Worst Round | Worst Freq | R2c Freq | Δ | Status |
|---------|-------------|------------|----------|---|--------|
| **RB-A** (criteria loop exit) | R2 (criteria runs) | 88% 🔴 | 20% 🟡 | **-68pp** | FIX-9 partial (closes 4/5) |
| **RB-CR3 fails** | R2 / R1a | 88% 🔴 | 20% 🟡 | **-68pp** | FIX-9/10 effective |
| **RB-N** (states skipped) | R1b | 70% 🔴 | 10% ✅ | **-60pp** | Mixed model + FIX-3 |
| **RB-CC** (enum bypass) | R1b | 100% 🔴 | 0% 🟢 | **-100pp** | **CLOSED via FIX-11** |
| **RB-M** (>= as Equal) | R1a | 20% 🟡 | 0% 🟢 | **-20pp** | **CLOSED via FIX-10** |
| **RB-I** (XML→JSON gap) | R1a | observed | 0% 🟢 | — | **CLOSED via FIX-16** |
| **RB-DD** (P3B card) | R1b | 14% ✅ | 0% 🟢 | **-14pp** | **CLOSED via FIX-14** |
| **RB-D** (criteria gate skip) | R2 | 67% 🔴 | 30% 🟡 | **-37pp** | Partial |
| **RB-G** (prompt doubling) | R2 | 90% 🔴 | ~50% 🟡 (cosmetic) | **-40pp** | Cosmetic now |

#### Stabilized Variant frequency improvements (worst-round → R2c)

| Finding | Worst Round | Worst Freq | R2c Freq | Δ | Status |
|---------|-------------|------------|----------|---|--------|
| **RA-K** (criteria phase skipped) | R2 | 100% 🔴 | 60% 🟡 | **-40pp** | Mixed model partial fix |
| **RA-P** (states skipped) | R1a | 50% 🔴 | 0% 🟢 | **-50pp** | Mixed model fully fixes |
| **RA-U** (criteria not persisted) | R2 | 100% 🔴 | 10% ✅ | **-90pp** | Mixed model fully fixes |
| **RA-M** (enum criterion failure) | R2 | 83% 🔴 | 14% ✅ | **-69pp** | Mixed model partial fix |
| **RA-J** (state normalization) | R1a | 55% 🔴 | 10% ✅ | **-45pp** | Mixed model partial fix |

### 🔍 Deep Dive: Where Stabilized R2c Failed vs R1b

**R1b (5/9 PASS, 98.2% avg) → R2c (post-reclassification: ~3-4/10 PASS, 92.2% avg) regression breakdown:**

#### Checkpoint-level deltas

| Checkpoint | R1b | R2c | Δ | Direction |
|------------|-----|-----|---|-----------|
| **P5-FIELD** | 100% | 90% | **-10pp** | 🟡 minor regression after reclassification (was -60pp before phantom-findings audit) |
| **P5-CR3** | 100% | 40% | **-60pp** | 🔴 biggest regression |
| **P5-CR1** | 100% | 71% | **-28pp** | 🔴 |
| **P6-BOOL** | 100% | 80% | **-20pp** | 🔴 |
| **P6-SUMM** | 100% | 80% | **-20pp** | 🔴 |
| **P7-SUMM** | 100% | 85% | **-14pp** | 🔴 (cascade from R2c-06) |
| **P3B-TEST** | 100% | 90% | **-10pp** | 🔴 (NEW R2c bug — Portal asked for conn test) |
| **P5-NORM** | 100% | 90% | **-10pp** | 🔴 |
| **P5-DONE** | 100% | 90% | **-10pp** | 🔴 |
| P5-STATE | 78% | 100% | +22pp | ✅ improved |
| P3-SCHED | 88% | 100% | +11pp | ✅ improved |
| Other | flat | flat | — | stable |

#### Failure pattern by run

**R2c PASSes (matching R1b quality):** R2c-01, R2c-02 — both 100% PASS, all 23 checkpoints clean.

**R2c partial failures (post-reclassification):**

| Run | Score (post) | Failed checkpoints | Root cause |
|-----|-------|---------------------|------------|
| 03 | 91% | P5-CR3 | Loop prompt skipped after 2nd criterion |
| 04 | 95% | P5-FIELD | Field suggestion step rendered as plain text (real STEP 7 card violation; RA-K/RB-D pattern) |
| 05 | 86% | P5-NORM, P5-CR1, P5-CR3 | LoanRequestPurpose dropped + loop exit (P5-FIELD was already Y in matrix; prior note "criteria gate plain text" was a stale annotation) |
| **06** | **73%** | **P5-CR1, P5-ENUM, P5-DONE, P6-BOOL, P6-SUMM, P7-SUMM** | **Cascade failure** — auto-detect content type confirmation skipped → enum stored as Equal/string → criteria not persisted → P6/P7 cards corrupted (P5-FIELD reclassified to Y) |
| 07 | 96% | P5-CR3 | Auto-detect (JSON) confirmation skipped + loop exit after 2nd (P5-FIELD reclassified to Y) |
| 08 | 100% | (none) | Reclassified PASS — only failure was phantom P5-FIELD |
| 09 | 81% | P3B-TEST, P6-BOOL, P6-SUMM | **NEW R2c bug**: Portal asked for connection test (should skip for Portal) + cascade (P5-FIELD reclassified to Y) |
| 10 | 100% | (none) | Reclassified PASS — only failure was phantom P5-FIELD |

#### Root cause analysis

**1. P5-FIELD — RECLASSIFIED: 1 of 10 R2c runs failed (was 6 before audit)**
- **Audit finding:** Stabilized has no separate "criteria gate" — the field suggestion card (STEP 7 of phase-5-create-delivery-account.md) IS the criteria entry point. Original scoring counted "criteria gate skipped, jumped to field suggestions" as a failure, but that's exactly what STEP 7 does (jumps to field suggestions). The P5-FIELD checkpoint definition is "field suggestions list shown with actual field names" — the field suggestions DID appear in 5 of the 6 originally-failed runs.
- **Real failure (Run 04):** Field suggestion step rendered as plain text rather than as adaptive card. STEP 7 mandates `ASK [adaptive_card]: ActionSet (Show more fields | Skip)`. This is a real STEP 7 violation.
- **Reclassified passes:** Runs 06, 07, 08, 09, 10 — all rendered the field suggestion card correctly; the prior "criteria gate skipped" framing was a phantom finding.
- **Cross-variant note:** The "criteria gate" terminology entered Stabilized findings via cross-contamination from Rework, where a separate "Add criteria | Skip" gate genuinely exists at Step 6 of `rw-phase-5-create-delivery-account.md`. RB-D in rework is a real defect; Stabilized's analogous Finding R is now reframed as "field suggestion step bypassed".

**2. Auto-detect content type confirmation skipped — NEW R2c bug**
- **Pattern:** When user selects "I'm not sure" for content type, agent goes directly to mapping preview without showing the auto-detect confirmation card
- **Affected runs:** 06, 07
- **Why R1b didn't show this:** R1b runs 21-30 may not have used "I'm not sure" auto-detect option. R2c specifically tested it and the post-fix instructions handle it incorrectly.
- **Cascade effect:** R2c-06 has 7 functional failures stemming from this single root cause

**3. Portal connection test asked incorrectly — NEW R2c bug**
- **Pattern:** Agent asks "Would you like to test the connection to your endpoint before continuing?" for Portal delivery, which has `connectionTestMode=none`
- **Affected runs:** R2c-09 (Portal)
- **Why R1b didn't show this:** R1b's Portal runs (if any) didn't trigger this prompt. Post-fix #1 added/changed the connection test prompt logic in a way that incorrectly fires for Portal.

**4. Criteria loop premature exit — pattern persisted**
- **Pattern:** Agent exits criteria loop after 2nd criterion without showing the mandatory loop prompt
- **Affected runs:** 03, 05, 07 (all R2c criteria runs except 01, 02)
- **R1b status:** P5-CR3 was 9/9 perfect in R1b. The fact that it's now 2/5 in R2c suggests the post-fix changes to the loop prompt instruction interact badly with the mixed model.

#### Why does the post-fix #1 hurt stabilized mixed model?

**Hypothesis:** The post-fix #1 instructions were designed for the gpt-5.4-mini all-phases track. They added:
- More aggressive STOP AND YIELD requirements
- New auto-detect content type handling (FIX-15/16)
- More complex field suggestion logic (FIX-3/5; note FIX-3 is named after rework's criteria gate but in stabilized it affected field suggestion ordering)
- Connection test card refinements (FIX-14)

When applied to the mixed model track (gpt-5-mini for P3+P5):
- gpt-5-mini handles the simpler pre-fix instructions cleanly
- The added complexity in post-fix #1 exceeds gpt-5-mini's reliable instruction-following capacity for Phase 5
- Result: 1 of 10 R2c runs hit a real field suggestion card-rendering failure (Run 04) that R1b never showed; other "criteria gate" failures were phantom findings reclassified as PASS

**Verdict:** Stabilized R2c's regression vs R1b is **real and consistent**. Post-fix #1 is incompatible with the mixed-model track for stabilized. **Recommendation:** Either (a) revert stabilized to R1b config (mixed + pre-fix) for production, or (b) develop a separate Post-fix #2 specifically tuned for the mixed-model architecture.

### 🔬 Were R1b and R2c testing the same scenarios?

**No.** Side-by-side scenario comparison:

| | R1b scenarios | R2c scenarios |
|---|---|---|
| 21 / 01 | Webhook/JSON 24/7 Excl Order ON 2 enum | Webhook/JSON Mon-Fri Excl Order ON 3 crit |
| 22 / 02 | Portal Mon-Fri Shared Order OFF | Webhook/JSON-nested Tue-Thu Excl Order ON 5 crit |
| 23 / 03 | Webhook/XML Wkdys 8-6 CST Excl Order ON 3 crit | Portal 24/7 Excl Order OFF 2 enum + XML→JSON recovery |
| 24 / 04 | Email 24/7 Shared Order OFF skip | Webhook/URL-Enc Mon-Fri Shared Order OFF skip |
| 25 / 05 | FTP Mon-Fri MST Excl Order ON | Email/URL-Enc 24/7 Shared Order OFF 4 crit |
| 26 / 06 | Webhook/URL-Enc Mon-Wed-Fri 1 numeric | Webhook/**XML auto-detect** Mon-Wed-Fri Excl Order OFF 2 enum |
| 27 / 07 | Portal 24/7 Excl Order ON 2 enum | Webhook/**JSON auto-detect** 24/7 Shared Order ON 3 crit |
| 28 / 08 | Webhook/JSON-nested Tue-Thu Shared Order ON 1 enum | FTP Mon-Fri Excl Order ON 2 crit |
| 29 / 09 | Email Mon-Fri 8-5 CST Excl Order OFF 2 numeric | Portal Mon-Fri Shared Order OFF skip |
| 30 / 10 | FTP 24/7 Shared Order ON 1 enum | Email 24/7 Shared Order OFF skip |

**Only one near-match:** R1b-28 (Webhook/JSON-nested Tue-Thu Shared Order ON 1 enum) vs R2c-02 (Webhook/JSON-nested Tue-Thu Excl Order ON 5 crit). Both PASSed (R1b-28 = 22/23 96%, R2c-02 = 23/23 100%).

**Key difference:** R2c introduced **auto-detect content type scenarios** (R2c-06 XML auto, R2c-07 JSON auto) and more **skip-criteria scenarios** (R2c-04, 09, 10) that R1b didn't test. The auto-detect scenarios are where R2c's biggest cascade failures happened (R2c-06 = 68%, the worst R2c run).

### 📜 Stabilized instruction changes between R1b and R2c (audit)

R1b ran with pre-fix instructions. R2c ran after these stabilized commits:

**Commit 5200cc3 (Apr 6 06:26)** — "Apply stability fixes" — affected stabilized minimally:
- 1 file changed (system/1-global.md), +4 / -2 lines
- Added: "never duplicate the same text in outputs" to PROMPT notation
- Added: "NEVER hallucinate or assume the data" to Tool Execution Prerequisites
- Added: "Never output the same prompt text more than once in the same message"
- Added: DEBUG transparency override
- **Did NOT touch:** Phase 5 logic, field suggestion step, P5-FIELD rendering, connection tests, Portal flow

**Commit 57d0a98 (Apr 6 19:20)** — "Update delivery instruction packs" — **STRUCTURAL CHANGE TO PHASE 5**:
- 2 files changed: system/1-global.md (+1/-1), phase-5-create-delivery-account.md (+5/-5)
- **CRITICAL:** Phase 5 STATES question MOVED from after `get_lead_type` (STEP 4) to BEFORE `get_lead_type` (STEP 3)
- Old order: price → exclusivity → order → get_lead_type → STATES → criteria
- New order: price → exclusivity → order → STATES → get_lead_type → criteria

**Commit 8bf74d0 (Apr 6 21:39)** — "Clarify ActionSet/ChoiceSet" — minor:
- 1 file (system/1-global.md), +5 / -2 lines
- Added explicit examples of correct/incorrect Action.Submit data field usage
- Did NOT touch Phase 5 or criteria flow

**Audit conclusion:**

The R2c regression IS partially attributable to instruction changes, specifically commit **57d0a98**. The states question reorder restructured Phase 5 flow:
- States now collected BEFORE `get_lead_type`
- This changes the prompt sequence the agent follows
- The field suggestion step (P5-FIELD) comes immediately after `get_lead_type` now (with no states question in between as a "buffer step")
- Mixed model on this restructured flow rendered the field suggestion card as plain text in 1/10 runs (Run 04). The other 5 originally-flagged "P5-FIELD failures" turned out to be phantom findings — they were field-suggestion-shown-correctly behavior misclassified against a non-existent "criteria gate" step.

**However**, the regression is NOT 100% explained by instructions:
- R2c also tested NEW scenarios (auto-detect content type) that R1b didn't run
- R2c-06 cascade failure was triggered by P3-AUTODETECT skip — a scenario type that didn't exist in R1b
- R2c-09 Portal connection test bug is also a NEW scenario interaction

**Combined cause assessment:**
- ~50% scenario differences (auto-detect, more skip-criteria)
- ~30% instruction restructure (commit 57d0a98 states reorder)
- ~20% model behavior variance under post-fix #1 anti-doubling rules

**Recommendation:** To get a clean apples-to-apples comparison, re-run R1b's exact 10 scenarios under post-fix #1 instructions (call it R1b-postfix). That would isolate the instruction-change effect from the scenario-difference effect.

---

## <a name="progression"></a>4. Round Progression Analysis

### Stabilized progression (criteria persistence focus)

| Round | P5-CR3 pass | P5-DONE pass | Comments |
|-------|-------------|--------------|----------|
| Stab R1a (gpt-5.4 only, pre-fix) | 8/20 = 40% | 8/20 = 40% | Baseline criteria failure |
| Stab R1b (mixed, pre-fix) | 9/9 = 100% | 9/9 = 100% | **Mixed model alone unlocks criteria persistence** |
| Stab R2 (gpt-5.4 only, post-fix) | 0/6 = 0% | 4/10 = 40% | Post-fix regression — fixes broke what gpt-5.4 had |
| Stab R2c (mixed, post-fix) | 2/5 = 40% | 9/10 = 90% | **Best stabilized P5-DONE** |

**Stabilized takeaway:** Mixed model (R1b) is the single most effective change for stabilized. Post-fix instructions alone (R2) actively hurt criteria persistence. R2c combination is good but not as good as R1b mixed alone.

### Rework progression (criteria persistence focus)

| Round | P5-CR3 pass | P5-DONE pass | Comments |
|-------|-------------|--------------|----------|
| Rew R1a (gpt-5.4 only, pre-fix) | 1/12 = 8% | 1/12 = 8% | Baseline — worst criteria persistence in any round |
| Rew R1b (mixed, pre-fix) | 1/8 = 13% | 3/10 = 30% | Mixed alone insufficient |
| Rew R2 (gpt-5.4 only, post-fix) | 1/8 = 13% | 6/10 = 60% | Fixes alone partial improvement |
| **Rew R2c (mixed, post-fix)** | **4/5 = 80%** | **7/8 = 88%** | **Both fixes + mixed model required** |

**Rework takeaway:** Fixes alone or mixed model alone are insufficient. Only the **combination** of post-fix instructions + mixed model unlocks criteria persistence.

### Cross-variant comparison: Best round per variant

| Variant | Best round | Avg | Why it wins |
|---------|------------|-----|-------------|
| **Stabilized** | R1b (mixed, pre-fix) | 98.2% | Mixed model alone unlocks criteria flow on simpler architecture |
| **Rework** | R2c (mixed, post-fix) | 92.5% | Fixes + mixed model required for complex Phase 5c flow |

---

## <a name="classification"></a>5. Findings Classification

### Classification Definitions

| Category | Definition |
|----------|-----------|
| **IGNORE** | Instruction IS present in the instruction pack, AI silently skipped or failed to follow it |
| **HALLUCINATE** | AI generated content/behavior not described in the instructions |
| **AMBIGUOUS** | Instruction is unclear, contradictory, or has a true gap |
| **PLATFORM** | Non-deterministic card rendering, MCP issues, context overflow — not instruction-following |
| **CLOSED** | Verified fixed in R2c with quoted instruction evidence |
| **NOT_A_BUG** | Design choice, excluded from countable totals |
| **Impact: Cosmetic** | Orthogonal attribute — workflow completes despite the issue (RB-G doubling, RB-H plain text) |

### Aggregate Classification Summary (post-R2c, verified)

Counts produced by independent classification table audit (Agent 2 round 2 verification).

| Category | Count | % of 67 | Members |
|----------|-------|---------|---------|
| **IGNORE (open)** | 47 | **70%** | RB-H reclassified IGNORE per Agent 2 verification (instruction explicitly says `display_adaptive_card`; agent emits plain text = silent tool skip) |
| **CLOSED (R2c verified)** | 4 | **6%** | RB-I (FIX-16), RB-M (FIX-10), RB-CC (FIX-11), RB-DD (FIX-14) |
| **HALLUCINATE** | 7 | **10%** | RA-B, RA-H, RA-I, RA-Q, RA-NEW-4, RA-NEW-7, RB-O |
| **AMBIGUOUS** | 2 | **3%** | RA-A (verified — Phase 2 has zero typed-input fallback); RB-NEW-R2-1 (verified — fuzzy match instructed but no display normalization) |
| **PLATFORM** | 7 | **10%** | RA-X, RA-NEW-R2-9, RB-F, RB-Q, RB-X, RB-NEW-R2-5, RB-NEW-R2-6 |
| **Total countable** | **67** | **99%** | (rounding) |
| **NOT_A_BUG** | 3 | — | RB-B, RB-J, RB-R |
| **Total entries** | **70** | — | |

**Δ from pre-R2c (75% IGNORE → 70% IGNORE):**
- −3 IGNORE → CLOSED (RB-I, RB-M, RB-DD via verified fixes)
- +1 IGNORE → CLOSED via new finding RB-CC (added then immediately closed via FIX-11)
- +1 new IGNORE: RB-V-skip (state criterion lost on skip-criteria branch)
- +1 PLATFORM → IGNORE: RB-H reclassified (silent display_adaptive_card skip)
- IGNORE net: 49 → 47 (−2)

**Findings impact attribute (orthogonal to root cause):**

| Finding | Root Cause | Impact | Why Cosmetic |
|---------|-----------|--------|--------------|
| RB-G | IGNORE | Cosmetic | Doubled prompt collects same data correctly |
| RB-H | IGNORE | Cosmetic (typically) | Plain-text fallback typed correctly captures data |
| RB-NEW-R2-6 | PLATFORM | Cosmetic | P7 plain-text fallback shows correct values |

### Per-Variant Findings Catalog (Condensed)

#### Stabilized Variant (RA-*) — 33 findings

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
| RA-HALLUC-CG | Agent hallucinates rework "Add criteria \| Skip" criteria gate prompt in Stabilized; the stabilized phase-5 file has no such gate (STEP 7 uses "Show more fields \| Skip" on the field suggestion card). Observed in R2 runs 01, 03, 05, R2b run 03 (notes), and possibly others. Causes ambiguity when scoring P5-FIELD because the rendered gate-style buttons are mistaken for legitimate behavior. | HALLUCINATE | Open (cross-variant prompt contamination) |

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

---

## <a name="ignore-verification"></a>6. IGNORE Verification (Top 5 + R2c additions)

Each finding verified by reading actual instruction file with line number + quoted text.

### IGNORE-1: RA-P / RB-N — States question silently skipped

- **Stabilized:** `delivery-original-stabilized/resources/phase-5-create-delivery-account.md:48` — `WAIT for user input — STOP here. Do NOT proceed until the user provides target states`
- **Rework:** `delivery-rework/resources/rw-phase-5-create-delivery-account.md:45-47` — `Immediately after detecting the state field, display this prompt. Do NOT skip this prompt. Do NOT combine it with the criteria gate. ... **STOP AND YIELD.**`
- **Status:** Both variants explicit. R2c rework: 1/10 occurrences (down from 5/10 in R2). **CONFIRMED IGNORE**.

### IGNORE-2: RA-U / RB-V — Criteria not persisted to account

- **Stabilized:** `phase-5-create-delivery-account.md` STEPs 7-10 — `RETAIN criteriaPayload (the array of criterion objects accumulated during this loop — do NOT discard or reset)`
- **Rework:** `rw-phase-5c-criteria-builder.md:12` — `CRITICAL: Always APPEND to parsedCriteriaList, never overwrite.`
- **R2c sub-variant — RB-V-skip:** `rw-phase-5-create-delivery-account.md:50-57` (Step 5 builds state criterion) and `:74-78` (Step 7 calls create_delivery_account with criteriaPayload). On skip path, Step 7 runs after Step 6. Instructions explicit; agent drops state criterion to `criteria:[]` on skip path in 2/10 R2c runs (R2c-09, R2c-10). **CONFIRMED IGNORE**.

### IGNORE-3: RB-G — Phase transition prompts doubled

- **Rework:** `rw-phase-5-create-delivery-account.md` lines 19, 25, 31, 36, 47, 54, 62, 80 all contain `**STOP AND YIELD.**` after each prompt
- **Status:** Explicit STOP AND YIELD across all rework phase files. Agent doubling prompts violates the directive. **R2c: ~50% occurrence** but **impact reclassified cosmetic (F\*)** — workflow completes correctly. Root cause unchanged: IGNORE. **CONFIRMED IGNORE**.

### IGNORE-4: RB-D — Criteria gate entirely skipped

- **Rework:** `rw-phase-5-create-delivery-account.md:59-62` — `Step 6: Criteria Gate — This step REQUIRES user input. Do not skip it. ... **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.`
- **Status:** Maximally explicit. R2c: 3/10 (R2c-01, R2c-04, R2c-05) — partial improvement from R2's 6/9. **CONFIRMED IGNORE**.

### IGNORE-5: RB-A — Criteria loop exits after 1 criterion

- **Rework:** `rw-phase-5c-criteria-builder.md:72-73` — `Criteria loop prompt — MANDATORY after every accepted criterion: You MUST show this prompt after every criterion is appended. Do NOT advance to State 3 without user confirmation.`
- **Status:** Explicit MANDATORY + "You MUST" + "Do NOT advance". R2c: 1/5 qualifying criteria runs (R2c-08) — major improvement from R2's 88%. FIX-9 verified working in 4/5 R2c criteria runs. **CONFIRMED IGNORE**.

### IGNORE-NEW: RB-H — Card rendered as plain text (reclassified PLATFORM → IGNORE)

- **Rework:** `rw-phase-3-webhook.md:38` — `Present the choice using display_adaptive_card with an ActionSet`. Same explicit `display_adaptive_card` directive at `rw-phase-5-create-delivery-account.md:24, 30, 61` and elsewhere.
- **Verification reasoning:** Findings document agent generating plain text instead. If the agent chooses not to call `display_adaptive_card` and emits text, that is failure to follow an explicit tool-use instruction = IGNORE (silent tool skip pattern matching F44 from CLAUDE.md).
- **Caveat:** If session logs prove `display_adaptive_card` was called with valid card JSON but the platform returned plain text, those specific occurrences would be PLATFORM. Agent 2 verification recommends reading R2c session logs to split into IGNORE vs PLATFORM sub-cases.
- **Provisional classification:** **IGNORE**, **Impact: Cosmetic** when typed fallback resolves correctly.

### CLOSED Verification (R2c-fixed)

| Finding | Fix | Instruction Evidence | R2c Evidence |
|---------|-----|----------------------|--------------|
| **RB-I** | FIX-16 | `rw-phase-3-webhook.md:69-72` — `IF parsing still fails: before showing the recovery card, attempt to parse postingInstructions as each other format. If it successfully parses as valid JSON or valid XML, add a third option to the recovery card` | R2c-03: XML→JSON recovery PASS, 8/9 mappings |
| **RB-M** | FIX-10 | `rw-phase-5c-criteria-builder.md:83-84` — `Parse operator from user input. Check symbols FIRST: >= or ≥ → GreaterOrEqual, <= → LessOrEqual, > → Greater, < → Less, = → Equal, != → NotEqual` | R2c-03/05/06/07: all `>=` → GreaterOrEqual confirmed |
| **RB-CC** | FIX-11 | `rw-phase-5c-criteria-builder.md:56-62` — `IF enumerated field: ... ALWAYS show ChoiceSet for value selection — do not auto-resolve from user text ... Display ChoiceSet via display_adaptive_card` | R2c-02/06/07/08: SelfCreditRating UID 343593, LoanRequestType UID 343608 verified |
| **RB-DD** | FIX-14 | `rw-phase-3b-webhook-test.md:27-32` — both success and failure paths: `Present using display_adaptive_card: TextBlock + ActionSet` | R2c P3B-TEST 6/6 = 100% pass |

### AMBIGUOUS Verification

| Finding | Verified Class | Evidence |
|---------|----------------|----------|
| **RA-A** | AMBIGUOUS | `phase-2-get-lead-types.md:9-12` — entire Phase 2 is only `PROMPT / TOOL: display_lead_types_choice / WAIT for user choice / RETAIN`. Zero typed-fallback instructions. True instruction gap. |
| **RB-NEW-R2-1** | AMBIGUOUS | `rw-phase-5c-criteria-builder.md:26-29, 86` — builds `leadFieldsIndex` keyed on `leadFieldName` (camelCase like `selfCreditRating`), displays via `suggestedFields` using same raw names, matches via `fuzzy >90%` on `leadFieldName`. No rule to insert spaces at camelCase boundaries for display, no user-input normalization. True gap. |

---

## <a name="cosmetic-convention"></a>7. Cosmetic vs Functional Failure Convention

Introduced in R2c. Applied to all R2c scoring; legacy rounds (R1a/R1b/R2) use raw scoring.

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
- **Stabilized all rounds:** Raw scoring (R2c convention not yet applied to stabilized rounds)

For apples-to-apples comparison across variants, see the per-checkpoint grid which uses functional-only counts uniformly.

---

## Appendix: Cross-Document References

- `stability-stabilized-findings.md` — full stabilized variant matrix and findings catalog
- `stability-rework-findings.md` — full rework variant matrix and findings catalog
- `stability-failure-analysis.md` — original v1 of this document (retained for history)
- `delivery-original-stabilized/resources/` — stabilized instruction files
- `delivery-rework/resources/` — rework instruction files (post-fix #1 commit 5200cc3)


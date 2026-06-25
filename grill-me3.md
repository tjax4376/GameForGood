# Grill‑Me Session 3 – SETI Pipeline + Reality Check

**Date:** 2026-06-24  
**Prior sessions:** [grill-me.md](./grill-me.md) · [grill-me2.md](./grill-me2.md)  
**Inputs:** [seti-at-home.md](./seti-at-home.md) · [reality-check-report.md](./reality-check-report.md)  
**Project:** GameForGood – volunteer distributed compute (Gabr Lab TIM-3 / ICOS AML)

---

## Why Session 3

Sessions 1–2 locked privacy, TEE, Gabr workload, UX. Two external reviews exposed **different** gaps:

### From SETI ([seti-at-home.md](./seti-at-home.md))

- Proof tokens alone ≠ science trust — SETI used **redundant quorum + fuzzy compare**
- Backend **never rejoined raw input** — assimilated small result lists, then batch cross-match (Nebula)
- Docking is **stochastic** — Session 1 "deterministic shard" language may be wrong layer
- Screensaver/idle hook + auto-upload drove retention at scale

### From Reality Check ([reality-check-report.md](./reality-check-report.md))

| Finding | Implication for grill |
|---|---|
| **Validated:** no PHI + encryption boundary | Keep — non-negotiable across all options |
| **Gap:** TEE-capable volunteer base may be **90%+ smaller** than SETI's open pool | Every SETI pattern (redundancy, scale) hits harder — must quantify before pipeline lock |
| **Gap:** Gabr partnership engagement reliability | Need SLA, sign-off cadence, fallback if lab slows |
| **Gap:** cross-platform TEE + pharma compute complexity likely exceeds resources | Feasibility spike **before** volunteer-facing code |
| **Hypothesis:** spike TEE workflow on <100 public ligands first | Should be **Q0 gate** — pass/fail before Q1–Q6 |

### Core tension (SETI vs GameForGood)

```
SETI:  millions of PCs · zero special HW · 2–3× redundancy · proof not required
GFG:   TEE-only pool · encrypted IP · attestation required · redundancy TBD
       → same pipeline ideas, much smaller effective fleet
```

Session 3 resolves **feasibility + pipeline** together. Session 2 science targets (TIM-3/ICOS, Gabr) stand unless Q0 forces pivot.

---

## Question map (revised)

```
Q0  Feasibility spike gate (reality-check hypothesis)
Q1  Effective volunteer pool — TEE floor vs scale target
Q2  Result trust — proof vs quorum (SETI) under TEE cost
Q3  Merge semantics — assimilator vs ciphertext-combine (SETI)
Q4  Post-campaign analysis — Nebula-equivalent for enrichment
Q5  Partnership operating model — Gabr engagement SLA (reality-check gap)
Q6  Upload + idle UX — SETI auto-upload vs Session 1 manual submit
```

One question at a time. Dependencies flow Q0 → Q1 → Q2 → …

---

## SETI → GameForGood mapping (reference)

| SETI component | GameForGood analog | Reality-check flag |
|---|---|---|
| Splitter | Provider portal shard generator | — |
| Work unit (~350 KB) | Encrypted `(target, ligand_batch)` shard | TEE decrypt cost |
| Client result (~8 signals) | Encrypted score vector + pose hashes | — |
| 2–3× redundant WU | **Q2** — conflicts with small TEE pool | ⚠️ pool × redundancy |
| Validator (quorum, canonical) | **Q2** | needs tolerance for stochastic docking |
| Assimilator → DB | **Q3** | — |
| Nebula (cross-WU scoring) | **Q4** campaign enrichment job | — |
| Birdies | Session 2 Q6 TIM-3 replay | aligns with Q0 spike |
| No special hardware | **Q1** — opposite assumption | ⚠️ critical gap |

---

## Q0: Feasibility Spike Gate (Reality Check)

**Question:**  
Before locking pipeline or building volunteer client — do you **commit** to the reality-check spike as a hard gate?

Spike scope (from [reality-check-report.md](./reality-check-report.md)):

- Pharmacophore + docking inside **attested TEE** (TDX / SEV-SNP / Apple confidential path)
- Encrypted inputs/outputs in/out of enclave
- Proof token + backend verification
- **Public** structures only, **<100 ligands**
- **One** platform first (not all three) — or all three before proceed?

Decide:

| Branch | Meaning |
|---|---|
| **A. Hard gate** | No `grill-me3` Q1–Q6 locks until spike passes written pass/fail criteria |
| **B. Parallel** | Spike runs while Session 3 continues — pipeline design assumes TEE works |
| **C. Spike-first, pivot-ready** | Spike runs; if fail → Session 3 reopens Session 2 Q4 (TEE tier) for non-TEE fallback |
| **D. Skip spike** | Proceed to volunteer SDK — reject reality-check recommendation |

Also lock spike **pass/fail**:

| Criterion | Suggested bar |
|---|---|
| Enclave boot + attestation quote verifies on backend | Must pass |
| Decrypt → Vina/smina on 50–100 ligands completes in enclave RAM | Must pass |
| No plaintext written to volunteer disk | Must pass |
| Proof token chain validates end-to-end | Must pass |
| Wall-clock per ligand vs bare-metal | Document; no volunteer launch if >10× slower |
| Cross-platform | Spike **one** OS first (recommend Windows TDX) or block until all three? |

**Recommended answer:**  
**C. Spike-first, pivot-ready** — respects reality check without freezing design.

| Field | Choice |
|---|---|
| Gate type | Hard gate on **volunteer client + Gabr in-house shards**; soft gate on Session 3 pipeline **design** (can draft Q1–Q6 while spike runs) |
| Spike platform order | **Windows TDX first** (largest gaming-PC pool) → Linux TDX → macOS M1 confidential VM |
| Pass bar | All four "must pass" rows above |
| Fail pivot | Reopen Session 2 Q4: options = lab-owned cloud workers only, public-data non-TEE campaign, or defer Gabr IP until TEE matures |
| Timeline | Spike ≤4 weeks; no Gabr in-house shard dispatch until spike pass **and** Session 2 Q6 hard gate |
| SETI lesson | SETI shipped Classic client only after splitter + validator worked on lab machines — same "prove backend path on small data first" |

**Why Q0 first:** Reality check says TEE feasibility is the load-bearing assumption. SETI pipeline questions (quorum, assimilate) are moot if TEE docking doesn't work or is 50× too slow.

---

### Answer (Q0): ✅ LOCKED

**Choice:** **C. Spike-first, pivot-ready**

| Field | Choice |
|---|---|
| Gate type | Hard gate on volunteer client + Gabr in-house shards; Session 3 pipeline design proceeds in parallel |
| Spike platform order | Windows TDX → Linux TDX → macOS M1 confidential VM |
| Pass bar | Attestation verifies; decrypt → Vina/smina on 50–100 ligands in enclave RAM; no plaintext on disk; proof chain end-to-end valid |
| Performance | Document wall-clock vs bare-metal; block volunteer launch if >10× slower |
| Fail pivot | Reopen Session 2 Q4: lab-owned cloud workers only, public non-TEE campaign, or defer Gabr IP until TEE matures |
| Timeline | Spike ≤4 weeks; no Gabr in-house shard dispatch until spike pass **and** Session 2 Q6 hard gate |

---

## Q1: Effective Volunteer Pool — TEE Floor vs Campaign Scale

**Question:**  
Reality check: TEE-capable PCs may be **~10%** of install base. SETI ran **millions** of ordinary machines. Gabr TIM-3 campaign (Session 2) targets **in-house library screens** at scale.

Given Q0 spike (assume TEE works at acceptable speed), what is the **minimum effective fleet** you design for — and what happens below that floor?

Decide:

| Field | Need answer |
|---|---|
| **TEE-eligible install rate** | What % of installers pass pre-flight? (model: 5% / 10% / 20%) |
| **Active contributor rate** | Of TEE-eligible, what % run campaigns regularly? SETI ~30–50% churn; pick assumption |
| **Redundancy budget** | Reserve 2× shards for quorum (Q2) or 1× until pool proven? |
| **Campaign throughput target** | Ligands/day needed to hit Session 2 90-day win (TIM-3 + ICOS in-house libs) |
| **Below-floor behavior** | Wait for more volunteers, shrink library, supplement with lab cloud, or pause campaign? |

**Math sketch (for discussion):**

```
effective_ligands_per_day ≈
  (TEE_machines × active_rate × ligands_per_machine_per_day) / redundancy_factor
```

Example: 1,000 TEE machines × 40% active × 500 ligands/day ÷ 2 redundancy = **100,000 ligands/day**

**Options:**

| Option | Fleet assumption | Redundancy | Below floor |
|---|---|---|---|
| **A. SETI-scale ambition** | Target 10K+ TEE machines within 6 months | 2× from day one | Lab cloud backfill for deficit |
| **B. Realistic bootstrap** | Plan for 500–2K TEE machines at launch | 1× until >1K active; then 2× for in-house | Shrink batch size; extend timeline |
| **C. Lab-hybrid default** | 200–500 volunteers + Gabr lab GPU for baseline throughput | 2× on volunteer shards only | Volunteers = burst capacity, not primary |
| **D. Defer scale decision** | No fleet target until spike + 30-day closed beta counts real TEE installs | TBD post-beta | Pause in-house until numbers known |

**Recommended answer:**  
**B. Realistic bootstrap** — honest about TEE filter without giving up volunteer model.

| Field | Choice |
|---|---|
| TEE-eligible rate | Model **10%** of installers pass pre-flight (reality-check midpoint) |
| Active rate | **35%** of TEE-eligible run ≥1 shard/week (SETI-like churn) |
| Launch fleet target | **500 TEE-capable installs** at soft-gate beta; **2,000** before hard-gate in-house |
| Redundancy | **1×** during soft-gate + until ≥1,000 weekly-active TEE machines; **2×** for in-house IP shards once floor met |
| Throughput floor | Minimum **25,000 ligands/day** campaign-wide to justify volunteer narrative vs lab-only |
| Below floor | (1) extend campaign timeline with Gabr, (2) lab GPU backfill for **deficit shards only**, (3) never fake volunteer numbers in UI |
| SETI lesson | SETI never required special hardware — your pool is smaller; **don't copy SETI redundancy until pool math works** |

**Why now:** Q2 (proof vs quorum) depends on whether 2× redundancy is affordable. Reality check + Q0 C mean you design pipeline now but **scale quorum policy to fleet size**.

---

### Answer (Q1): _pending_

---

## Q2: Result Trust Model (preview — blocked on Q1)

_Proof vs redundant quorum under TEE cost. SETI 2–3× same WU; policy scales with Q1 fleet floor._

---

*Session 3 in progress — one question at a time. Answer Q1 to continue.*

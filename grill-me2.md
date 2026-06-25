# Grill‑Me Session 2 – Cancer Research Focus

**Date:** 2026-06-24  
**Prior session:** [grill-me.md](./grill-me.md)  
**Project:** GameForGood – volunteer distributed compute for scientific research  
**Pivot:** Lock architecture against a concrete cancer-research workload, not generic healthcare compute.

---

## Session 1 Recap (what’s already locked)

| Area | Decision |
|---|---|
| Privacy | No PHI in remote datasets; encrypted shards; provider-owned keys |
| Compute | Idle detection (≥99% idle → full capacity); trickle profile otherwise |
| Task model | Independent encrypted shards; deterministic sharding; proof tokens |
| Backend | K8s + job queue; ciphertext-compatible merge; secure FTP delivery |
| Client | Open-source SDK (Apache-2.0); backend closed, provider-hosted |
| Observability | Prometheus/Grafana/Loki/Tempo; shard/hr + cost-offset KPIs |

**Gap:** Session 1 never named a cancer workload, data source, research partner, or success metric. Architecture may be over-built—or wrong—for actual oncology use.

---

## Cancer Research Branch Map (to resolve)

```
Cancer impact goal
├── Workload type (genomics, imaging ML, molecular sim, literature NLP, …)
├── Data source (public vs institutional; PHI risk tier)
├── Research partner (lab, consortium, open science)
├── Scientific output (paper, model, pipeline, drug candidate list)
├── Volunteer fit (embarrassingly parallel? GPU-heavy? latency-sensitive?)
└── Compliance overlay (HIPAA, IRB, GINA, re-identification risk for genomics)
```

Dependencies flow top-down: **workload choice** gates data, sharding, crypto, and partner model.

---

## Q1: Primary Cancer Research Workload

**Question:**  
What is the **first concrete cancer research job** GameForGood will run end-to-end—not “distributed healthcare compute,” but a named task a cancer lab would recognize?

Be specific:
- Input data type (e.g., WGS FASTQ, H&E slides, PDB structures, SMILES, clinical trial CSV)
- Computation (e.g., variant calling, segmentation training, docking, survival model, pathway enrichment)
- Output artifact researchers need (VCF, trained weights, ranked compounds, hazard ratios, …)
- Why volunteer idle compute beats existing options (institutional cluster, Galaxy, Colab, Folding@home, commercial cloud credits)

**Recommended answer:**  
Start with **de-identified, embarrassingly parallel oncology workloads on public or provider-pre-stripped data**, not primary clinical genomics pipelines.

**Strong first candidate:** **Structure-based drug screening (molecular docking / MM-GBSA rescoring)** against cancer-relevant targets (e.g., kinases, immune checkpoint proteins) using public structures (PDB) and compound libraries (ZINC, ChEMBL subsets). Shards = (target, compound batch) pairs; outputs = binding scores + poses; merge = rank-and-filter. No PHI; GPU-friendly; independent shards match Session 1 model; output directly useful to medicinal chemistry groups.

**Alternatives (ranked):**

| Option | Pros | Cons vs your architecture |
|---|---|---|
| Docking / virtual screening | PHI-free, parallel, GPU, clear merge | Crowded space (AutoDock-GPU, etc.)—need differentiation |
| Imaging ML (tumor segmentation on TCGA/TCIA) | High impact | Large shards, model aggregation harder than score merge |
| Secondary genomics (mutation signature, pathway) | Core oncology | Re-identification risk; IRB even on “de-identified” genomics |
| Protein folding (oncology targets) | Popular volunteer narrative | Folding@home dominates; merge is non-trivial |

**Why this question first:** Shard size, proof token contents, GPU vs CPU mix, merge math, and HIPAA posture all depend on workload. Picking docking (or another option) unlocks Q2 (data source + partner).

---

### Answer (Q1): ✅ LOCKED

**Workload:** Structure-based drug screening — molecular docking (+ optional MM-GBSA rescoring) against oncology targets.

| Field | Choice |
|---|---|
| Input | PDB protein structures (kinases, checkpoint proteins) + SMILES/SDF compound batches from ZINC/ChEMBL |
| Computation | Dock compound batch → target; score binding affinity; optional pose refinement |
| Output | Ranked hit list (compound ID, score, pose coordinates) per target |
| Volunteer fit | Embarrassingly parallel (target × batch shards); GPU-heavy; no cross-shard dependency; merge = sort + dedupe |
| vs alternatives | PHI-free; matches Session 1 shard model; faster/cheaper than lab buying cloud GPU for million-compound screens |

---

## Q2: Data Source & Custody

**Question:**  
Where do target structures and compound libraries come from, and who is the **legal data controller** for the campaign?

Decide:
- **Target list** — curated internally, or from a partner lab's priority list (e.g., KRAS G12C, EGFR, PD-L1)?
- **Structures** — RCSB PDB (public) vs AlphaFold DB vs proprietary models from a pharma partner?
- **Compound library** — full ZINC20 (~billions, impractical), ChEMBL oncology subset, Enamine REAL, or partner-provided in-house library?
- **Custody** — GameForGood org, a named university lab, or a hospital provider (Session 1 "provider portal" model)?
- **Licensing** — PDB (CC0/PD), ChEMBL (CC BY-SA), ZINC (varies) — can volunteers redistribute inputs in open-source client?

**Recommended answer:**  
**Public-data campaign, GameForGood as data controller, curated oncology subset.**

| Asset | Source | Rationale |
|---|---|---|
| Target list | 5–10 high-priority oncology targets (EGFR, BRAF V600E, CDK4/6, PD-1/PD-L1, KRAS G12C) curated from open literature | Small enough for v1; high volunteer narrative value |
| Structures | RCSB PDB (experimental) + AlphaFold DB (fallback if no co-crystal) | Free, no PHI, no partner dependency for launch |
| Compound library | ChEMBL 33 oncology-relevant subset (~50K–500K compounds) pre-filtered by Lipinski/PAINS | Manageable shard count; CC BY-SA allows redistribution with attribution |
| Custody | GameForGood (non-profit) hosts object store + campaign metadata | Avoids hospital HIPAA overhead for v1; partner lab is *consumer* of results, not data controller |
| Shard packaging | Provider portal uploads pre-sliced (target, batch) job definitions + structure files to object store | Volunteers pull public inputs; no encrypted PHI shard complexity needed for v1 |

**Why this question now:** Custody determines whether Session 1 HIPAA/encrypted-shard stack is required for v1 or overkill.

---

### Answer (Q2): ✅ LOCKED

**MVP:** Recommended public-data campaign **with mandatory encryption end-to-end — no plaintext, no PHI at any stage.**

| Asset | MVP source | Encryption / handling |
|---|---|---|
| Target list | 5–10 public oncology targets (EGFR, BRAF V600E, CDK4/6, PD-1/PD-L1, KRAS G12C) | Structures encrypted at ingest under campaign key; volunteers decrypt only inside enclave/TDX or process on ciphertext if HE path chosen |
| Structures | RCSB PDB + AlphaFold fallback | Encrypted shards in object store; client never writes plaintext to disk |
| Compound library | ChEMBL oncology subset (~50K–500K), Lipinski/PAINS filtered | Batched into encrypted shards; SMILES never exposed in clear |
| Custody | GameForGood = data controller for MVP campaign | Provider portal model from Session 1 applies from day one |
| Results | Binding scores + pose hashes uploaded encrypted | Merge engine aggregates ciphertext-compatible scores; re-encrypt under controller key for download |

**Hard constraints (user):**
- ❌ No PHI — ever
- ❌ No plaintext structures, SMILES, or results at rest on volunteer machines or object store
- ✅ Session 1 encrypted-shard + proof-token model required for MVP, not deferred

**Migration path to partner targets (post-MVP):**
- Same shard envelope format (`campaign_id`, `target_id`, `batch_id`, ciphertext blob, proof schema)
- Partner uploads proprietary structures/libraries via Provider Portal → encrypted under partner-owned key (Session 1 remote-key model)
- Swap campaign data source only; client SDK, merge engine, and proof verification unchanged
- Partner targets may include confidential but non-PHI IP (undisclosed protein models, in-house libraries)

**Why encrypt public data?** Establishes production security posture before partner IP arrives; prevents volunteer-side leakage of campaign inputs; one code path for MVP → enterprise.

---

## Q3: Research Partner & Success Metric

**Question:**  
Who validates that MVP docking output is **scientifically useful for cancer research**, and what is the **90-day measurable win**?

Decide:
- **Partner for MVP** — named lab now, or validate internally first then recruit partner with results in hand?
- **Partner role** — data controller, wet-lab validator, publication co-author, or advisory only?
- **Gold standard** — how do you prove ranked hits aren't garbage (known actives enrichment, redocking RMSD, literature benchmarks)?
- **90-day deliverable** — screen completion %, enrichment factor, LOI for wet-lab assay, preprint, specific target nomination?
- **Cancer impact claim** — what can you honestly tell volunteers their GPU cycles achieved?

**Recommended answer:**  
**MVP validates internally against ChEMBL known actives; recruit one computational oncology lab as advisory + post-MVP wet-lab path.**

| Field | Choice |
|---|---|
| MVP partner | No binding wet-lab commitment required for launch; pursue 1 computational med-chem / oncology lab (university) as **advisory validator** — reviews methodology, not data controller |
| Gold standard | Per target: hold out ChEMBL known actives (IC50 < 1 µM); measure enrichment in top 1% / top 0.1% of ranked dock scores vs random decoys |
| Redocking sanity | 3–5 co-crystal ligands per target; success = RMSD < 2 Å on native pose before campaign scales |
| 90-day win | (1) 5 targets × 100K compounds screened, (2) ≥3× known-active enrichment in top 1% on ≥3 targets, (3) signed advisory LOI from one lab to assay top 20 compounds if enrichment met |
| Volunteer narrative | "Your idle GPU screened compound X against EGFR — campaign ranked 2.1M poses; 14 known cancer drugs in top 500" |
| Publication | Methods preprint after 90-day win; partner lab optional co-author on validation paper |

**Defer to post-MVP:** Wet-lab assay funding, pharma proprietary target campaigns, NCI partnership formalization.

**Why now:** Partner role determines whether MVP needs IRB (likely no for encrypted public structural data + in-silico only). Success metric gates shard count, target list size, and volunteer messaging.

---

### Answer (Q3): ✅ LOCKED

**Partner:** [Gabr Lab, Weill Cornell Medicine — MI3](https://radiology.weill.cornell.edu/research/molecular-imaging-innovations-institute-mi3/moustafa-gabr-laboratory) (Dr. Moustafa Gabr, Associate Professor of Chemistry in Radiology)

| Field | Choice |
|---|---|
| Method | **Pharmacophore + virtual screening** — lab's published SMAP workflow: (1) key residues from mAb–checkpoint cocrystal, (2) pharmacophore-based VS, (3) hit validation, (4) hit-to-lead |
| MVP targets | **TIM-3** and **ICOS** immune checkpoints in **AML** (aligns with ACS Discovery Boost grant: small-molecule TIM-3 inhibition for AML) |
| Structures | Co-crystal structures from lab's published campaigns (not generic public target list) |
| Compound sets | Lab **in-house compound sets** + literature **known actives** as gold standard |
| Partner role | **Research partner + data controller** for campaign inputs (cocrystals, pharmacophore models, in-house libraries); GameForGood provides volunteer compute orchestration |
| Why volunteer compute | Lab has **constrained academic GPU** — institutional cluster insufficient for large pharmacophore/VS campaigns at scale |
| Published precedent | TIM-3 AML pharmacophore VS (*J Med Chem* 2023); ICOS pharmacophore screening (*ChemMedChem* 2023); LAG-3/ICOS small-molecule inhibitors (*ACS Med Chem Lett*, *RSC Med Chem* 2023) |

**90-day success metric (Gabr-aligned):**

| # | Deliverable | Pass criteria |
|---|---|---|
| 1 | TIM-3 campaign | Complete pharmacophore + VS screen of in-house library; reproduce enrichment of published known actives in top ranks |
| 2 | ICOS campaign | Same for ICOS cocrystal pharmacophore model |
| 3 | Validation | ≥3× enrichment of lab's known actives vs decoys in top 1%; redocking RMSD < 2 Å on native co-crystal ligands |
| 4 | Lab handoff | Encrypted ranked hit list delivered to Gabr Lab portal for TR-FRET / hit-validation assays (lab already runs high-throughput TR-FRET for LAG-3) |
| 5 | Volunteer narrative | "Your GPU screened candidates for TIM-3 inhibitors in AML — Gabr Lab identified 3 of these hits in prior work" |

**Q1/Q2 refinement from partner lock:**
- MVP targets narrow from generic 5–10 oncology list → **TIM-3 + ICOS** (expand to LAG-3 post-MVP; active grant exists)
- MVP compound source shifts from public ChEMBL-only → **Gabr in-house sets** (encrypted, partner-keyed) with ChEMBL known actives as holdout benchmark
- IRB: in-silico only on structural/pharmacophore data, no patient data — confirm with Weill Cornell IRB if lab uploads unpublished structures

---

## Q4: Encryption Implementation Tier (No Plaintext)

**Question:**  
Q2 mandates no plaintext on volunteer machines. **How** does pharmacophore + docking actually run without exposing structures, pharmacophore definitions, or SMILES?

Decide:
- **Compute model** — decrypt inside TEE/enclave then run AutoDock/Vina, vs homomorphic encryption (impractical for docking today), vs lab-owned cloud workers only (defeats volunteer model)?
- **What volunteers see** — ciphertext shard only, or ephemeral decrypt in Secure Enclave / Intel TDX / Apple SEP with attestation?
- **Pharmacophore models** — separate encrypted asset from protein structure; same envelope?
- **Result upload** — encrypted scores only, or scores + encrypted poses?
- **Attestation** — must client prove it ran in enclave before backend accepts proof token?
- **Platform scope** — macOS only (Apple SEP), or cross-platform (TDX/SEV on Linux/Windows)?

**Recommended answer:**  
**TEE decrypt-in-enclave for MVP; defer full HE; partner key never leaves HSM/backend.**

| Layer | MVP choice |
|---|---|
| Shard format | AES-256-GCM encrypted blob: `{pharmacophore_xml \| receptor_pdbqt, ligand_batch_sdf, job_params}` wrapped in Session 1 shard envelope |
| Key model | Gabr Lab = data owner; campaign key derived via provider portal; volunteers get **session-wrapped shard keys** only after auth + attestation |
**Recommended answer:**  
**TEE decrypt-in-enclave for MVP; defer full HE; partner key never leaves HSM/backend.**

| Layer | MVP choice |
|---|---|
| Shard format | AES-256-GCM encrypted blob: `{pharmacophore_xml \| receptor_pdbqt, ligand_batch_sdf, job_params}` wrapped in Session 1 shard envelope |
| Key model | Gabr Lab = data owner; campaign key derived via provider portal; volunteers get **session-wrapped shard keys** only after auth + attestation |
| Volunteer compute | Decrypt **only inside attested enclave**; plaintext exists in enclave RAM only, wiped on job end |
| Docking engine | AutoDock Vina / smina inside enclave — pharmacophore filter pre-step using lab's published workflow libs |
| Proof token | SHA-256(input_shard_hash) + SHA-256(output_scores_hash) + enclave attestation quote — no plaintext in upload |
| Result upload | Encrypted ranked scores + pose hashes (not full coordinates in MVP) pushed on submit |
| Backend merge | Decrypt score vectors on backend HSM only (backend trusted; volunteer edge untrusted) |
| Migration | Same envelope when lab swaps TIM-3 cocrystal v2 or adds proprietary libraries — re-encrypt under new campaign key |

**Why TEE over HE for docking:** Full HE cannot run Vina at practical speed today. TEE satisfies "no plaintext at rest on disk / object store" while allowing real physics-based scoring.

---

### Answer (Q4): ✅ LOCKED

**Tier:** TEE decrypt-in-enclave (recommended) + user constraint **cross-platform from day one — no macOS-first.**

| Platform | TEE technology | MVP priority |
|---|---|---|
| **Windows 10/11** (primary volunteer base) | Intel TDX via Hyper-V isolated VM; fallback AMD SEV-SNP where CPU supports | **P0** — gaming rigs |
| **Linux** (Ubuntu/Fedora) | Intel TDX or AMD SEV-SNP via KVM | **P0** — parity with Windows |
| **macOS** | Apple Secure Enclave / Virtualization.framework confidential guest | **P0** — Apple Silicon M1+; Intel Mac → waitlist |

**Cross-platform client SDK design:**

| Component | Approach |
|---|---|
| TEE abstraction | Rust core trait `ConfidentialExecutor`: probe CPU → select TDX / SEV-SNP / none |
| Install gate | No TEE-capable CPU → client registers but **cannot claim Gabr shards** (join waitlist or redirect to non-IP campaigns later) |
| GPU path | MVP: **CPU docking inside TEE** (Vina/smina CPU). Consumer GPU confidential compute immature — GPU boost deferred to v2 when NVIDIA CC or equivalent available on volunteer hardware |
| Attestation | Backend verifies Intel TDX quote or AMD SNP report before issuing shard key; proof token binds to quote |
| Single installer | One Windows `.msi` + one Linux `.deb`/AppImage; no separate Mac-first rollout |

**Explicitly rejected:** macOS-first rollout; full homomorphic encryption for docking; plaintext fallback.

**Known MVP limitation:** Volunteers without TDX/SEV-capable CPU excluded from IP campaigns — document minimum CPU list (Intel 4th-gen Xeon Scalable+, Intel Core 12th+ with TDX; AMD EPYC Milan+, Ryzen Pro 7000+ with SEV-SNP).

---

## Q5: Volunteer UX & Cancer Narrative

**Question:**  
How do volunteers on **any Windows/Linux PC** understand what they're contributing to, and stay engaged through long pharmacophore/VS campaigns?

Decide:
- **Campaign branding** — GameForGood generic, "Gabr Lab", or named initiative (e.g., "Checkpoint AML Screen")?
- **Transparency level** — can volunteers see target name (TIM-3), disease (AML), or only opaque job IDs (encrypted campaign)?
- **Progress model** — compounds docked, % of library complete, "known actives found in top N"?
- **Consent copy** — what legal/ethical text for in-silico oncology without patient data?
- **Opt-in granularity** — per-campaign (Gabr only), per-target (TIM-3 vs ICOS), GPU/CPU caps?
- **Credit/gamification** — leaderboard, badges, exportable volunteer certificate for Gabr Lab acknowledgment?

**Recommended answer:**  
**Named campaign: "AML Immune Checkpoint Screen" — co-branded GameForGood × Gabr Lab (Weill Cornell).**

| Element | Choice |
|---|---|
| Branding | Campaign card: *"Help Gabr Lab find small-molecule inhibitors for TIM-3 & ICOS in acute myeloid leukemia"* — link to lab page + plain-language 2-min explainer |
| Transparency | Show **target + disease** (public science); hide compound structures and pharmacophore definitions (encrypted IP) |
| Progress | Dashboard: compounds screened / batch complete / campaign % / optional *"X published actives rediscovered in top ranks"* (aggregate only, no structures) |
| Consent | "In-silico only. No patient data. Compute runs in encrypted enclave. You may pause or revoke anytime." |
| Opt-in | Per-campaign toggle; per-target sub-toggle (TIM-3, ICOS); trickle + idle profiles from Session 1 |
| Gamification | Volunteer rank by compounds screened; seasonal badge *"AML Checkpoint Contributor"*; Gabr Lab acknowledgment page listing anonymized volunteer IDs |

**Why now:** Cross-platform TEE may exclude some machines — clear narrative maximizes install-to-retention on eligible hardware.

---

### Answer (Q5): ✅ LOCKED

**Branding:** Co-branded **GameForGood × Gabr Lab (Weill Cornell)** — campaign *"AML Immune Checkpoint Screen"*.

**UI principle:** Volunteer-facing surface is **progress screen only** during active compute — no data inspection, no structure viewer, no plaintext leak surface.

**Progress visualization — "micro-defrag" metaphor:**

| Aspect | Spec |
|---|---|
| Visual | Grid of **small blocks** (smaller than classic disk-defrag UI); blocks start scattered/fragmented |
| Animation | As shard processes, blocks **migrate and coalesce** toward consolidated clusters — evokes defragmentation without copying Windows UX literally |
| Block mapping | 1 block ≈ 1 ligand pose computed (or 1 pharmacophore-filter pass) within current shard; block color = state (pending / computing / done / proof-submitted) |
| Shard completion | When shard finishes, blocks form a **solid compact region** → brief co-brand splash → fade to next shard or idle trickle state |
| Campaign-level | Optional outer ring or second layer showing TIM-3 vs ICOS campaign fraction complete (aggregate block count, no IP) |
| Transparency | Co-brand header shows target + disease (TIM-3 / ICOS · AML); **no** compound names, SMILES, or scores on volunteer UI |
| Consent | Shown once at campaign opt-in; progress screen has pause/revoke only |
| Opt-in | Per-campaign + per-target toggles in settings (not on progress screen) |
| Gamification | Deferred to post-shard summary toast (*"Shard complete — 2,400 compounds screened for TIM-3"*) + seasonal badge backend; no leaderboard on progress screen |

**Platform:** Canvas/WebView or native render in cross-platform client (Tauri recommended) — Windows + Linux P0; macOS P1.

**Design intent:** Satisfying visual feedback without exposing Gabr IP; defrag metaphor communicates "your PC is assembling cancer research" without technical jargon.

---

## Q6: Validation Replay (Published Gabr Benchmarks)

**Question:**  
Before scaling volunteer network on **in-house encrypted libraries**, how do you prove the pipeline reproduces Gabr Lab's **published** pharmacophore + VS results?

Decide:
- **Gold campaigns** — which paper(s) to replay first (TIM-3 AML *J Med Chem* 2023 vs ICOS *ChemMedChem* 2023)?
- **Replay environment** — lab cluster baseline vs GameForGood TEE workers only?
- **Pass criteria** — rank correlation, known-active overlap in top N%, enrichment factor vs published hit list?
- **Who signs off** — Gabr Lab PI, postdoc (Calvo Barreiro / Stavrou), or internal only?
- **Gate** — block public volunteer rollout until replay passes?

**Recommended answer:**  
**Mandatory pre-launch replay of TIM-3 AML campaign (*J Med Chem* 2023, 66(16), 11464–11475) — Gabr Lab signs off before in-house library goes to volunteers.**

| Field | Choice |
|---|---|
| First replay | TIM-3 pharmacophore VS (AML) — most mature grant narrative; published hit list + known actives available |
| Second replay | ICOS pharmacophore campaign (*ChemMedChem* 2023) — run in parallel if lab provides params |
| Baseline | Gabr Lab re-runs on academic GPU → canonical ranked list |
| Volunteer test | 10–50 TEE-capable beta machines (Windows/Linux mix) process **public replay shards** (published structures only, still encrypted in envelope) |
| Pass criteria | (1) Top-100 overlap ≥ 70% with lab baseline, (2) all published actives in top 5%, (3) redocking RMSD < 2 Å on co-crystal ligand, (4) proof-token + attestation chain validates on backend |
| Sign-off | Dr. Gabr or designated postdoc (e.g., Dr. Stavrou — checkpoint drug discovery) written approval |
| Gate | **Hard gate** — no in-house compound shards dispatched until TIM-3 replay passes |

**Why now:** Gabr partnership credibility depends on reproducibility; volunteers see defrag UI but science must match published work first.

---

### Q6 Trade-offs (user asked)

#### A. Which campaign to replay first?

| Option | Upside | Downside |
|---|---|---|
| **TIM-3 AML only first** (*J Med Chem* 2023) | Matches active ACS grant; most complete public narrative; single pass/fail focus | ICOS pipeline bugs may lurk undiscovered until later |
| **ICOS only first** (*ChemMedChem* 2023) | Simpler published set in some cases; tests second major MVP target early | Less alignment with current AML grant story; lab may prioritize TIM-3 |
| **Both in parallel** | Fastest confidence both targets work | 2× lab effort to produce baselines; 2× beta coordination; harder to debug failures |
| **Skip published replay → synthetic benchmark** | Fastest to volunteer beta; tests crypto/TEE/proof only | Gabr credibility risk; volunteers may waste cycles on wrong science; partnership trust hit |

#### B. How strict is the pass bar?

| Strictness | Upside | Downside |
|---|---|---|
| **Strict** (top-100 overlap ≥70%, actives in top 5%, RMSD <2 Å) | High scientific confidence; defensible to Weill Cornell | Slower launch; docking stochasticity may cause false failures |
| **Moderate-strict** (enrichment ≥3×, overlap ≥60%, RMSD <2 Å) | Tolerates cross-machine ranking noise; still scientifically defensible | Requires agreed tolerance with lab |
| **Loose** (proof + attestation only) | Fastest path to real machines | Not science validation; Gabr sign-off unlikely |

#### C. Gate before public rollout?

| Gate | Upside | Downside |
|---|---|---|
| **Hard gate** | No volunteer trust spent until science passes | Launch delay; smaller early community |
| **Soft gate** (public replay beta live; in-house gated) | Builds volunteer base; tests defrag UX + TEE early | Reputational risk if ranks wrong on public replay |
| **No gate** | Fastest growth | Unacceptable for encrypted in-house libraries |

---

### Answer (Q6): ✅ LOCKED

**Gate model:** **Soft gate** — public TIM-3 replay beta live early; **hard gate** on Gabr in-house compound libraries until validation passes.

**Non-negotiable constraint:** **Strictly no PHI at volunteer side** — ever, including during soft-gate beta.

| Field | Choice |
|---|---|
| First replay | TIM-3 AML pharmacophore VS (*J Med Chem* 2023) |
| Soft-gate beta shards | **Published structures + published known actives only** — pre-screened by Gabr + GameForGood compliance checklist; encrypted in TEE envelope but **zero patient-linked fields** |
| PHI enforcement | Shard schema allowlist: `receptor`, `pharmacophore`, `ligand_batch`, `job_params` only — **no** patient IDs, clinical notes, genomic sequences, trial records; backend rejects shards with forbidden keys; client SDK refuses decrypt if attestation policy ≠ `phi_free_campaign` |
| Pass bar (in-house hard gate) | Enrichment ≥3×; top-100 overlap ≥60% vs lab baseline; RMSD <2 Å; proof + attestation chain valid |
| Environment | Gabr lab baseline + 10–20 cross-platform TEE beta volunteers |
| Sign-off | Postdoc validates replay; **PI countersigns before in-house shards** dispatch |
| ICOS | Second replay after TIM-3 soft-gate beta stabilizes |

**Soft vs hard gate split:**

| Phase | Volunteer access | PHI rule |
|---|---|---|
| **Soft-gate beta** | Encrypted published TIM-3 replay shards; co-brand + defrag UI live | No PHI — structural/pharmacophore data only |
| **Hard-gate production** | Gabr in-house compound libraries | Still no PHI — in-house ≠ patient data; IP encrypted, not clinical |

---

## Q7: Minimum Hardware Spec & Non-TEE Messaging

**Question:**  
Cross-platform TEE excludes some PCs. What does installer show **before** volunteer commits, and how do non-qualifying machines degrade gracefully?

Decide:
- **Minimum CPU list** — Intel TDX vs AMD SEV-SNP detection; unified messaging?
- **RAM / disk floor** — enclave memory for docking batches?
- **Pre-install check** — run before download or after install?
- **Non-TEE UX** — waitlist, donate-to-cloud-fund, or redirect to non-IP campaigns later?
- **PHI reminder** — show at install even though campaigns are PHI-free?

**Recommended answer:**  
**Pre-flight checker in installer; clear pass/fail; non-TEE joins waitlist — no shard pull, no PHI, no plaintext ever.**

| Check | Minimum (MVP) |
|---|---|
| CPU | Intel 12th-gen+ with TDX **or** AMD Ryzen 7000+/EPYC Milan+ with SEV-SNP |
| RAM | 16 GB system RAM (8 GB enclave budget for ligand batch) |
| OS | Windows 10 22H2+ / Windows 11, or Ubuntu 22.04+ / Fedora 38+ |
| Disk | 2 GB free for enclave runtime + shard cache (encrypted, wiped on revoke) |
| TEE probe | On install: detect → show green/red badge; red = *"This PC can't run encrypted cancer screens yet — join waitlist"* |
| Non-TEE path | Register account; **cannot claim Gabr shards**; optional email when platform support expands |
| PHI copy | Install screen: *"GameForGood never sends patient data to your PC. All work is in-silico structural screening in an encrypted enclave."* |

---

### Answer (Q7): ✅ LOCKED

**Model:** Recommended pre-flight checker + waitlist for non-TEE — **user amendment: macOS included as full peer platform** (not deferred).

| Check | Minimum (MVP) |
|---|---|
| **Windows** | Intel 12th-gen+ with TDX **or** AMD Ryzen 7000+ with SEV-SNP; Windows 10 22H2+ / Windows 11 |
| **Linux** | Same CPU requirements; Ubuntu 22.04+ or Fedora 38+ |
| **macOS** | Apple Silicon **M1+**; macOS 13 Ventura+; TEE via Secure Enclave + confidential VM path (Virtualization.framework); Intel Macs **not supported** for Gabr shards (no TEE → waitlist) |
| RAM | 16 GB system RAM (8 GB enclave budget for ligand batch) — all platforms |
| Disk | 2 GB free for enclave runtime + encrypted shard cache (wiped on revoke) |
| TEE probe | Single installer per OS; on first launch: detect platform → TDX / SEV-SNP / Apple confidential path → green/red badge |
| Non-TEE path | Register account; **cannot claim Gabr shards**; waitlist email when support expands |
| PHI copy | Install screen (all platforms): *"GameForGood never sends patient data to your PC. All work is in-silico structural screening in an encrypted enclave."* |

**Platform parity (amended from Q4):** Windows · Linux · macOS (Apple Silicon) = **P0**. Intel Mac = waitlist.

---

## Session 2 — Locked Summary

| # | Decision |
|---|---|
| Q1 | Pharmacophore + VS docking; TIM-3 / ICOS AML |
| Q2 | MVP encrypted public replay data; Gabr in-house later; **no plaintext, no PHI** |
| Q3 | **Gabr Lab, Weill Cornell** — partner + controller; constrained academic GPU |
| Q4 | TEE decrypt-in-enclave; cross-platform **Windows + Linux + macOS (M1+)** |
| Q5 | Co-brand *AML Immune Checkpoint Screen*; **micro-defrag progress UI only** |
| Q6 | **Soft gate** published TIM-3 beta; **hard gate** in-house libs; **strict no PHI** on volunteer side |
| Q7 | Pre-flight TEE checker; 16 GB RAM; non-TEE → waitlist; macOS M1+ included |

**Next steps (post-grill):** Requirements block for client SDK → API contract → Gabr TIM-3 replay shard pack → TEE abstraction spike on Win/Linux/macOS.

---

*Session 2 complete.*

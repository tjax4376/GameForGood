# Journal – Grill-Me Session 2 (Cancer Research Focus)

**Version:** 1.0  
**Author:** Cursor Agent  
**Timestamp:** 2026-06-24  
**Change rationale:** Pivot grill session from generic distributed healthcare compute to concrete cancer-research workload selection.

---

## Context

GameForGood is a volunteer distributed compute platform (README + blueprint). Session 1 (`grill-me.md`) locked privacy, sharding, idle detection, K8s backend, and observability—but never named a cancer workload, partner, or success metric. User requested Session 2 with cancer-research focus; output in `grill-me2.md`.

---

## Discussion Points

1. Reviewed `grill-me.md`, `bliueprint.md`, `README.md`—repo is early-stage (docs only, no code).
2. Identified gap: architecture decisions assume encrypted independent shards + ciphertext merge, but oncology workloads vary wildly in PHI risk, parallelism, and merge complexity.
3. Mapped decision tree for cancer pivot; Q1 gates all downstream choices.
4. **Q1 posed:** What is the first concrete cancer research job (data type, computation, output, why volunteer compute)?
5. **Q1 locked (user accepted recommendation):** Molecular docking / virtual screening — oncology targets × ChEMBL/ZINC batches; output = ranked hits; GPU-parallel shards.
6. **Q2 posed:** Data source & custody — public PDB/AlphaFold + ChEMBL subset vs institutional data; who is legal data controller.
7. **Q2 locked (user refinement):** MVP = recommended public sources; mandatory encryption end-to-end (no plaintext, no PHI); same shard envelope for future partner proprietary targets; GameForGood data controller.
8. **Q3 locked (user):** Gabr Lab Weill Cornell — pharmacophore + VS; TIM-3/ICOS in AML; co-crystal structures; in-house compound sets + known actives; constrained academic GPU.
9. **Q4 locked (user):** TEE enclave recommended; cross-platform day one (Windows/Linux P0, macOS P1); no macOS-first; CPU docking in TEE for MVP.
10. **Q5 locked (user):** Co-brand GameForGood × Gabr Lab; progress-only UI; micro-defrag block animation maps to shard compute; no IP on screen.
11. **Q6 locked (user):** Soft gate on published TIM-3 replay beta; hard gate on in-house libs; strict no PHI at volunteer side (schema allowlist + attestation policy).
12. **Q7 locked (user):** Recommended pre-flight + waitlist; macOS Apple Silicon M1+ as P0 peer with Windows/Linux; Intel Mac waitlist.
13. **Session 2 complete** — all Q1–Q7 locked; summary + next steps in grill-me2.md.

---

## Code Changed

| File | Change |
|---|---|
| `grill-me2.md` | Q1–Q3 locked; Q4 added (Gabr-tailored) |
| `journal/2026-06-24-grill-me2-cancer.md` | This journal file |

No application code modified.

---

## Test Plan

N/A (documentation / design session only).

---

## Security / PHA Note

Workload choice determines PHI exposure tier. Q1 recommendation explicitly avoids primary clinical genomics until re-identification and IRB posture are resolved in later questions.

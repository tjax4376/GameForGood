# Grill‑Me Session – Distributed Computing Project

## Initial Questions & Answers

### Q1: Data Privacy & Security Architecture
**Question:** 
- Encryption at rest, in‑transit?
- De‑identification rules?
- Consent model?
- Audit trail capture?

**Answer:**
Data is encrypted by a remote key (owned by the data owner) using provider‑approved HIPAA / GDPR data‑center or cloud keys. The backend is hosted in a provider‑approved datacenter. **No PHI will ever be present** in any dataset that’s processed remotely. Data is sliced into encrypted shards (“bitparts”) that are cryptographically independent. Each user receives only a random subset of shards; the same dataset is never re‑used across users, ensuring that nothing recognizable can be linked or reconstructed locally.

### Q2: Computation Dispatch & Idle‑Cycle Logic
**Questions:**
- Idle detection method (full idle, screen‑saver, CPU/GPU thresholds)?
- Resource throttling algorithm for GPU vs. CPU?
- User opt‑in granularity (per‑project, per‑dataset, time‑based)?
- Result upload approach preserving anonymity?

**Answers:**
Installation defines a “trickle” compute profile (min/max CPU/GPU allocation). When the system enters ≥99 % idle (no foreground activity, CPU/GPU utilization <5 % for X minutes, screensaver active), the SDK escalates usage to the full idle capacity. Users can set per‑profile min/max limits. Results are uploaded only when the user explicitly clicks a "submit" button or at the end of a full processing cycle; the upload includes the hash of the input shard and the proof token.

### Q3: Data Partitioning & Task Model
**Questions:**
- Slice granularity (row‑chunks, column subsets, encrypted shards)?
- Deterministic reproducibility across machines?
- Dependency graph (independent vs. merging required)?
- Verification without re‑execution?

**Answers:**
Use **encrypted shards** as the atomic unit. The sharding method is deterministic (e.g., deterministic blind encryption) so that the same dataset always yields the same shards, regardless of the processing machine. Shards are independent – they do not depend on other shards from the same file. The backend receives each shard along with a cryptographic proof token.

### Q4: Backend Merging & Verification Workflow
**Questions:**
- Ciphertext‑compatible merge strategy (re‑encrypt before/after merge)?
- Proof generation/verification (hash, zk‑SNARK, Merkle?)?
- Conflict handling (duplicate shards from different sources)?
- Replay protection (stale shard re‑submission)?

**Answers:**
Merging uses **ciphertext‑compatible math**: the backend keeps all material encrypted under the provider’s master key and applies plain‑text‑compatible algorithms on ciphertext to combine. Each shard carries a **cryptographic proof token** (hash of input, hash of output, and a SHA‑256 of the computation metadata). If two shards arrive that both claim the same logical range, the later one is parked in a “holding area” and only released for validation after a full dataset verification pass. A shard that is already audited or under verification will **fail merge** and move to the holding area. Resubmitted shards caused by transient failures are allowed through – the backend tracks a simple state (accepted, verified, parked) and does not re‑audit accepted shards.

### Q5: Infrastructure & Scalability
**Questions:**
- Backend stack (K8s, serverless, batch)?
- Auto‑scaling (GPU heavy merge, shard dispatch)?
- Final dataset delivery (direct download, rsync, peering)?
- Open‑source vs. closed (client SDK vs. backend)?

**Answers:**
- **Stack:** Kubernetes clusters are used for orchestration. Auto‑scale based on queue depth and GPU‑node metrics (HPA + custom Prometheus exporter). Shards are submitted to a job queue; workers auto‑scale out on surge.
- **Egress:** The final, re‑encrypted dataset is placed on a **secure FTP site** with a web portal for authenticated downloads.
- **Licensing:** End‑user components will be open‑source (Apache‑2.0). The backend resides in the healthcare provider’s data center and will be **closed‑source**, sold under a one‑time license + support/implementation fees.

### Q6: Monitoring, Governance, and Future Extensibility
**Questions:**
- Observability stack (metrics, alerts, dashboards) for providers and users?
- Governance of the proof standard (standard body vs. internal)?
- Extensibility for new math (floating‑point, stats) with ciphertext compatibility?
- Handling rule changes (HIPAA/GDPR) without breaking existing shards?

**Answers:**
- **Observability:** Grafana dashboards that track shards processed per hour, datasets per hour, CPU idle time saved, and a simple cost offset model (idle compute credits applied to backend costs). Prometheus + Loki + Tempo for metrics and logs.
- **Governance:** The proof token is based on **RSA/AES with SHA‑256** combined with a deterministic hash of the input state. This will be documented in the client SDK release notes; we may later submit it to NIST if the project scales.
- **Extensibility:** Future data types will rely on **ciphertext‑compatible libraries** (e.g., lattice‑based HE) that support floating‑point arithmetic.
- **Compliance Drift:** The backend will house a **policy layer** that can trigger a migration of de‑identification logic. Upon a rule change, any in‑flight shards are re‑enqueued under the new policy.

## Updated Flow Summary (as clarified in follow‑ups)
1. **Provider Portal** → Upload → Object Store.
2. **User Sign‑up** → End‑User Client pulls shards via the same object store.
3. Idle detection triggers compute → Process shard → generate proof.
4. **Push Flow:** End‑User → Portal (secure FTP) pushes processed shard + proof + audit log.
5. **Backend Merge:** Portal receives shard + proof → verify token → perform ciphertext‑compatible merge → re‑encrypt dataset → store on FTP portal for download.
6. **Audit Logging** is part of every shard merge transaction (logged to backend audit store).

## Remaining Considerations (and final guidance)
All critical architecture, security, scale, and compliance questions have been locked down:
- No PHI exposure.  
- Zero‑trust encryption key model.  
- Stateless shard slicing, deterministic re‑processing.  
- Cryptographic proof verification and replay mitigations.  
- Provider‑controlled backend architecture with open‑source client SDK.  
- Observable dashboards aligned to provider cost/benefit KPIs.  
- Extensible design for future ciphertext‑compatible computations.  
- Migration path for future compliance rule changes.

**Blueprint and full grill session now saved.** Next steps: generate initial API skeleton, scaffold idle‑detection SDK, and create the Docker/K8s compose for the backend.

---
*End of Grill‑Me Record*

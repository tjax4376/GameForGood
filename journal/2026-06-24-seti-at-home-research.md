# Journal – SETI@home Research Conversation

**Version:** 1.0  
**Author:** Cursor Agent  
**Timestamp:** 2026-06-24  
**Change rationale:** Capture SETI@home architecture research session and persist to `seti-at-home.md`.

---

## Context

User requested research on how SETI@home built their 90s–00s screensaver, processed datasets, and "joined" results at the backend. GameForGood is a volunteer distributed compute platform (docs-only repo); SETI@home is direct architectural precedent for work-unit sharding, validation, and assimilation.

---

## Discussion Points

1. User asked for ELI10 explanation of SETI@home screensaver, dataset processing, and backend file joining.
2. Researched primary sources: CACM paper, SETI Classic docs, BOINC wiki (validation/assimilation), Nebula manual, 2025 findings paper.
3. Key clarification delivered: volunteers never reassembled raw telescope data; backend "joining" = redundant validation → canonical result → DB assimilation → later Nebula cross-workunit multiplet analysis.
4. User requested `/caveman ultra` compression of same material.
5. User requested `seti-at-home.md` with full conversation persisted.

---

## Code / Files Changed

| File | Action |
|------|--------|
| `seti-at-home.md` | Created — full ELI10 + caveman ultra + sources + GameForGood relevance |
| `journal/2026-06-24-seti-at-home-research.md` | Created — this journal entry |

No application code changed.

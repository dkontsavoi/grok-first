# Team board

Lightweight Jira-style tracker for Main / First / Second / DK.  
**As-of:** 2026-09-06 · Europe/Madrid  
**Repo copy:** `docs/TEAM-BOARD.md` on [grok-first](https://github.com/dkontsavoi/grok-first)

---

## How to use

| Column | Meaning |
|--------|---------|
| **Backlog** | Ideas / suspended — not in flight |
| **Ready** | Specced enough to pick up |
| **In progress** | Someone actively working |
| **Blocked** | Waiting on DK / dependency |
| **Done** | Shipped / closed |

**Ticket fields:** `ID` · `Title` · `Status` · `Owner` · `Priority` · `Links` · `Notes`

Add new tickets under the right status section. Keep IDs sequential: `TB-001`, `TB-002`, …

---

## Board

### Backlog

#### TB-001 — CMC Vault agent API & list sync
| | |
|--|--|
| **Status** | Backlog (suspended) |
| **Owner** | DK → Grok builder |
| **Priority** | P2 |
| **Type** | Infra / product |
| **Created** | 2026-09-06 |
| **Updated** | 2026-09-06 |

**Summary**  
Make [cmc-vault.grok.me](https://cmc-vault.grok.me/) useful as agent infra (not only a browser dig overlay).

**Acceptance (when resumed)**  
1. `GET /api/desk` (+ health/docs)  
2. Restricted-list sync including ARB, DASH, SYRUP, VVV, ZEN  
3. Optional webhook / Arm → Flow 2 backup  
4. Export / RevX basis badge  

**Links**  
- Builder brief: [docs/CMC-VAULT-BUILDER-UPDATES.md](https://github.com/dkontsavoi/grok-first/blob/main/docs/CMC-VAULT-BUILDER-UPDATES.md)  
- Site: https://cmc-vault.grok.me/  

**Notes**  
Suspended by DK 2026-09-06. Until API lands, Vault stays **dig overlay only** (Second/Main cite state/score/`reportedNotReal`). Flow 2 primary remains RevX `active_levels.json`.

---

### Ready

_None_

### In progress

_None_

### Blocked

_None_

### Done

_None_

---

## Changelog

| Date | Change |
|------|--------|
| 2026-09-06 | Board created; TB-001 CMC Vault suspended to Backlog |

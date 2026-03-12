# DEV_CHECKLIST.md — [APP_NAME]
# Master Phase Tracker

> One glance to see where the project stands.
> Detail lives in individual phase checklist files.
> Update this alongside the phase checklists.

---

## Phase Status

| Phase | Name | Status | Checklist |
|-------|------|--------|-----------|
| 1 | Infrastructure | ⬜ Pending | `checklists/phase-1-infrastructure.md` |
| 2 | Auth & Storage Foundation | ⬜ Pending | `checklists/phase-2-auth-storage.md` |
| 3 | Domain Layer Setup | ⬜ Pending | `checklists/phase-3-domain-layer.md` |
| 4 | [Core Feature 1] | ⬜ Pending | `checklists/phase-4-[name].md` |
| 5 | [Core Feature 2] | ⬜ Pending | `checklists/phase-5-[name].md` |
| 6 | Offline Support | ⬜ Pending | `checklists/phase-6-offline.md` |
| 7 | [Feature — flagged] | 🚩 Flagged | `checklists/phase-7-[name].md` |

**Status key:** ⬜ Pending | 🔄 In Progress | ✅ Complete | 🚩 Flagged (not started)

---

## SQL Migration Status

| File | Description | Run? |
|------|-------------|------|
| `sql/001-enable-uuid-extension.sql` | uuid-ossp extension | ⬜ |

---

## Feature Flag Status

| Flag | Default | Current | Enable When |
|------|---------|---------|-------------|
| [FLAG_NAME] | false | false | [condition] |

---

## Test Status

| Phase | Happy Path | Offline | Edge Cases | Regression |
|-------|-----------|---------|------------|------------|
| Phase 1 | ⬜ | N/A | ⬜ | N/A |
| Phase 2 | ⬜ | ⬜ | ⬜ | ⬜ |

---

## Notes / Blockers

- [Add blockers or decisions pending here]

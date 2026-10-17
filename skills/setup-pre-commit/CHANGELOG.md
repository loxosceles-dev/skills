## 2026-10-17 — Align pre-push reminder with setup and documentation policy

**Problem:** The checklist rejected dev in prose even when authorized standalone setup/docs belong there.
**Fix:** Distinguish feature/fix PRs from authorized direct-dev setup/docs in the reminder. The default coding-project hook still blocks direct main pushes.
**Verified:** Shell syntax check and static review; no hook was installed into another repository.

## 2026-10-17 — Keep review stages inside the authorized harness

**Problem:** Mandatory model choices could encourage unavailable-model workarounds outside the user's authorization.
**Fix:** Use the current harness default unless the user requests another available model; remove stage model overrides and require explicit authorization for credential, provider, CLI, or additional-cost changes.
**Verified:** Static review of frontmatter and stage examples confirms model overrides are absent. No cross-provider calls or live behavioral pipeline tests were performed.

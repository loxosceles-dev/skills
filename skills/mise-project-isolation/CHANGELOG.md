## 2026-10-17 — Scope host-native tooling and refresh current skills

**Problem:** Onboarding replayed frozen agent skills despite the latest-upstream
policy, and did not state its boundary with container-managed projects.
**Fix:** Use update --yes followed by install --force; apply mise host-native
examples only to that approved mode, without implicit container removal.
**Verified:** Static onboarding/mode assertions pass. No project runtimes or
dependencies were installed as part of this instruction correction.

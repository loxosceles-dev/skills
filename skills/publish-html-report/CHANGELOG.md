## 2026-10-17 — Preserve current report rules when package names collide

**Problem:** The public copy shadowed newer shared-tool instructions, restoring
light templates and omitting the local-only output branch on host installs.
**Fix:** Recover the shared tool's dark-scheme defaults and local-versus-published
branch in the public compatibility copy. Preserve existing project availability.
**Verified:** Source byte-equality with the current shared copy and assertions for
dark CSS, Mermaid theme, responsive metadata and explicit publication intent.
This is instruction validation, not a browser rendering test.

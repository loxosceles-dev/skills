## 2026-10-17 — Recover explicit infrastructure environment selection

**Problem:** The canonical public copy shadowed a shared copy carrying the
environment convention, so normal host discovery omitted that safeguard.
**Fix:** Recover the generic root-script environment rule without private paths,
identities, history or project details.
**Verified:** Source byte-equality with the existing shared instruction and
assertions for local/dev/prod separation. No infrastructure command was executed.

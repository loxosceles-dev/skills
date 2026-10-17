## 2026-10-17 — Respect project-owned execution modes

**Problem:** An unconditional host-tool ban contradicted approved host-native
migrations and isolated mise workflows. The exceptions also named a retired skill
installer instead of current host administration tooling.
**Fix:** Follow the project's declared container or isolated host-native mode;
ask on ambiguity, retain root scripts, and keep host administration separate.
Keep the shared compatibility copy identical to avoid restoring old rules.
**Verified:** Source equality and static mode/installer assertions pass. No
project migration, build or dependency installation was performed.

## 2026-10-17 — Recover explicit infrastructure environment selection

**Problem:** The canonical public copy shadowed a shared copy carrying the
environment convention, so normal host discovery omitted that safeguard.
**Fix:** Recover the generic root-script environment rule without private paths,
identities, history or project details.
**Verified:** Source byte-equality with the existing shared instruction and
assertions for local/dev/prod separation. No infrastructure command was executed.

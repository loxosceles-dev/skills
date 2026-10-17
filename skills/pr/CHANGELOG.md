## 2026-10-17 — Add concise pull-request description policy

**Problem:** Default PR descriptions added process boilerplate rather than explaining the actual change and exceptional review context.
**Fix:** Respect repository/user templates; otherwise use a concise Summary and only material Special considerations. Keep examples generic and let explicit user requests override defaults.
**Verified:** Static review of frontmatter, package registration, and generic examples; no live PR posting performed.

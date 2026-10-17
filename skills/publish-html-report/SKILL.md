---
name: publish-html-report
description: Generate self-contained HTML reports for local viewing or publish them via gist + htmlpreview. Use when the user asks for a temporary HTML document, an HTML report, or a rich visualization such as an architecture diagram, dependency graph, or before/after comparison.
type: pattern
---

# Publish HTML Report

**This is a reference pattern.** Learn from the approach, adapt to your context — don't copy verbatim.

**Problem**: Markdown is limited for rich visualizations — dependency graphs, before/after architecture diagrams, layered cross-sections, and interactive Mermaid charts don't render well (or at all) in GitHub markdown.

**Solution**: Generate a self-contained HTML file using Tailwind + Mermaid CDNs, publish as a secret GitHub gist, and construct a viewable URL via htmlpreview.github.io.

---

## Pattern

**Workflow**:
```
Generate HTML → Write to docs/  → Push to branch
                                → Create secret gist
                                → Construct preview URL (htmlpreview.github.io)
                                → Link in PR description
```

**Why this works**:
- Secret gists are unlisted but publicly accessible via raw URL
- `htmlpreview.github.io` fetches and renders any raw GitHub HTML
- No GitHub Pages, no paid plan, no hosting infrastructure required

---

## HTML Scaffold

Every report is a single self-contained file — no external assets beyond CDNs:

**Dark scheme:** Always render HTML documents and reports in a dark scheme, independently of the operating system theme. Set `color-scheme: dark`, use dark page and content backgrounds, and maintain high-contrast text, links and diagrams. Apply this preference in the shared tool rather than adding it to project `AGENTS.md` files. Preserve the source content when changing its presentation.

For a temporary or local-only document, save it in the requested temporary directory and open it locally. Publishing is a separate step performed when requested.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>:root { color-scheme: dark; }</style>
    <title>{{Report Title}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "dark", securityLevel: "loose" });
    </script>
  </head>
  <body class="bg-slate-950 text-slate-100 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>
        <h1 class="text-3xl font-bold">{{Title}}</h1>
        <p class="text-sm text-slate-300">{{date}} · {{repo-name}}</p>
      </header>
      <!-- Content sections -->
    </main>
  </body>
</html>
```

**Rules**:
- Only CDN scripts allowed: Tailwind and Mermaid
- No app code, no interactivity beyond Mermaid rendering
- File must render correctly when opened locally (`open file.html`)

---

## Diagram Techniques

Mix these — don't make every diagram look the same:

**Mermaid** — for graph-shaped relationships (call flows, dependencies, sequences):
```html
<div class="rounded-lg border border-slate-600 bg-slate-900 p-4">
  <pre class="mermaid">
    flowchart LR
      A[Module] --> B[Dependency]
      B -.-> C[Leaking concern]
  </pre>
</div>
```

**Hand-built divs** — when Mermaid's layout fights you (mass diagrams, layered cross-sections, module depth comparisons):
```html
<div class="relative border rounded p-4">
  <div class="h-8 border-l-4 border-indigo-500 pl-3 flex items-center text-xs">Interface</div>
  <div class="h-32 border-l-4 border-slate-300 pl-3 flex items-center text-xs">Implementation</div>
</div>
```

**Inline SVG** — for arrows and connecting lines between positioned elements.

---

## Publishing Workflow

### 1. Write the report

Place in `docs/reports/<date>-<slug>.html`. Also produce a markdown fallback at the same path with `.md` extension.

### 2. Create a secret gist

```sh
gh gist create docs/reports/<date>-<slug>.html \
  --desc "<Report Title> — <repo-name> (<date>)"
```

### 3. Construct the preview URL

```sh
GIST_ID=$(gh gist list --limit 1 | head -1 | awk '{print $1}')
RAW_URL=$(gh api gists/$GIST_ID --jq '.files | to_entries[0].value.raw_url')
echo "https://htmlpreview.github.io/?$RAW_URL"
```

### 4. Link in PR description

```markdown
📄 **[Report Title — Visual](<HTMLPREVIEW_URL>)** — opens in browser
📄 **[Markdown fallback](docs/reports/<date>-<slug>.md)** — renders on GitHub
```

### 5. Cleanup convention

Mark `docs/reports/` for deletion before merge:
```markdown
> ⚠️ **CLEANUP**: Delete `docs/reports/` directory before merging.
```

Reports are ephemeral artifacts for review, not permanent documentation.

---

## Integration: Architecture Reviews (mattpocock/skills)

When used with the `improve-codebase-architecture` skill:

1. Run the architecture analysis (exploration, candidate identification)
2. Generate the HTML report following that skill's HTML-REPORT format (candidate cards, before/after diagrams, recommendation badges)
3. Publish using this skill's gist workflow
4. Create a PR with candidates table and preview link
5. Include multi-agent workflow instructions for follow-up:

```markdown
### How to Work on This

1. `/to-prd` — generate PRD from candidates
2. `/to-issues` — one issue per candidate
3. Execute each issue in a separate session
4. `/grill-me` for deeper design decisions
```

---

## Use Cases Beyond Architecture

This pattern works for any visualization that benefits from HTML over markdown:

- **Dependency graphs** — module relationships with colour-coded health
- **Test coverage maps** — visual heat maps of coverage gaps
- **Performance reports** — before/after timing comparisons with charts
- **Migration progress** — visual status of multi-step migrations
- **Data flow diagrams** — complex pipelines with branching logic
- **API surface area** — interface width visualizations

Adapt the scaffold and diagram techniques to the content. The publishing workflow stays the same.

---

## Style Guidance

- Editorial, not corporate-dashboard. Generous whitespace.
- Colour sparingly: one accent (emerald/indigo) + red for problems + amber for warnings
- Diagrams ~320px tall so before/after fits side by side
- `text-xs uppercase tracking-wider` for schematic labels
- Serif optional for headings (`font-serif`) with stone/slate palette

---

## Fallback

If `gh gist create` fails (auth issues, rate limits):
1. Fall back to markdown-only version
2. Include instructions to open HTML locally: `open docs/reports/<file>.html`

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.

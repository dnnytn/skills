---
name: course-creation
description: Self-contained HTML courses — single-file architecture, Playwright E2E via file://, Vercel deployment, scroll-to-unlock gating
version: 1.0.0
source: local-git-analysis
analyzed_commits: 12
---

# learning-stuffs Patterns

## Project Shape

This repo ships **self-contained single-file HTML courses** — all CSS and JS are inlined in the `<style>` and `<script>` tags of one `.html` file. No build step, no bundler. The `courses/` directory is the Vercel output root.

```
learning-stuffs/
├── courses/
│   └── mcp-server.html     ← entire course: HTML + CSS + JS
├── tests/
│   └── toc-collapse.spec.js ← Playwright E2E via file:// URL
├── vercel.json              ← outputDirectory=courses, root rewrite
├── playwright.config.js
└── package.json             ← devDep: @playwright/test only
```

## Commit Conventions

Two distinct patterns — use the right one for the context:

| Context | Pattern | Example |
|---------|---------|---------|
| Infrastructure / tooling / UI features | Conventional commits with bullet-body | `feat: collapsible TOC panel + Playwright E2E tests + Vercel config` |
| Content writing (new sections) | Narrative with chapter progress suffix | `Write §8 Topologies, §9 Authorization, §10 Airgapped (chapter 5 of 7)` |

Conventional commit types used: `feat:`, `chore:`, `fix:`, `refactor:`.

When writing content commits, append `(chapter N of M)` to communicate incremental progress at a glance.

## CSS Architecture

All design tokens live in `:root` custom properties. Never hardcode palette, spacing, or shadows directly:

```css
:root {
  --bg: #0f1217;
  --surface: #171b22;
  --surface-2: #1f2430;
  --border: #2a303c;
  --accent: #5aa9ff;
  --accent-dim: #2a5a92;
  --text: #e6e6e6;
  --text-muted: #9aa4b2;
  --code-bg: #0b0e13;
  --shadow: 0 1px 0 rgba(255,255,255,0.02) inset, 0 8px 24px rgba(0,0,0,0.25);
}
```

Dark theme is the default and only theme for this repo.

## Layout: Collapsible TOC Sidebar

The `.layout` grid has a collapsible left sidebar. Collapse state is toggled on `.layout` via the `.toc-collapsed` class and persisted to `localStorage`:

```js
function setCollapsed(collapsed) {
  layout.classList.toggle('toc-collapsed', collapsed);
  btn.setAttribute('aria-expanded', String(!collapsed));
  btn.setAttribute('aria-label', collapsed ? 'Expand navigation' : 'Collapse navigation');
  try { localStorage.setItem('tocCollapsed', collapsed ? '1' : '0'); } catch (e) {}
}
if (localStorage.getItem('tocCollapsed') === '1') setCollapsed(true);
```

CSS drives the visual collapse — `grid-template-columns` narrows to `48px` (desktop) or `max-height` collapses to `52px` (mobile). The `.toc-inner` display is `none` when collapsed.

## Playwright E2E Tests

Tests run against a local `file://` URL — no dev server needed:

```js
const htmlPath = path.resolve(__dirname, '../courses/mcp-server.html');
const FILE_URL = 'file:///' + htmlPath.replace(/\\/g, '/');
```

### Test Structure

Separate `describe` blocks per viewport; state reset helper before stateful tests:

```js
async function clearTocState(page) {
  await page.evaluate(() => {
    try { localStorage.removeItem('tocCollapsed'); } catch (e) {}
  });
}

test.describe('Desktop (1440px)', () => {
  test.use({ viewport: { width: 1440, height: 900 } });
  // ...
});

test.describe('Mobile (375px)', () => {
  test.use({ viewport: { width: 375, height: 667 } });
  // ...
});
```

### What to Test

For every interactive UI component, cover:
1. Page loads without JS errors (capture `page.on('pageerror', ...)`)
2. Element visible on initial load
3. Toggle/interaction collapses/expands
4. Toggle again reverses
5. Main content accessible in both states
6. Persisted state survives `page.reload()`

### Running Tests

```bash
npm test           # runs playwright test
```

## Vercel Deployment

```json
{
  "outputDirectory": "courses",
  "rewrites": [{ "source": "/", "destination": "/mcp-server.html" }]
}
```

Root `"/"` rewrites to the course HTML. Add new courses to `courses/` and add a new rewrite entry.

`.vercelignore` keeps `node_modules` and `test-results` out of the deployment.

## Content Authoring Workflow

When adding new course sections to the HTML file:

1. Write 2–3 sections per commit — enough to be reviewable but not overwhelming
2. Use `§N Title` notation for section references in both HTML anchors (`id="s-N"`) and commit messages
3. Commit message includes which sections were added and a `(chapter N of M)` progress marker
4. After completing all sections, make a final commit to add navigation, diagrams, or UI polish

## Accessibility Checklist

- `aria-expanded` and `aria-label` must be updated whenever toggle state changes
- Navigation uses `aria-label="..."` on `<nav>` elements
- Slide navigation has `aria-label="Slide navigation"`
- Focus-visible outline on all interactive controls: `outline: 2px solid var(--accent); outline-offset: 2px`

# Cyberpunk Terminal Theme Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply a cyan/blue cyberpunk terminal aesthetic with JetBrains Mono font, electric green metadata, and high-contrast body text — working in both dark and light mode.

**Architecture:** Congo loads color schemes from `assets/css/schemes/{colorScheme}.css` and merges `assets/css/custom.css` automatically — no template changes needed. We create a `cyberpunk.css` scheme (RGB color tokens consumed by Tailwind utility classes) plus a `custom.css` for font imports and prose overrides. One `params.toml` line switches the active scheme.

**Tech Stack:** Hugo, Congo v2 theme, Tailwind CSS (pre-compiled), Google Fonts (JetBrains Mono)

---

### How Congo's color system works (read before coding)

Congo defines CSS custom properties as raw RGB triplets (no `rgb()` wrapper):

```css
--color-primary-400: 0, 229, 255;
```

Tailwind utility classes consume them like this:

```css
.text-primary-400 { color: rgba(var(--color-primary-400), 1); }
```

Dark mode is toggled by adding `.dark` to the `<html>` element (via `appearance.js`). The compiled Tailwind CSS has rules like:

```css
.dark\:text-neutral-400:is(.dark *) { color: rgba(var(--color-neutral-400), 1); }
```

Key mappings that drive our design decisions:

| CSS variable | Where Congo uses it | Our value |
|---|---|---|
| `--color-neutral` | Light page bg + dark body text | `248, 249, 255` |
| `--color-neutral-50` | Dark mode prose headings | `245, 248, 255` (near-white) |
| `--color-neutral-300` | Dark mode prose body | `245, 248, 255` (#f5f8ff) |
| `--color-neutral-400` | Dark mode metadata/footer/breadcrumbs | `57, 255, 20` (electric green) |
| `--color-neutral-500` | Light mode metadata/footer | `26, 138, 0` (dark green) |
| `--color-neutral-700` | Light mode prose body | `7, 8, 15` (#07080f near-black) |
| `--color-neutral-800` | Dark mode page bg (`dark:bg-neutral-800`) | `5, 13, 26` (#050d1a) |
| `--color-neutral-900` | Light mode main text (`text-neutral-900`) | `3, 8, 18` (near-black) |
| `--color-primary-400` | Dark mode links | `0, 229, 255` (electric cyan) |
| `--color-primary-600` | Light mode links | `0, 102, 255` (neon blue) |
| `--color-secondary-400` | Dark mode inline code | `57, 255, 20` (electric green) |

---

## File Map

| Action | Path | Purpose |
|---|---|---|
| **Create** | `assets/css/schemes/cyberpunk.css` | Congo color scheme — all RGB token values |
| **Create** | `assets/css/custom.css` | JetBrains Mono import + prose overrides |
| **Modify** | `config/_default/params.toml` | Switch active scheme to `cyberpunk` |

---

### Task 1: Create the cyberpunk color scheme

**Files:**
- Create: `assets/css/schemes/cyberpunk.css`

- [ ] **Step 1: Create the color scheme file**

```css
/* Cyberpunk scheme */
:root {
  /* Base white — light mode page background */
  --color-neutral: 248, 249, 255;
  /* Near-white — dark mode prose headings */
  --color-neutral-50: 245, 248, 255;
  --color-neutral-100: 225, 235, 255;
  --color-neutral-200: 190, 210, 255;
  /* Near-white — dark mode prose body text */
  --color-neutral-300: 245, 248, 255;
  /* Electric green — dark mode metadata (date, read-time, footer, breadcrumbs) */
  --color-neutral-400: 57, 255, 20;
  /* Dark green — light mode metadata */
  --color-neutral-500: 26, 138, 0;
  --color-neutral-600: 40, 70, 160;
  /* Near-black — light mode prose body */
  --color-neutral-700: 7, 8, 15;
  /* Deep navy — dark mode page background (dark:bg-neutral-800) */
  --color-neutral-800: 5, 13, 26;
  /* Near-black — light mode main text (text-neutral-900) */
  --color-neutral-900: 3, 8, 18;
  --color-neutral-950: 2, 5, 12;

  /* Primary: cyan-blue scale (links, interactive elements) */
  --color-primary-50: 240, 249, 255;
  --color-primary-100: 220, 242, 255;
  --color-primary-200: 180, 230, 255;
  --color-primary-300: 100, 210, 255;
  /* Electric cyan — dark mode links */
  --color-primary-400: 0, 229, 255;
  --color-primary-500: 0, 180, 220;
  /* Neon blue — light mode links */
  --color-primary-600: 0, 102, 255;
  --color-primary-700: 0, 70, 200;
  /* Deep blue — light mode heading target */
  --color-primary-800: 0, 51, 170;
  --color-primary-900: 0, 30, 120;
  --color-primary-950: 0, 15, 80;

  /* Secondary: green scale (inline code, tags) */
  --color-secondary-50: 240, 255, 230;
  --color-secondary-100: 210, 255, 190;
  --color-secondary-200: 160, 255, 120;
  --color-secondary-300: 100, 255, 60;
  /* Electric green — dark mode inline code */
  --color-secondary-400: 57, 255, 20;
  --color-secondary-500: 30, 200, 0;
  --color-secondary-600: 26, 138, 0;
  --color-secondary-700: 18, 100, 0;
  --color-secondary-800: 12, 70, 0;
  --color-secondary-900: 8, 48, 0;
  --color-secondary-950: 4, 28, 0;
}
```

- [ ] **Step 2: Verify the file exists**

```bash
ls assets/css/schemes/cyberpunk.css
```

Expected: file listed (no error)

---

### Task 2: Create custom CSS overrides

**Files:**
- Create: `assets/css/custom.css`

- [ ] **Step 1: Create the custom CSS file**

```css
/* Cyberpunk terminal theme — font & prose overrides */

/* Load JetBrains Mono from Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap');

/* Apply monospace font globally */
html,
body,
h1, h2, h3, h4, h5, h6,
.prose,
nav,
footer {
  font-family: 'JetBrains Mono', 'Courier New', monospace !important;
}

/* Prose body font size: 14px (Congo default is 16px via 1rem) */
.prose {
  font-size: 0.875rem;
}

/* Light mode: prose headings → deep blue */
.prose :where(h1, h2, h3, h4, h5, h6):not(:where([class~="not-prose"] *)) {
  color: rgba(0, 51, 170, 1);
}

/* Dark mode: prose headings → electric cyan */
.dark .prose :where(h1, h2, h3, h4, h5, h6):not(:where([class~="not-prose"] *)) {
  color: rgba(0, 229, 255, 1);
}
```

- [ ] **Step 2: Verify the file exists**

```bash
ls assets/css/custom.css
```

Expected: file listed (no error)

---

### Task 3: Activate the cyberpunk scheme

**Files:**
- Modify: `config/_default/params.toml` line 5 (`colorScheme`)

- [ ] **Step 1: Update params.toml**

Change `colorScheme = "congo"` to:

```toml
colorScheme = "cyberpunk"
```

Also set dark as the default appearance (optional but consistent with the design):

```toml
defaultAppearance = "dark"
```

- [ ] **Step 2: Verify Hugo can build without errors**

```bash
hugo --minify 2>&1 | grep -E "ERROR|WARN|Built"
```

Expected output contains something like: `Built in Xms`  
No `ERROR` lines should appear.

---

### Task 4: Visual verification

- [ ] **Step 1: Start the Hugo dev server (if not already running)**

```bash
hugo server -D
```

Open http://localhost:1313 in your browser.

- [ ] **Step 2: Verify dark mode**

With the browser open, check:
- Page background is deep navy (`#050d1a`)
- Post titles / headings are electric cyan
- Body/paragraph text is near-white and clearly readable at 14px
- Date, reading time, footer text are electric green
- Font is monospace (JetBrains Mono)

- [ ] **Step 3: Verify light mode**

Click the appearance toggle (or use browser dev tools to remove the `.dark` class from `<html>`). Check:
- Background is cool white
- Headings are deep blue
- Body text is near-black and clearly readable
- Date, reading time, footer text are dark green
- Font remains monospace

- [ ] **Step 4: Commit**

```bash
git add assets/css/schemes/cyberpunk.css assets/css/custom.css config/_default/params.toml
git commit -m "feat: apply cyberpunk terminal theme with JetBrains Mono"
```

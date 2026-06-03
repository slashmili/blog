# Cyberpunk Terminal Theme Design

**Date:** 2026-06-03  
**Status:** Approved

## Overview

Redesign the milad-blog visual theme to a cyberpunk/hacker aesthetic using monospace typography, a cyan-blue-on-dark color palette, and green captions. Works in both dark and light mode via Congo's built-in `autoSwitchAppearance`.

The blog uses Hugo with the **Congo v2 theme** (Tailwind CSS). Congo supports custom color schemes via CSS files in `assets/css/schemes/` and arbitrary CSS overrides via `assets/css/custom.css`.

---

## Color Palette

### Dark Mode
| Role | Color | Hex |
|------|-------|-----|
| Background | Deep navy | `#050d1a` |
| Headings / titles | Electric cyan | `#00e5ff` |
| Body text | Near-white | `#f5f8ff` |
| Captions / metadata | Electric green | `#39ff14` |
| Links / accents | Neon blue | `#0066ff` |
| Borders / dividers | Neon blue @ 20% | `#0066ff33` |

### Light Mode
| Role | Color | Hex |
|------|-------|-----|
| Background | Cool white | `#f8f9ff` |
| Headings / titles | Deep blue | `#0033aa` |
| Body text | Near-black | `#07080f` |
| Captions / metadata | Dark green | `#1a8a00` |
| Links / accents | Neon blue | `#0066ff` |
| Borders / dividers | Deep blue @ 20% | `#0044cc33` |

---

## Typography

- **Font:** JetBrains Mono (Google Fonts), fallback `'Courier New', monospace`
- **Body text size:** 14px (base `text-sm` in Tailwind)
- **Applied to:** all text — body, headings, nav, captions, code blocks

---

## Implementation

### 1. `assets/css/schemes/cyberpunk.css`
Custom Congo color scheme file. Congo uses CSS custom properties as RGB triplets (e.g. `--color-primary-500: 0, 229, 255`).

- `--color-primary-*` → cyan/blue scale (headings, links, interactive)
- `--color-secondary-*` → green scale (captions, tags, metadata)
- `--color-neutral-*` → dark navy scale (backgrounds, body text)

### 2. `assets/css/custom.css`
Overrides for:
- Load JetBrains Mono from Google Fonts (`@import`)
- Apply monospace font to `body`, `h1–h6`, `nav`, `prose`
- Set body text size to `14px`
- Style article metadata / captions to use the green secondary color

### 3. `config/_default/params.toml`
- `colorScheme = "cyberpunk"`
- `defaultAppearance = "dark"`
- `autoSwitchAppearance = true` (already set)

---

## Constraints

- No layout changes — only visual/CSS layer is touched
- Congo theme files in `_vendor/` are read-only; all overrides go in the project's `assets/` directory
- The `_vendor/` directory should not be committed (already gitignored via Hugo module vendoring conventions)

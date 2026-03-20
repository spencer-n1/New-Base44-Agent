# WordPress Elementor — Reference File
**Version:** 3.1 — March 19, 2026
**Pair with:** `wordpress-elementor.md` (execution file — use mid-build)

This file explains the WHY behind every rule in the execution file. Read once at session start. Do not re-read mid-build — use the execution file for that.

---

## Why functions.php Is Non-Negotiable

The Elementor JSON only applies CSS class names to widgets (e.g. `_css_classes: "ss-bubble"`). The actual CSS rules that make those classes do anything live exclusively in functions.php. If you push JSON but skip functions.php:

- Bubble/pill labels = plain unstyled text
- Cards = no border-radius, no dark background
- Buttons = may not have outline style
- Nav = does not exist
- Page looks completely broken

There is no recovery path that doesn't involve writing functions.php. Do not call a build done until it's written.

---

## Why Cookie Auth — Not Application Passwords

WordPress Application Passwords are shown **once** at creation. If the token isn't saved immediately, it's gone. Cookie auth via `/wp-login.php` + `/admin-ajax.php?action=rest-nonce` is always reproducible — you can re-authenticate any time with credentials. Never rely on Application Passwords for automated builds.

The cookie file lives at `/tmp/wp_cookies.txt`. The nonce is fetched fresh each session. Both are reliable and repeatable.

---

## Why Python — Not Bash Heredoc

Bash heredoc strips or corrupts certain characters — PHP dollar signs, JavaScript template literals, escaped quotes. Any PHP or JS content written through a heredoc can silently break. Python `subprocess` passes content as raw bytes with no transformation. Always Python.

---

## CSS Targeting — Deep Rules

### Why `.elementor-column { width: 100% }` Destroys Layouts

Elementor controls column widths via inline `flex-basis` styles set directly on the element. Any CSS `width` or `flex` override using `!important` beats Elementor's inline styles. Setting `width: 100%` on `.elementor-column` forces every column in every multi-column layout to full width — they stack vertically instead of sitting side by side. This is the single most destructive CSS rule that can be written. Never write it.

### Why `.e-con.e-flex { justify-content: center }` Clips Cards

This targets every flex container on the page — including card rows. When a card row container has `justify-content: center` applied globally, the cards overflow their parent on both sides and get clipped symmetrically. The visual result is cards that appear correctly centered but are actually being cut off. Always target specific containers by data-id.

### Why `overflow: hidden` Rules Are Banned

Any `overflow: hidden` on `.e-con-inner`, `.elementor-widget-wrap`, or `.elementor-element-populated` will clip the right or both sides of multi-column card grids. It's a common instinct to add this for "clean" layout edges — it always breaks something. Never use.

### Why `body { overflow-x: hidden }` Is Banned

This masks the symptom (horizontal scroll caused by card overflow) without fixing the cause. The real fix is always correcting the container width or the card flex settings. Never mask layout bugs with body overflow.

### The e-con-inner Inner Wrapper Problem

Elementor flex containers with `content_width: "boxed"` render an extra `.e-con-inner` div inside the outer container element. Setting `justify-content` only on the outer `.elementor-element-XXXXX` selector is not enough — the content is actually rendered inside `.e-con-inner`. You must always target both:

```css
.elementor-element-XXXXX { justify-content: center !important; }
.elementor-element-XXXXX > .e-con-inner {
    justify-content: center !important;
    align-items: center !important;
    width: 100% !important;
}
```

### ⚠️ CRITICAL: `_css_classes` Is Silently Dropped on Column Elements Inside e-con Containers

**Confirmed in DOM inspection (March 19, 2026):** Elementor silently drops `_css_classes` from column elements (`elType: "column"`) that are nested inside `e-con` flex containers. The class appears in the JSON, the push returns success, but the rendered HTML element has no custom class — only Elementor's generated classes like `elementor-col-33`.

This affects:
- Card column classes (e.g. `xpo-card-col`) — class never appears in DOM
- Any structural class applied via col() that you intend to use in CSS

**The consequence:** Any CSS rule targeting `.xpo-card-col` or similar column classes will never fire. The elements are completely invisible to class-based CSS. This is NOT an error — there's no warning and the JSON push succeeds.

**The only reliable approach for styling column elements:**
Always target by `data-id` after rendering. The pattern:

```css
/* Instead of: .xpo-card-col { flex: 1 1 0 } */
/* Do this: */
.elementor-element-268ovahe,
.elementor-element-lrjob16w,
.elementor-element-lydp0fdb {
    flex: 1 1 0 !important;
    width: 0 !important;
    min-width: 200px !important;
}
```

**Mandatory post-render step:** After every build, fetch the rendered HTML, extract `data-id` values for all card columns and containers that need specific CSS, then write the CSS targeting those data-ids. Never assume a class applied to a column via `_css_classes` will appear in the DOM.

**What still works with `_css_classes`:** Widget elements (heading, button, image) — classes on these DO appear in the DOM and can be used for CSS targeting.

**Sections and top-level containers:** `_css_classes` on top-level `elType: "section"` elements DOES appear in the DOM class list. The bug is specific to columns inside e-con containers.

Always target inner columns by their `data-id` instead.

### Why Card Background and Border-Radius Must Be in col() JSON

CSS class backgrounds and border-radius on Elementor column elements are unreliable because Elementor applies its own inline styles directly on the element. Those inline styles have higher specificity than even `!important` CSS rules in some cases. Setting `background_color`, `border_border`, `border_color`, and `border_radius` directly in the column's JSON settings forces Elementor to render them as widget properties — which always works. If you are about to write `border-radius` in CSS for a card column — stop. It goes in `col()`.

### Why nth-child Breaks on Sections and Containers

Elementor wraps sections and containers in extra divs during rendering. The actual DOM structure has more elements than the JSON structure, which means nth-child indexes don't match what you'd expect from the JSON. Always use `data-id` selectors. **Exception:** nth-child IS safe on `.elementor-column .elementor-widget-heading` because columns don't inject extra wrapper divs between their direct widget children.

### Hello Elementor Global Reset — Why First

Hello Elementor ships with default column hover states, border styles, and transitions that activate on `.elementor-element-populated`. If you apply card styling globally to `.elementor-element-populated`, it will also style outer section wrappers — creating unwanted borders around entire page sections. The reset must come first to clear the theme defaults. Then apply card styling only to specific column data-ids.

### Why JSON `align: "center"` Doesn't Always Center Text

The Elementor heading widget has an `align` setting in its JSON. In theory this sets text alignment. In practice it doesn't always propagate to the rendered `.elementor-heading-title` element due to CSS specificity layering in Elementor's generated styles. For any section where text must be centered, always add a scoped `text-align: center !important` rule in SS_STYLES_BLOCK:

```css
.{section-class} .elementor-heading-title { text-align: center !important; }
```

Never set this globally — it will center-align sections that should be left-aligned.

---

## Typography — Deep Extraction Guide

### How to Measure From a Screenshot

When you have a reference screenshot and no dev tools access:

1. **Find the baseline** — section bubble labels are almost always the smallest text on the page, typically 10-12px. Use this as your anchor.
2. **Calculate ratios** — measure the pixel height of the headline element relative to the bubble. If the headline is 5-6x taller, it's approximately 55-72px.
3. **Check weight visually** — headlines are clearly bold (700+), body text is regular (400), buttons are semi-bold (600).
4. **Look at line breaks** — if the headline breaks at a specific word, that tells you the approximate max-width. If it's 2 lines at ~720px viewport, the max-width is probably 700-750px.
5. **Check letter-spacing on bubbles** — bubble labels almost always have elevated letter-spacing (0.1-0.15em). Look for the spaced-out uppercase look.

### Sanity Check Ranges — Use Only to Validate, Never as Defaults

| Element | Typical Range | Weight | Notes |
|---------|---------------|--------|-------|
| Bubble label | 10-12px | 600 | Uppercase, letter-spacing 0.1-0.14em |
| Hero headline | 64-80px | 700-800 | Use ss-hero-h1 — Elementor typography fields unreliable |
| Hero subheadline | 16-20px | 400 | max-width ~520px. Use ss-hero-sub |
| Section headline | 32-42px | 700 | Always assign ss-section-h2 |
| Section subheadline | 14-18px | 400 | Muted color |
| Card number | 56-72px | 900 | Low opacity (0.15-0.25) |
| Card title | 18-22px | 600-700 | |
| Card body | 13-15px | 400 | line-height 1.5-1.6 |
| Button text | 12-14px | 600 | |
| Trust badge | 11-13px | 500 | |

### Why ss-section-h2 Instead of h2.elementor-heading-title

Elementor generates a CSS rule `.elementor-size-default` that targets `.elementor-heading-title` with a specific font-size. This rule has high specificity and overrides plain `h2.elementor-heading-title` rules. A class-specific selector like `.ss-section-h2 .elementor-heading-title` wins the specificity battle. This is why every non-hero section headline widget must have the `ss-section-h2` CSS class assigned — targeting the tag alone is unreliable.

---

## Color Extraction — Deep Guide

### What to Document Per Section

For every section in the reference, you need:

| Color | What to Extract |
|-------|-----------------|
| Section background | The fill behind all content |
| Card background | The card fill — should contrast with section bg |
| Card border | The card outline — often rgba for subtle effect |
| Card border-radius | Extract from reference — typically 8-24px |
| Primary text | Headlines, card titles |
| Secondary text | Body copy, descriptions — usually muted/lower opacity |
| Accent text | Highlighted words, colored gradient portions |
| Button background | Solid button fill |
| Button text | Text on solid button |
| Outline button border | Border color on outline-style buttons |
| Trust badge background | Pill background — often slightly elevated from section bg |
| Trust badge border | Pill border — usually rgba |
| Divider | Horizontal rules, vertical column dividers |

### Background Image Sections

When the reference shows a full-bleed background image on a section, do not attempt to replicate it. Use a solid color that matches the overall palette and darkens the section appropriately. Document this in Step 0 as "bg: [hex] — reference shows image, using solid palette match."

### Alternating Section Colors

If the reference alternates between dark backgrounds (e.g. `#0a0a0a` and `#111111` or `#1a1a1a`), document each section's exact color separately. Never assume all sections share the same background.

---

## Container Width — When Boxed vs Full

Elementor flex containers have a `content_width` setting:

- `"boxed"` — content is constrained to ~1200px centered column
- `"full"` — content spans the full viewport width

**Use `"boxed"` for:** all content groups — hero content, card rows, header groups, button rows, badge rows. This creates the Relume/Base44 look where content sits in a centered max-width column. Cards at 33.333% of 1200px = ~400px each. Cards at 33.333% of 1920px = 640px wide — way too big.

**Use `"full"` for:** outer section wrappers when a background color or image needs to bleed edge-to-edge, or for truly full-viewport-width visual elements.

The default in the execution file's `container()` helper is `"boxed"` for this reason.

---

## Card Layout — Full Explanation

### Why Container → Columns (Not inner_sec)

`inner_sec` (inner section with `isInner: true`) creates a nesting structure of Section → Column → Section → Column → Widgets. This bloats the Elementor panel with extra layers, creates unexpected spacing behaviors, and is deprecated in modern Elementor. The correct pattern is Container → Columns → Widgets, which produces a clean flat structure that Elementor renders predictably.

### Why col() JSON for Border, Not CSS

See "Why Card Background and Border-Radius Must Be in col() JSON" in the CSS Targeting section above.

### Equal-Width Card Fix

When card columns (`elementor-column` elements) sit inside a flex container (`e-con-boxed`), their `col-33` / `col-34` percentage widths break. Those percentages are calculated relative to a section's width — but inside a flex container, the sizing context is different. The fix forces equal widths using `flex: 1 1 0` which distributes available space equally regardless of the declared percentage:

```css
.elementor-element-{CARD_ROW_ID} > .e-con-inner > .elementor-column {
    flex: 1 1 0 !important;
    width: 0 !important;
    min-width: 0 !important;
    max-width: none !important;
}
```

`flex: 1 1 0` means: grow equally, shrink equally, start from zero base width.

### Card Feature List Spacing

When cards contain a feature list (a series of short bullet-style heading widgets stacked vertically), the default container gap creates too much space between items. Use `gap=6` inside the card container and apply tighter margin CSS:

```css
.{card-class} .elementor-widget-heading .elementor-heading-title { margin-bottom: 2px !important; }
.{card-class} .elementor-widget-heading:first-child .elementor-heading-title { margin-bottom: 10px !important; }
.{card-class} .elementor-widget-heading:nth-child(2) .elementor-heading-title { margin-bottom: 14px !important; }
```

---

## ReactBits Components — Full Pipeline

Any component from [reactbits.dev](https://reactbits.dev) can be embedded as a self-contained JS bundle.

**The design prompt specifies which component to use.** Do NOT default to Beams or any other component without explicit instruction.

### Get Source

```bash
curl -s "https://raw.githubusercontent.com/DavidHDev/react-bits/main/src/content/Backgrounds/{Name}/{Name}.jsx" > Component.jsx
curl -s "https://raw.githubusercontent.com/DavidHDev/react-bits/main/src/content/Backgrounds/{Name}/{Name}.css" > Component.css
```

Repo structure: `src/content/{Animations|Backgrounds|Components|TextAnimations}/{Name}/{Name}.jsx`

### Entry Point

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import { ComponentName } from './ComponentName.jsx';

function App() {
  return (
    <div style={{ position: 'fixed', inset: 0, width: '100%', height: '100%' }}>
      <ComponentName {/* props from design prompt */} />
    </div>
  );
}
const el = document.getElementById('reactbits-root');
if (el) createRoot(el).render(<App />);
```

### Vite Config — IIFE Bundle, No Chunking

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      input: { bundle: './main.jsx' },
      output: {
        entryFileNames: 'bundle.js',
        chunkFileNames: 'bundle-[hash].js',
        assetFileNames: 'bundle.[ext]',
        format: 'iife',
        name: 'ReactBitsApp',
        inlineDynamicImports: false,
      }
    },
    minify: true,
  }
});
```

### Build and Deploy

```bash
npm install react react-dom [EXTRA_DEPS] vite @vitejs/plugin-react
npx vite build
# Upload dist/bundle.js to CDN, get public URL
```

### Inject via functions.php

```php
// In wp_body_open action:
<div id="reactbits-root" style="position:fixed;inset:0;z-index:0;pointer-events:none;"></div>

// In wp_head action:
<script src="https://CDN_URL/bundle.js" defer></script>
```

```css
#reactbits-root canvas { position: fixed !important; inset: 0 !important; z-index: 0 !important; }
.elementor-section { position: relative; z-index: 1; }
```

### Component Dependencies

| Component | Extra npm deps |
|-----------|----------------|
| Beams | `three @react-three/fiber @react-three/drei` |
| Aurora / Silk | `three @react-three/fiber` |
| Particles | `@tsparticles/react tsparticles` |
| Most others | just `react react-dom` |

### Verified Beams Bundle

- **URL:** `https://base44.app/api/apps/69b3447b3a57d07e9773887d/files/public/69b3447b3a57d07e9773887d/3c0cc400a_beams.js`
- **Config:** `beamWidth=3, beamHeight=30, beamNumber=20, lightColor="#ffffff", speed=2, noiseIntensity=1.75, scale=0.2, rotation=30`

---

## Inline Symbols — Reference List

Unicode symbols typed directly into a heading widget's `"title"` field are fully supported and render consistently across all browsers and OS. They are just text characters.

**Allowed:**
```
Trust badge decorators:  ✦ ★ ◆ •
Directional arrows:      → ← ↑ ↓ ↗ ↙
Checkmarks / crosses:    ✓ ✗ ✔ ✘
Dividers / bullets:      | · —
Section label accents:   ✦ ◆ ·
```

**Not allowed:**
- Emoji (🖥️ 📄 📊 🚀) — render inconsistently across OS/browsers
- Elementor icon widgets
- Image widgets used as icons

The rule: if the symbol is part of the text string → fine. If it's a separate widget element → never.

---

## Comparison Table Layout

Two-column inner section. Left column = standard side. Right column = premium/us side. Each row of comparison = a heading widget. Style each column separately via data-id selectors.

```python
comparison_section = section([
    col([
        container([
            col([
                heading("Standard", tag="h4"),
                heading("Basic support",        tag="p"),
                heading("Manual updates",       tag="p"),
                heading("No analytics",         tag="p"),
            ], pct=50, bg=FROM_REF, padding=card_pad),
            col([
                heading("With Us", tag="h4"),
                heading("Dedicated support",   tag="p"),
                heading("Automated updates",   tag="p"),
                heading("Full analytics",      tag="p"),
            ], pct=50, bg=FROM_REF, padding=card_pad,
               border_color=FROM_REF, border_radius=FROM_REF),
        ], direction="row", justify="center", align="stretch",
           gap=FROM_REF, wrap="nowrap", width="boxed")
    ], pct=100)
], bg=FROM_REF)
```

---

## Wave Shape Divider

When the reference shows a wave transition between sections (common between hero and the next section, or above the footer):

```css
/* Wave at bottom of section */
.{section-class}::after {
    content: '';
    display: block;
    position: absolute;
    bottom: -1px;
    left: 0;
    width: 100%;
    height: 60px;
    background: url("data:image/svg+xml,%3Csvg viewBox='0 0 1440 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M0,30 C360,60 1080,0 1440,30 L1440,60 L0,60 Z' fill='{next-section-color}'/%3E%3C/svg%3E") no-repeat center/cover;
}
```

Document the color of the section below the wave (it fills the wave shape) in Step 0.

---

## Vertical Dividers Between Columns

Use `::after` pseudo-element on the column — not `border-right` (which affects layout flow):

```css
.elementor-element-{col-data-id}::after {
    content: '';
    position: absolute;
    right: 0;
    top: 20%;
    height: 60%;
    width: 1px;
    background: rgba(255,255,255,0.12);
}
```

Apply to all columns except the last one.

---

## Mistakes — Full Explanations

Brief explanations for each rule in the mistakes table. The table itself is in the execution file.

| # | Full Explanation |
|---|-----------------|
| 3 | Elementor uses inline `flex-basis` for column widths. CSS `width: 100% !important` overrides inline styles and forces all columns to stack vertically. |
| 4 | Elementor's rendering engine strips CSS classes from inner sections during output. The class appears in the JSON but never reaches the DOM. |
| 7 | Building nav as an Elementor sticky section creates z-index conflicts with other elements and requires per-page nav management. The `wp_body_open` approach gives one nav across all pages. |
| 15 | Reference sites almost never use pure `#0a0a0a`. Defaulting to it without checking produces a visible difference from the reference. Always extract. |
| 16 | functions.php contains WP core hooks loaded by the Hello Elementor theme. Replacing it wholesale removes those hooks and can break the theme or the admin. The SS STYLES / SS NAV pattern surgically manages only the custom blocks. |
| 24 | Elementor generates inline `style` attributes on column elements that include background and border properties. These inline styles have higher specificity than external CSS rules, even with `!important` in some contexts. The only reliable approach is setting these properties in the JSON itself so Elementor generates the inline styles correctly from the start. |
| 32 | Same as #24 — card border-radius applied via CSS on column elements is overridden by Elementor's own inline style generation. |
| 33 | Elementor's CSS stack includes a rule targeting `.elementor-heading-title` with text-align from the widget's align setting, but the specificity of this generated rule can be lost to other cascade layers. Explicit `!important` CSS in SS_STYLES_BLOCK is the only guaranteed approach. |

---

*Reference file — read once at session start. For mid-build quick reference → `wordpress-elementor.md`*
*Version 3.0 — March 19, 2026*


### Why Hero CSS Class Doesn't Control Section Height

**Confirmed in DOM inspection (March 19, 2026):** Top-level section elements DO receive their `_css_classes` in the DOM (unlike columns inside e-con). But the hero still doesn't respect `min-height` from CSS targeting `.xpo-hero` because Elementor adds `elementor-section-height-default` to every section that doesn't have height set in the JSON. This class has Elementor-generated CSS attached to it that conflicts.

**The correct approach — two steps, both required:**

Step 1: Set `min_height` in the section JSON settings:
```python
section(..., min_height=90)  # unit: vh — Elementor removes height-default class when this is set
```

Step 2: Reinforce with CSS by data-id (not by class — class specificity may still lose):
```css
.elementor-element-{hero-data-id} {
    min-height: 90vh !important;
    display: flex !important;
    flex-direction: column !important;
    justify-content: center !important;
}
```

**Why both:** The JSON `min_height` setting causes Elementor to switch from `elementor-section-height-default` to `elementor-section-height-min-height`, which removes the conflicting generated CSS. The data-id CSS then reliably applies the height. Using CSS alone without the JSON change doesn't work because `elementor-section-height-default` styles override with equal specificity.
---

## Multi-Color Headlines — Inline Span (Confirmed Working)

**Confirmed March 19, 2026:** Elementor heading widget titles render raw HTML. A headline with a word or phrase in a different color requires only ONE heading widget — no split, no second widget, no container tricks.

```python
heading(
    'Compassionate Rehabilitation Services in <span style="color:#cb9c23;">Riverside, CA</span>',
    tag="h1",
    css_class="xpo-hero-h1"
)
```

The span renders inline inside the `<h1>`. Any valid inline CSS works: `color`, `font-weight`, `font-style`, gradient via background-clip, etc.

**This replaces all previous guidance about multi-color headlines requiring a "plan" or split widget approach.** The Step 0 Content Map field for multi-color headlines now just notes the color hex — no architectural planning needed.
---

## Why Kit custom_css Is Banned

The Elementor Kit has a `custom_css` field accessible via the REST API. It seems like a convenient fallback for injecting CSS without SSH. **Do not use it.**

Reasons:
1. CSS injected via the Kit custom_css field does not consistently render on the frontend — it fires sometimes, fails silently other times
2. There is no reliable way to verify it applied without loading the page source and checking
3. It creates a second CSS injection path that conflicts with functions.php when both are present
4. It cannot be scoped to a specific page — it fires globally, potentially breaking other pages

**The only CSS injection path that works reliably is SSH → functions.php.** If SSH is unavailable, stop and report. Do not silently fall back to Kit CSS.

---

## Default Build Scope — Nav and Footer Are Always Excluded

Every build is content sections only unless the client explicitly names nav or footer as deliverables.

**Why this matters:** The reference site almost always has a nav and footer. An agent that scrapes the reference will see them and assume they should be built. They should not. Nav and footer are either:
- Already present on the WP site (injected globally via functions.php from a previous build)
- Out of scope for the current task

**Hard rule:** If the client sends a reference URL and says "build this page" or "make the landing page" — that means sections only. Nav = excluded. Footer = excluded. If in doubt, ask before building either.

---

## Why Images Are Always placehold.co

This pipeline is a structural transfer, not a finished design. Real images from the reference site:
- Are copyright-protected
- Are optimized for the reference site's layout, not the WP build
- Require file upload and media library management
- Distract from structural review — the client wants to evaluate layout, not imagery

Every image widget in every build gets a placehold.co URL:
```
https://placehold.co/600x400/1a1a1a/555555?text=Image
```
Adjust dimensions to approximately match the reference. Dark background, gray text. Never a real photo, never a blank widget, never an empty src.

---

## Why Two Heading Widgets for Multi-Color Headlines Is Wrong

When a headline has two colors (e.g. white text + gold accent), the instinct is to use two heading widgets stacked vertically. This is wrong because:
1. Vertical stacking creates a gap between the two parts — the headline no longer reads as one continuous line
2. Font size and line-height are set independently on each widget — they will drift
3. Max-width behaves differently on each widget — line breaks don't match the reference
4. CSS targeting is doubled — two widgets, two classes, two rules to maintain

The correct approach is one heading widget with an inline HTML span:
```python
heading(
    'Compassionate Rehabilitation Services in <span style="color:#cb9c23;">Riverside, CA</span>',
    tag="h1", css_class="xpo-hero-h1"
)
```
Elementor heading widget titles render raw HTML. The span fires inline. One widget, one class, one CSS rule.


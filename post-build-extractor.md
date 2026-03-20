# Post-Build Data-ID Extractor
**Version:** 1.0 — March 19, 2026
**Run this after EVERY build, before writing any CSS.**

This is Step 7 of the deployment flow. It is MANDATORY. No CSS can be written until this step is complete.

---

## Why This Exists

Elementor silently drops `_css_classes` from column elements inside e-con containers. Any CSS rule targeting a column by class name will never fire. The ONLY reliable CSS selector for columns, sections, and containers is `data-id`.

This script fetches the rendered HTML and extracts every data-id you need. Run it, paste the output into your CSS block.

---

## The Script

```python
import subprocess, re, time, json

# ── CONFIG — fill these in for each build ──────────────────────────────────────
WP_URL   = "https://YOUR-SITE.cloudwaysapps.com"
PAGE_SLUG = "your-page-slug"      # e.g. "xpo-rehab-group-home"
SLUG     = "xpo"                  # CSS class prefix for this build
# ──────────────────────────────────────────────────────────────────────────────

ts   = int(time.time())
url  = f"{WP_URL}/{PAGE_SLUG}/?nocache={ts}"
html = subprocess.run(['curl', '-s', url], capture_output=True, text=True).stdout

print(f"Fetched {len(html)} chars from {url}\n")

ids = {}

# 1. Hero section (first top-level section)
hero = re.search(r'<section[^>]*elementor-top-section[^>]*data-id="([^"]+)"', html)
ids['hero_section'] = hero.group(1) if hero else "NOT FOUND"

# 2. All top-level sections in order
sections = re.findall(r'<section[^>]*elementor-top-section[^>]*data-id="([^"]+)"', html)
ids['all_sections'] = sections

# 3. Card columns (elementor-col-33 or elementor-col-34)
card_cols = re.findall(r'<div[^>]*elementor-col-3[34][^>]*data-id="([^"]+)"', html)
ids['card_columns'] = card_cols

# 4. All e-con flex containers (button rows, header stacks, etc.)
econs = re.findall(r'<div[^>]*e-con[^>]*e-parent[^>]*data-id="([^"]+)"', html)
ids['econ_containers'] = econs

# 5. Button widgets
buttons = re.findall(r'<div[^>]*elementor-widget-button[^>]*data-id="([^"]+)"', html)
ids['buttons'] = buttons

# 6. Image widgets
images = re.findall(r'<div[^>]*elementor-widget-image[^>]*data-id="([^"]+)"', html)
ids['images'] = images

# 7. Check which CSS classes actually made it to DOM (widget-level only)
for cls in [f'{SLUG}-label-gold', f'{SLUG}-label-blue', f'{SLUG}-hero-h1',
            f'{SLUG}-hero-sub', f'{SLUG}-section-h2', f'{SLUG}-card-title',
            f'{SLUG}-card-body']:
    found = cls in html
    print(f"  {'✓' if found else '✗'} .{cls} in DOM")

print()

# ── OUTPUT CSS BLOCKS ─────────────────────────────────────────────────────────

print("=" * 60)
print("COPY THIS INTO YOUR CSS BLOCK")
print("=" * 60)

print(f"""
/* ── HERO FULL HEIGHT ── */
.elementor-element-{ids['hero_section']} {{
    min-height: 90vh !important;
    display: flex !important;
    flex-direction: column !important;
    justify-content: center !important;
}}
.elementor-element-{ids['hero_section']} > .elementor-container {{
    width: 100% !important;
    flex: 1 !important;
    display: flex !important;
    align-items: center !important;
}}
""")

if card_cols:
    col_selectors     = ',\n'.join([f'.elementor-element-{c}' for c in card_cols])
    col_inner_sel     = ',\n'.join([f'.elementor-element-{c} > .elementor-widget-wrap' for c in card_cols])
    print(f"""/* ── CARD EQUAL WIDTH + STYLING ── */
{col_selectors} {{
    flex: 1 1 0 !important;
    width: 0 !important;
    min-width: 200px !important;
}}
{col_inner_sel} {{
    height: 100% !important;
    display: flex !important;
    flex-direction: column !important;
    padding: 28px 24px !important;
    box-shadow: 0 4px 20px rgba(0,0,0,0.08) !important;
    border-radius: 12px !important;
    background: #ffffff !important;
    border: 1px solid #e5e7eb !important;
}}
""")
else:
    print("/* ⚠️  No card columns found — check page rendered correctly */\n")

if len(econs) >= 2:
    print(f"""/* ── HERO BUTTON ROW — centered ── */
.elementor-element-{econs[1]} {{
    justify-content: center !important;
    display: flex !important;
}}
.elementor-element-{econs[1]} > .e-con-inner {{
    justify-content: center !important;
    align-items: center !important;
    width: 100% !important;
    display: flex !important;
    gap: 16px !important;
}}
""")

if buttons:
    print(f"""/* ── BUTTON CENTERING ── */""")
    for b in buttons:
        print(f""".elementor-element-{b} {{ text-align: center !important; }}""")
    print()

print("=" * 60)
print("DATA-ID SUMMARY")
print("=" * 60)
for k, v in ids.items():
    print(f"{k}: {v}")
```

---

## How to Use

1. Run this script immediately after pushing Elementor JSON and clearing cache
2. Copy the CSS output block
3. Paste it into your `functions.php` CSS section (replacing placeholder comments like `/* CARD CSS HERE */`)
4. Write remaining class-based CSS (labels, typography, nav) separately — those still work fine
5. SSH write functions.php, clear cache, done

---

## What Each Section Does

| Block | Why data-id, not class |
|-------|----------------------|
| Hero full height | `elementor-section-height-default` overrides class-based CSS — data-id wins |
| Card equal width | `_css_classes` silently dropped on columns inside e-con — never reaches DOM |
| Card styling (shadow, radius, bg) | Same — column classes dropped |
| Button row centering | e-con inner wrapper needs both outer + `.e-con-inner` targeted |

---

## Column Class vs Widget Class — The Rule

| Element type | `_css_classes` reaches DOM? | Use for CSS? |
|---|---|---|
| `elType: "section"` (top-level) | ✅ YES | Yes — class works |
| `elType: "column"` inside e-con | ❌ NO — silently dropped | No — use data-id |
| `elType: "widget"` (heading, button) | ✅ YES | Yes — class works |
| `elType: "container"` (e-con) | ✅ YES | Yes — class works |

**Bottom line:** Only column elements inside e-con containers have the bug. Everything else is safe.

---

## Template CSS Block Structure

Every build's CSS in functions.php should follow this order:

```css
/* 1. GLOBAL RESET — always first */
/* 2. CENTERING */
/* 3. BASE (font, admin bar, page title) */
/* 4. DATA-ID BLOCKS — from extractor output */
/*    4a. Hero height */
/*    4b. Card columns equal width */
/*    4c. Card inner styling (shadow, radius, bg) */
/*    4d. Button row centering */
/* 5. CLASS-BASED BLOCKS — labels, typography, nav */
/*    5a. xpo-label-gold / xpo-label-blue */
/*    5b. xpo-hero-h1 / xpo-hero-sub */
/*    5c. xpo-section-h2 / xpo-sub / xpo-card-title / xpo-card-body */
/* 6. NAV */
```

Never mix data-id blocks and class blocks. Data-id always comes first so it can be regenerated cleanly on the next build without touching the class blocks.

---

*Post-Build Extractor v1.0 — March 19, 2026*
*Run after every build. No exceptions.*

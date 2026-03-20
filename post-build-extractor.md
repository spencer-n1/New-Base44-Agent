# Post-Build Data-ID Extractor
# Version: 2.0 — March 20, 2026
# Run this after EVERY build, before writing any CSS.

This is Step 7 of the deployment flow. It is MANDATORY. No CSS can be written until this step is complete.

---

## Why This Exists

Elementor silently drops `_css_classes` from column elements inside e-con containers. Any CSS rule targeting a column by class name will never fire. The ONLY reliable CSS selector for columns, sections, and containers is `data-id`.

**Additionally:** Card GAPS are controlled by the PARENT container, not the cards themselves. You MUST extract the container data-id to enforce the gap via CSS.

---

## The Complete Script

```python
import subprocess, re, time

# ═════════════════════════════════════════════════════════════════════════════
# CONFIG — Fill these in for each build
# ═════════════════════════════════════════════════════════════════════════════
WP_URL    = "https://YOUR-SITE.cloudwaysapps.com"
PAGE_SLUG = "your-page-slug"      # e.g., "xpo-rehab-home"
SLUG      = "xpo"                 # CSS class prefix
# ═════════════════════════════════════════════════════════════════════════════

ts   = int(time.time())
url  = f"{WP_URL}/{PAGE_SLUG}/?nocache={ts}"
html = subprocess.run(['curl', '-s', url], capture_output=True, text=True).stdout

print(f"Fetched {len(html)} chars from {url}\n")
print("=" * 70)
print("EXTRACTING DATA-IDS — Complete Braille Edition")
print("=" * 70)

ids = {}

# ═════════════════════════════════════════════════════════════════════════════
# 1. TOP-LEVEL SECTIONS (in order of appearance)
# ═════════════════════════════════════════════════════════════════════════════
sections = re.findall(r'<section[^\u003e]*elementor-top-section[^\u003e]*data-id="([^"]+)"', html)
ids['sections'] = {
    'all': sections,
    'hero': sections[0] if len(sections) > 0 else None,
    'section_2': sections[1] if len(sections) > 1 else None,
    'section_3': sections[2] if len(sections) > 2 else None,
    'section_4': sections[3] if len(sections) > 3 else None,
}

print(f"\n📄 SECTIONS ({len(sections)} found):")
for i, sid in enumerate(sections, 1):
    print(f"   Section {i}: {sid}")

# ═════════════════════════════════════════════════════════════════════════════
# 2. E-CON FLEX CONTAINERS (all of them)
# ═════════════════════════════════════════════════════════════════════════════
econs = re.findall(r'<div[^\u003e]*e-con[^\u003e]*e-parent[^\u003e]*data-id="([^"]+)"', html)
ids['econ_containers'] = econs

print(f"\n📦 E-CON CONTAINERS ({len(econs)} found):")
for i, eid in enumerate(econs, 1):
    print(f"   Container {i}: {eid}")

# ═════════════════════════════════════════════════════════════════════════════
# 3. CARD COLUMNS (elementor-col-33 or elementor-col-34)
# ═════════════════════════════════════════════════════════════════════════════
card_cols = re.findall(r'<div[^\u003e]*elementor-col-3[34][^\u003e]*data-id="([^"]+)"', html)
ids['card_columns'] = card_cols

print(f"\n🃏 CARD COLUMNS ({len(card_cols)} found):")
for i, cid in enumerate(card_cols, 1):
    print(f"   Card {i}: {cid}")

# ═════════════════════════════════════════════════════════════════════════════
# 4. CARD ROW CONTAINER (parent of card columns)
#    This controls the GAP between cards
# ═════════════════════════════════════════════════════════════════════════════
card_container = None
if card_cols and len(card_cols) >= 2:
    # Find the container that holds the first card column
    pattern = r'<div[^\u003e]*e-con[^\u003e]*data-id="([^"]+)"[^\u003e]*\u003e[\s\S]*?elementor-element-' + card_cols[0]
    match = re.search(pattern, html)
    if match:
        card_container = match.group(1)
        ids['card_container'] = card_container
        print(f"\n🎯 CARD ROW CONTAINER (gap control): {card_container}")
    else:
        print("\n⚠️  Card container not found — gap CSS may not work")
else:
    print("\nℹ️  No card columns found — skipping container extraction")

# ═════════════════════════════════════════════════════════════════════════════
# 5. BUTTON WIDGETS
# ═════════════════════════════════════════════════════════════════════════════
buttons = re.findall(r'<div[^\u003e]*elementor-widget-button[^\u003e]*data-id="([^"]+)"', html)
ids['buttons'] = buttons

print(f"\n🔘 BUTTONS ({len(buttons)} found):")
for i, bid in enumerate(buttons, 1):
    print(f"   Button {i}: {bid}")

# ═════════════════════════════════════════════════════════════════════════════
# 6. IMAGE WIDGETS
# ═════════════════════════════════════════════════════════════════════════════
images = re.findall(r'<div[^\u003e]*elementor-widget-image[^\u003e]*data-id="([^"]+)"', html)
ids['images'] = images

print(f"\n🖼️  IMAGES ({len(images)} found)")

# ═════════════════════════════════════════════════════════════════════════════
# 7. VERIFY CSS CLASSES IN DOM
# ═════════════════════════════════════════════════════════════════════════════
print(f"\n✓ CSS CLASS VERIFICATION:")
expected_classes = [
    f'{SLUG}-label-gold', f'{SLUG}-label-blue',
    f'{SLUG}-hero-h1', f'{SLUG}-hero-sub',
    f'{SLUG}-section-h2', f'{SLUG}-section-sub',
    f'{SLUG}-card-title', f'{SLUG}-card-body', f'{SLUG}-card-link',
    f'{SLUG}-btn-outline', f'{SLUG}-btn-solid'
]
for cls in expected_classes:
    found = cls in html
    status = "✓" if found else "✗"
    print(f"   {status} .{cls}")

# ═════════════════════════════════════════════════════════════════════════════
# OUTPUT: COMPLETE CSS BLOCK
# ═════════════════════════════════════════════════════════════════════════════
print("\n" + "=" * 70)
print("COPY THIS ENTIRE CSS BLOCK INTO functions.php")
print("=" * 70)

# Hero section
if ids['sections']['hero']:
    hero_id = ids['sections']['hero']
    print(f"""
/* ═══════════════════════════════════════════════════════════════════════
   HERO SECTION — Full Height
   ═══════════════════════════════════════════════════════════════════════ */
.elementor-element-{hero_id} {{
    min-height: 90vh !important;
    display: flex !important;
    flex-direction: column !important;
    justify-content: center !important;
}}
.elementor-element-{hero_id} > .elementor-container {{
    width: 100% !important;
    max-width: 1200px !important;
    margin: 0 auto !important;
}}
""")

# Card container gap
if card_container:
    print(f"""
/* ═══════════════════════════════════════════════════════════════════════
   CARD ROW — Gap Control (MATCH TO CONTENT MAP VALUE)
   ═══════════════════════════════════════════════════════════════════════ */
.elementor-element-{card_container} {{
    gap: 24px !important;  /* ← CHANGE TO MEASURED VALUE FROM CONTENT MAP */
    justify-content: center !important;
    display: flex !important;
}}
.elementor-element-{card_container} > .e-con-inner {{
    gap: 24px !important;  /* ← CHANGE TO MEASURED VALUE FROM CONTENT MAP */
    justify-content: center !important;
    width: 100% !important;
}}
""")

# Card columns equal width
if len(card_cols) >= 2:
    col_selectors = ',\n'.join([f'.elementor-element-{c}' for c in card_cols])
    col_inner = ',\n'.join([f'.elementor-element-{c} > .elementor-widget-wrap' for c in card_cols])
    
    print(f"""
/* ═══════════════════════════════════════════════════════════════════════
   CARD COLUMNS — Equal Width
   ═══════════════════════════════════════════════════════════════════════ */
{col_selectors} {{
    flex: 1 1 0 !important;
    width: 0 !important;
    min-width: 200px !important;
    max-width: none !important;
}}

/* Card inner layout */
{col_inner} {{
    height: 100% !important;
    display: flex !important;
    flex-direction: column !important;
}}
""")

# Button centering
if buttons:
    print("""
/* ═══════════════════════════════════════════════════════════════════════
   BUTTONS — Center Alignment
   ═══════════════════════════════════════════════════════════════════════ */
""")
    for bid in buttons:
        print(f".elementor-element-{bid} {{ text-align: center !important; }}")

# E-con containers centering (for button rows, etc.)
if len(econs) >= 2:
    print(f"""
/* ═══════════════════════════════════════════════════════════════════════
   FLEX CONTAINERS — Center Content
   ═══════════════════════════════════════════════════════════════════════ */
""")
    for eid in econs[1:]:  # Skip first (usually hero main container)
        print(f"""
.elementor-element-{eid} {{
    justify-content: center !important;
}}
.elementor-element-{eid} > .e-con-inner {{
    justify-content: center !important;
    align-items: center !important;
}}
""")

# ═════════════════════════════════════════════════════════════════════════════
# DATA-ID SUMMARY
# ═════════════════════════════════════════════════════════════════════════════
print("\n" + "=" * 70)
print("DATA-ID SUMMARY — Save This For Reference")
print("=" * 70)

print(f"""
HERO SECTION:        {ids['sections']['hero'] or 'NOT FOUND'}
CARD CONTAINER:      {card_container or 'NOT FOUND'}
CARD COLUMNS:        {', '.join(card_cols) if card_cols else 'NONE'}
BUTTONS:             {', '.join(buttons) if buttons else 'NONE'}

ALL SECTIONS:        {', '.join(sections)}
ALL E-CONs:          {', '.join(econs)}
""")

print("=" * 70)
print("NEXT STEPS:")
print("1. Copy the CSS block above into functions.php")
print("2. Update gap values to match your Content Map measurements")
print("3. Add class-based CSS (labels, typography) after the data-id blocks")
print("4. Clear cache and verify")
print("=" * 70)
```

---

## Critical Addition in v2.0: Container Gap Extraction

The gap between cards is controlled by the **parent container's** `gap` property, not the cards themselves. The extractor now finds:
- The container that wraps the card columns
- Its data-id for CSS targeting

### Gap Enforcement CSS

After extraction, you get CSS like:
```css
.elementor-element-{CONTAINER_ID} {
    gap: 24px !important;  /* ← Change this to your measured value */
}
```

### What to Change

Replace `24px` with the **exact value from your Content Map**:
```
## CARD LAYOUT — CONTAINER
- Card row gap: 24px  ← This value
```

---

## How to Use

1. Run this script immediately after pushing Elementor JSON and clearing cache
2. Copy the CSS output block
3. Paste it into your `functions.php` CSS section
4. **Update the gap value** to match your Content Map measurement
5. Write class-based CSS (labels, typography) separately — those still work fine
6. SSH write functions.php, clear cache, done

---

## What Each Section Does

| Block | Why data-id, not class |
|-------|----------------------|
| Hero full height | `elementor-section-height-default` overrides class-based CSS — data-id wins |
| Card container gap | Gap lives on parent container, not cards — must target container data-id |
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
/*    4b. Card container gap (NEW in v2.0) */
/*    4c. Card columns equal width */
/*    4d. Card inner styling (shadow, radius, bg) */
/*    4e. Button row centering */
/* 5. CLASS-BASED BLOCKS — labels, typography, nav */
/*    5a. xpo-label-gold / xpo-label-blue */
/*    5b. xpo-hero-h1 / xpo-hero-sub */
/*    5c. xpo-section-h2 / xpo-sub / xpo-card-title / xpo-card-body */
/* 6. NAV */
```

Never mix data-id blocks and class blocks. Data-id always comes first so it can be regenerated cleanly on the next build without touching the class blocks.

---

## Troubleshooting

### "No card columns found"
- Page might not be rendering correctly
- Check if Elementor cache was cleared
- Verify the page slug is correct

### "Card container not found"
- The regex might not match if Elementor changes HTML structure
- Manually inspect HTML to find the parent container
- Look for: `<div class="e-con..." data-id="...">` wrapping the columns

### Gap not applying
- Make sure the gap value in CSS matches your Content Map
- Check if container data-id is correct
- Verify no other CSS is overriding the gap

---

## Version History

- v2.0 (March 20, 2026): Added card container extraction for gap control
- v1.0 (March 19, 2026): Initial extractor — sections, columns, e-cons

---

*Post-Build Extractor v2.0 — March 20, 2026*
*Run after every build. No exceptions.*

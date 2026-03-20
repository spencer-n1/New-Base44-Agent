# WordPress Elementor Build Pipeline
# Version: 3.5 — Complete Braille Edition
# Date: March 20, 2026

---
type: procedural
force_checklist: true
---

## Pipeline Files — What's What

| File | Purpose | When to Read |
|------|---------|--------------|
| `wordpress-elementor.md` | **Execution file** — Step 0/0.5 templates, widget builders, deployment flow | **Before EVERY build** |
| `wordpress-elementor-reference.md` | **Reference file** — Deep explanations, troubleshooting, theory | Once at session start |
| `post-build-extractor.md` | **Data-id extraction script** — Run after every build | After JSON push, before CSS |
| `wordpress-pipeline.md` | **This file** — Index, quick reference | First |

---

## Quick Start (Every Session)

1. **Read** `wordpress-elementor.md` completely (yes, all of it)
2. **Fill out** Step 0 Content Map template — **measure EVERYTHING**
3. **Fill out** Step 0.5 Element Declaration — plan EVERY widget
4. **Build** JSON using widget builders
5. **Push** to WordPress
6. **Run** post-build extractor → get data-ids (including container for gap)
7. **Write** CSS: data-id blocks first, then class blocks
8. **Update** functions.php via SSH
9. **Report** URL and stop

---

## The "Complete Braille" Philosophy (v3.5)

This pipeline treats the reference website like a **tactile map** — every detail is documented, measured, and declared before any code is written. Nothing is assumed. Nothing is "from reference." Everything is explicit.

### What Changed in v3.5:

| Before | After |
|--------|-------|
| "Card gap: from reference" | "Card row gap: 24px (measured from screenshot)" |
| "Blue background" | "Section background: #2D5FA6 (measured from hero area)" |
| "About 64px" | "Hero headline font-size: 72px (measured as 6x the 12px label)" |
| Implicit structure | Explicit Element Declaration — every widget named |
| Gap assumed | Gap enforced via triple-redundancy |

### Card Gap Triple-Redundancy:
1. **Content Map**: Document `Card row gap: Xpx` (measured)
2. **JSON**: Set `flex_gap` in container
3. **CSS**: Extract container data-id, enforce `gap: Xpx !important`

---

## Critical Rules (No Exceptions)

### Default Scope
- **Content sections ONLY**
- Nav = NEVER (unless explicitly requested)
- Footer = NEVER (unless explicitly requested)

### Content = Heading Widgets ONLY
- No text editor widget
- All text uses `heading()` with appropriate `tag`

### Images = placehold.co ONLY
- Never real images from reference
- Format: `https://placehold.co/WxH/BG/TEXT?text=DESC`

### CSS Injection = SSH ONLY
- functions.php via paramiko
- Kit custom_css is BANNED

### Card Gaps = Measured & Documented
- Must be in Content Map: "Card row gap: Xpx"
- Must be in Element Declaration: container gap
- Must be extracted and enforced via data-id CSS

---

## Common Failures & Prevention

| Failure | Cause | Prevention |
|---------|-------|------------|
| Card gaps uneven | Gap not measured in Step 0 | Content Map template forces gap field |
| Wrong colors | "Blue" instead of hex | Content Map requires #HEX for every color |
| Two widgets for multi-color headline | Not using inline span | Element Declaration template shows exact pattern |
| Column classes don't work | Forgot they get dropped | Reference file explains, extractor uses data-id |
| CSS doesn't persist | Used Kit instead of SSH | Execution file enforces SSH only |

---

## Version History

- **v3.5** (March 20, 2026): Complete Braille Edition
  - Crystal-clear Content Map template with mandatory gap measurement
  - Crystal-clear Element Declaration template
  - Post-build extractor v2.0 with container gap extraction
  - Triple-redundancy for card gaps
  - "No 'from reference'" doctrine

- v3.4 (March 19, 2026): Nav/footer exclusion rules, placehold.co image rule, heading widget only, Kit CSS ban
- v3.3 (March 19, 2026): Data-id extraction mandatory
- v3.0 (March 19, 2026): Pipeline split into execution/reference files

---

*Pipeline v3.5 — Complete Braille Edition — March 20, 2026*

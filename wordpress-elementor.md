# WordPress Elementor — Execution File
# Version: 3.5 — Complete Braille Edition
# Date: March 20, 2026
# Pair with: `wordpress-elementor-reference.md` for deep rules

---
type: procedural
force_checklist: true
---

## Re-Read Map — Use This Mid-Build

| When you're about to... | Re-read this section |
|------------------------|---------------------|
| Start any build | This entire file (it's short) |
| Authenticate | → Authentication |
| Create or edit a page | → Page Creation / Edit Process |
| Write JSON | → Widget Builders + Full Page Structure |
| Edit functions.php | → functions.php Pattern (SSH) |
| Write CSS | → CSS Quick Rules |
| Hit a visual bug | → Diagnostic Protocol |
| After every build (mandatory) | → post-build-extractor.md |
| Update the pipeline | → reference file: Deep Rules |

---

# 🛑 MANDATORY PRE-BUILD CHECKLIST

- [ ] Re-read this entire file
- [ ] Step 0: Content Map complete — **every color, font, text, spacing MEASURED**
- [ ] Step 0.5: Element Declaration written
- [ ] Scope confirmed: "Building [X] only"
- [ ] Exclusions confirmed: "Not touching [nav / functions.php / etc]"
- [ ] **DEFAULT SCOPE = content sections only. NAV and FOOTER are EXCLUDED from every build unless the client explicitly requests them by name.**
- [ ] NAV CHECK: never build a nav. Not even if the reference has one. Not even if it looks incomplete without one. Nav = excluded by default, always.
- [ ] FOOTER CHECK: never build a footer. Same rule. Footer = excluded by default, always.
- [ ] Hard stop check: client say "just" or "only"? → strict scope, nothing extra
- [ ] Reference screenshot active in context for entire build

**Note:** Client may waive Step 0.5 approval to go straight to build. That's fine — but Step 0 content map must still be complete before writing any code.

**Hard Stop Phrases** — stop immediately:
- "Just X" / "Only X" → strict scope
- "Pipeline" → you violated a rule, stop and reassess

---

## 🎯 THE GOAL: 100% Reference Match

| Element | Must Match |
|---------|------------|
| Text content | Every word exactly |
| Text size | Exact px from reference |
| Text max-width | Line breaks match reference |
| Colors | Every hex — bg, text, borders, accents |
| Spacing | Gap, padding, margin — pixel-perfect |
| Alignment | Center / left / right — whatever reference shows |
| Border radius | Exact px from reference |
| Border style | Width, color, style |
| Section heights | min-height from reference |
| Card layouts | Image position, text order, everything |

---

## Critical Rules

- **Never work from memory** — re-read the relevant section before each step
- **NO ICONS EVER** — no Elementor icon widget, no image-as-icon, no emoji. Only inline Unicode in text strings (✦ ↗ ✓ etc.) where reference shows them
- **NO TEXT EDITOR WIDGET EVER** — every single piece of text uses a heading widget. Body copy, card text, labels, subheadlines — all heading widgets. Text editor widget is unpredictable, unstyled, and breaks CSS targeting. It does not exist in this pipeline.
- **Copy all text exactly** — never truncate or paraphrase
- **Images are ALWAYS placehold.co** — never pull real images from the reference site. Every image widget uses `https://placehold.co/WxH/1a1a1a/555555?text=Image` (adjust size to match reference). No exceptions.
- **Build once, report, stop** — client does the visual check, not you
- **Pipeline stays universal** — never hardcode build-specific hex, px, or class names into rules. Use `FROM_REF` and `{placeholder}`
- **No inherited CSS namespaces** — every build gets its own `{clientslug}-*` CSS class prefix. Never reuse classes from a previous project.

---

## ⚠️ STRUCTURAL RULES — DO NOT VIOLATE THESE

These are not style preferences. These are the root causes of every repeated bug.

### 1. SECTION LABEL COLORS — Every label widget gets its own unique class

**WRONG:**
```python
# One class for all labels, override by section context
heading("OUR SERVICES", css_class="xpo-label")   # gold
heading("WHO WE ARE",   css_class="xpo-label")   # also gold — BREAKS when override misses
```

**CORRECT:**
```python
# Each label has its own class with its own explicit color. No overrides needed.
heading("OUR SERVICES", css_class="xpo-label-blue")   # always blue, no context needed
heading("WHO WE ARE",   css_class="xpo-label-blue")   # always blue
heading("WELCOME TO",   css_class="xpo-label-gold")   # always gold, in hero only
```

**CSS:**
```css
.xpo-label-gold .elementor-heading-title  { color: #GOLD_HEX !important; font-size: 12px !important; font-weight: 500 !important; text-transform: uppercase !important; letter-spacing: 0.15em !important; }
.xpo-label-blue .elementor-heading-title  { color: #BLUE_HEX !important; font-size: 12px !important; font-weight: 500 !important; text-transform: uppercase !important; letter-spacing: 0.1em !important; }
```

**Rule:** Every distinct label color = its own CSS class. Never one class + context override. Context overrides are invisible and forgotten across builds.

---

### 2. CARD EQUAL WIDTH — ALWAYS by data-id, NEVER by CSS class

**⚠️ CONFIRMED BUG:** `_css_classes` set on `elType: "column"` elements inside e-con containers are silently dropped by Elementor at render time. The column has no custom class in the DOM. CSS targeting `.xpo-card-col` will NEVER fire.

**The ONLY reliable approach:**

After rendering, fetch the HTML, extract the `data-id` of each card column, then write CSS targeting those data-ids explicitly:

```css
/* Card equal width — by data-id (the only method that works) */
.elementor-element-{col1_id},
.elementor-element-{col2_id},
.elementor-element-{col3_id} {
    flex: 1 1 0 !important;
    width: 0 !important;
    min-width: 200px !important;
}
.elementor-element-{col1_id} > .elementor-widget-wrap,
.elementor-element-{col2_id} > .elementor-widget-wrap,
.elementor-element-{col3_id} > .elementor-widget-wrap {
    height: 100% !important;
    display: flex !important;
    flex-direction: column !important;
    padding: 28px 24px !important;
    box-shadow: 0 2px 16px rgba(0,0,0,0.08) !important;
    border-radius: 12px !important;
    background: {CARD_BG} !important;
    border: 1px solid {BORDER} !important;
}
```

**How to get the data-ids:** After pushing JSON and clearing cache, run:
```python
import re
html = # fetch rendered page HTML
col_ids = re.findall(r'<div[^>]*elementor-col-3[34][^>]*data-id="([^"]+)"', html)
print(col_ids)  # → ['268ovahe', 'lrjob16w', 'lydp0fdb']
```

**Never:** assign `css_class` to columns expecting it to appear in the DOM for CSS targeting. It won't.
**Never:** set different `pct` values (33/33/34) and expect equal visual width.

---

### 3. HERO FULL HEIGHT — Always explicit

Hero sections must always set `min_height` in the `section()` call:
```python
hero_section = section(
    [...],
    bg=BLUE,
    min_height=90,   # ALWAYS — unit is vh. Never omit.
    ...
)
```

If the hero looks like a short box without visual breathing room, `min_height` was omitted.

---

### 4. CARD BACKGROUNDS + BORDER RADIUS — Always in col() JSON, never in CSS

This is in the reference file and must be repeated here because it keeps being violated:

```python
# CORRECT — background and border-radius go in col() settings
col(widgets, bg="#ffffff", border_color="#e5e7eb", border_radius=12)

# WRONG — never set card background or border-radius via CSS
# .xpo-card { background: #fff; border-radius: 12px; }  ← Elementor inline styles override this
```

The `col()` function must support `bg`, `border_color`, `border_radius`, and `border_width` as JSON settings. These are rendered as Elementor widget properties which cannot be overridden by Elementor's own inline styles.

---

### 5. CARD GAP CONTROL — Triple-Redundancy Approach

Card gaps fail because they're controlled by the PARENT container, not the cards. Use this triple-redundancy:

**Step 1: Measure in Content Map**
```
## CARD LAYOUT — CONTAINER
- Card row gap: 24px  ← MEASURE THIS FROM REFERENCE
```

**Step 2: Set in JSON**
```python
container(
    [card1, card2, card3],
    direction="row",
    gap=24  # ← FROM CONTENT MAP
)
```

**Step 3: Extract container data-id and enforce in CSS**
```css
.elementor-element-{CONTAINER_ID} {
    gap: 24px !important;  ← MATCH CONTENT MAP
}
.elementor-element-{CONTAINER_ID} > .e-con-inner {
    gap: 24px !important;
}
```

**Why triple:** If JSON gap fails, CSS catches it. If CSS specificity loses, the JSON provides baseline.

---

### 6. HERO HEIGHT ISSUE — "hero is not full viewport height"

This is always caused by missing `min_height`. Fix:
```python
section(..., min_height=90)  # 90vh
```
Additionally add to CSS:
```css
.{slug}-hero { min-height: 90vh !important; display: flex !important; align-items: center !important; }
.{slug}-hero > .elementor-container { width: 100% !important; }
```

---

## Deployment Flow

1. **Step 0** — Content Map (MEASURE EVERYTHING — no "from reference")
2. **Step 0.5** — Element Declaration (client may waive approval, but declaration must still be written)
3. Build full Elementor JSON in Python
4. Push via REST API — always include `"template": "elementor_header_footer"`
5. Edit `functions.php` via **SSH (paramiko) only** — all CSS goes here — **MANDATORY, page is broken without this**
   - **Kit custom_css field is BANNED** — CSS injected via the Elementor Kit does not reliably render on the frontend. Do not use it as a fallback. If SSH fails, stop and report the error — do not silently fall back to Kit CSS.
6. Clear Elementor cache: `DELETE /wp-json/elementor/v1/cache`
7. **Run post-build extractor** → `post-build-extractor.md` — MANDATORY, no exceptions
   - Fetches rendered HTML, extracts all data-ids, outputs ready-to-paste CSS block
   - Do NOT write card/hero/button CSS until extractor has run and returned real data-ids
8. Write CSS in order: data-id blocks first (from extractor), then class-based blocks (labels, typography, nav)
9. Verify: `?nocache=<timestamp>`
10. Report: "Built. Live at [URL]?nocache=[timestamp]. Sections: [list]." — stop. Client reviews.

**No visual self-check after deploy.** Build, push, report URL, stop. Client reviews.

---

## Step 0 — Content Map Template (COMPLETE BRAILLE)

**INSTRUCTIONS:**
1. Create ONE block per section (Hero, Who We Are, Services, etc.)
2. **EVERY field must be filled** — no "from reference", no guessing
3. Measure pixels from screenshot using any tool
4. Hex colors only — no "blue" or "dark gray"
5. If field doesn't apply, write "N/A"
6. **CARD GAP IS MANDATORY** — measure the space between card edges

```
# ═══════════════════════════════════════════════════════════════
# SECTION: [NAME — e.g., HERO]
# ═══════════════════════════════════════════════════════════════

## SECTION-LEVEL COLORS
- Section background: #[HEX] (solid color, measure from screenshot)
- Section background image: YES/NO (if YES, document: "using solid #HEX match")
- Section min-height: [X]vh (measure from reference)

## TYPOGRAPHY — LABEL (small text above headline)
- Label text: "[EXACT TEXT]"
- Label color: #[HEX]
- Label font-size: [X]px
- Label font-weight: [100-900]
- Label uppercase: YES/NO
- Label letter-spacing: [X]em (typically 0.1-0.15)
- Label CSS class: [slug]-label-[color]

## TYPOGRAPHY — HEADLINE (H1/H2)
- Headline text: "[EXACT TEXT]"
- Headline has multi-color: YES/NO
- If YES, colored portion: "[TEXT]" color: #[HEX]
- Headline color (primary): #[HEX]
- Headline color (accent): #[HEX] (if multi-color)
- Headline font-size: [X]px (desktop)
- Headline font-weight: [100-900]
- Headline line-height: [X] (unitless, e.g., 1.2)
- Headline max-width: [X]px (controls line breaks)
- Headline alignment: center/left/right
- Headline CSS class: [slug]-[section]-h1

## TYPOGRAPHY — SUBHEADLINE
- Subheadline text: "[EXACT TEXT]"
- Subheadline color: #[HEX]
- Subheadline font-size: [X]px
- Subheadline font-weight: [100-900]
- Subheadline line-height: [X]
- Subheadline max-width: [X]px
- Subheadline alignment: center/left/right
- Subheadline CSS class: [slug]-[section]-sub

## TYPOGRAPHY — BODY (if applicable)
- Body text: "[EXACT TEXT — all paragraphs]"
- Body color: #[HEX]
- Body font-size: [X]px
- Body font-weight: [100-900]
- Body line-height: [X]
- Body CSS class: [slug]-[section]-body

## SPACING — SECTION
- Section padding-top: [X]px
- Section padding-bottom: [X]px
- Section padding-left/right: [X]px (if asymmetric)

## SPACING — CONTENT STACK (vertical gaps)
- Label to headline: [X]px
- Headline to subheadline: [X]px
- Subheadline to buttons: [X]px
- Button row gap: [X]px (between buttons)

## BUTTONS
### Button 1:
- Text: "[EXACT]"
- Style: solid/outline
- Background: #[HEX] (if solid)
- Text color: #[HEX]
- Border color: #[HEX] (if outline)
- Border width: [X]px
- Border radius: [X]px
- Font size: [X]px
- Font weight: [100-900]
- CSS class: [slug]-btn-[type]

### Button 2:
- Text: "[EXACT]"
- (same fields as Button 1)

## LAYOUT — CONTENT STACK
- Number of elements stacked vertically: [N]
- Content alignment: center/left/right
- Content max-width: [X]px (container width)

## IMAGES
### Image 1:
- Present: YES/NO
- Position: left/right/top/background
- Dimensions: [W]x[H]px
- Placehold.co URL: https://placehold.co/[W]x[H]/[BG]/[TEXT]?text=[DESC]
- Border radius: [X]px

## SPECIAL ELEMENTS
- Decorative shapes: YES/NO (describe)
- Background patterns: YES/NO (describe)
- Animations: YES/NO (describe)

# ═══════════════════════════════════════════════════════════════
```

---

## CARD SECTIONS (Services, Features, etc.) — ADD TO TEMPLATE

```
## CARD LAYOUT — CONTAINER (CRITICAL)
- Number of cards: [N]
- Cards per row: [N] (typically 3 or 4)
- **Card row gap: [X]px (MEASURE THIS — space between cards horizontally)**
- Card row max-width: [X]px (container width, typically 1200px)
- Card row alignment: center/left/right
- Card equal width method: flex 1 1 0 (always this)

## CARD STYLING — PER CARD (measure one, apply to all)
- Card background: #[HEX]
- Card border color: #[HEX]
- Card border width: [X]px
- Card border radius: [X]px
- Card shadow: [X]px [Y]px [BLUR]px [SPREAD]px rgba(R,G,B,A)
- Card padding: [X]px top / [X]px right / [X]px bottom / [X]px left

## CARD TYPOGRAPHY — TITLE
- Title color: #[HEX]
- Title font-size: [X]px
- Title font-weight: [100-900]
- Title CSS class: [slug]-card-title

## CARD TYPOGRAPHY — BODY
- Body color: #[HEX]
- Body font-size: [X]px
- Body font-weight: [100-900]
- Body line-height: [X]
- Body CSS class: [slug]-card-body

## CARD LINKS
- Link text: "[EXACT — e.g., Learn More →]"
- Link color: #[HEX]
- Link font-size: [X]px
- Link font-weight: [100-900]
- Link CSS class: [slug]-card-link

### Card 1:
- Title: "[EXACT]"
- Body: "[EXACT — full text]"
- Link: "[EXACT]"

### Card 2:
- Title: "[EXACT]"
- Body: "[EXACT — full text]"
- Link: "[EXACT]"

### Card 3:
- Title: "[EXACT]"
- Body: "[EXACT — full text]"
- Link: "[EXACT]"
```

**If it's not in your Step 0 notes, you don't know it.**

**Typography is a hard blocker.** `FROM_REF` in a typography field means stop — you do not have that value yet. Do not write any code with a guessed or defaulted font size, weight, or color.

**Section label style must be explicitly identified.** Is it a pill with background + border? Plain colored text? Underlined? Nothing? Never assume it's a bubble/pill just because another site used one.

**Multi-color headlines — ONE widget, inline span. This is MANDATORY.**
```python
heading('Compassionate Rehabilitation Services in <span style="color:#cb9c23;">Riverside, CA</span>', tag="h1", css_class="...")
```
**NEVER** use two separate heading widgets for a two-color headline. One heading widget, one inline `<span style="color:HEX">`. That's it. Two widgets causes alignment, spacing, and sizing problems every time.

---

## Step 0.5 — Element Declaration (COMPLETE STRUCTURE)

**INSTRUCTIONS:**
1. Declare EVERY element that will exist on the page
2. Use exact text from Content Map
3. Specify CSS class for every element
4. Hierarchy must match JSON structure
5. NO element can exist in JSON that isn't declared here

```
# ═══════════════════════════════════════════════════════════════
# PAGE: [Page Name]
# SLUG: [slug]
# PAGE ID: [ID if editing, or "NEW"]
# ═══════════════════════════════════════════════════════════════

## GLOBAL SETTINGS
- Font family: Inter (Google Fonts)
- Container max-width: 1200px
- Base font size: 16px

## SECTION 1: [NAME — e.g., HERO]

### Section Properties:
- elType: section
- Background: #[HEX] (from Content Map)
- Min-height: [X]vh
- CSS class: [slug]-hero

### Container (e-con flex):
- Direction: column
- Align items: center
- Justify content: center
- Max width: [X]px
- Gap: [X]px (vertical gap between elements)
- CSS class: [slug]-hero-container

### Element 1: Label
- Widget: heading
- Tag: p
- Text: "[EXACT FROM CONTENT MAP]"
- CSS class: [slug]-label-[color]

### Element 2: Headline
- Widget: heading
- Tag: h1
- Text: "[EXACT] <span style='color:#[HEX];'>[COLORED PORTION]</span>"
- CSS class: [slug]-hero-h1

### Element 3: Subheadline
- Widget: heading
- Tag: p
- Text: "[EXACT]"
- CSS class: [slug]-hero-sub

### Element 4: Button Row (e-con flex):
- Direction: row
- Gap: [X]px
- Justify: center

#### Button 1:
- Widget: button
- Text: "[EXACT]"
- Style: outline
- CSS class: [slug]-btn-outline

#### Button 2:
- Widget: button
- Text: "[EXACT]"
- Style: outline
- CSS class: [slug]-btn-outline

---

## SECTION 3: [NAME — e.g., SERVICES] (Card Grid)

### Section Properties:
- elType: section
- Background: #[HEX]
- Padding: [X]px top / [X]px bottom
- CSS class: [slug]-services

### Header Stack (e-con flex column):
- Align: center
- Gap: [X]px

#### Element 1: Label
- Widget: heading
- Tag: p
- Text: "[EXACT]"
- CSS class: [slug]-label-[color]

#### Element 2: Headline
- Widget: heading
- Tag: h2
- Text: "[EXACT]"
- CSS class: [slug]-section-h2

#### Element 3: Subheadline
- Widget: heading
- Tag: p
- Text: "[EXACT]"
- CSS class: [slug]-section-sub

### Card Row (e-con flex row):
- Direction: row
- **Gap: [X]px (THIS IS THE CARD GAP FROM CONTENT MAP)**
- Justify: center
- Max width: [X]px

#### Card 1 Column:
- elType: column
- Width: 33.333%
- Background: #[HEX] (JSON)
- Border: [X]px solid #[HEX] (JSON)
- Border radius: [X]px (JSON)
- Padding: [X]px (JSON)
- CSS class: [slug]-card-col (for tracking)
- Data-id target: YES (for CSS extraction)

##### Card 1 Content Stack:
- Direction: column
- Gap: [X]px

###### Element 1: Title
- Widget: heading
- Tag: h3
- Text: "[EXACT]"
- CSS class: [slug]-card-title

###### Element 2: Body
- Widget: heading
- Tag: p
- Text: "[EXACT]"
- CSS class: [slug]-card-body

###### Element 3: Link
- Widget: heading (styled as link)
- Tag: p
- Text: "[EXACT]"
- CSS class: [slug]-card-link

#### Card 2 Column: [mirror Card 1 structure]
#### Card 3 Column: [mirror Card 1 structure]
```

**Self-review before showing client:**
- [ ] Every section declared — none missing
- [ ] Every card declared individually — no "same as above"
- [ ] Every text element with exact copy
- [ ] Every button with exact text and style
- [ ] Every color references Step 0 Content Map
- [ ] Every section label style confirmed from reference (not assumed)
- [ ] Every label widget has its own color-specific class — no shared label classes
- [ ] Multi-color headline plan confirmed if needed
- [ ] CSS class prefix set for this build — no `ss-*` or classes from another project
- [ ] All card columns use shared `{slug}-card-col` class for equal-width enforcement
- [ ] Card row gap explicitly declared (from Content Map)
- [ ] Nav present? → REMOVE IT unless client explicitly unlocked it

---

## Authentication

```python
import subprocess

# Login — save cookies
subprocess.run([
    'curl', '-s', '-c', '/tmp/wp_cookies.txt', '-b', '/tmp/wp_cookies.txt',
    '-X', 'POST',
    '--data-urlencode', 'log=USERNAME',
    '--data-urlencode', 'pwd=PASSWORD',
    '--data-urlencode', 'wp-submit=Log In',
    '--data-urlencode', 'redirect_to=/wp-admin/',
    '--data-urlencode', 'testcookie=1',
    '-H', 'Cookie: wordpress_test_cookie=WP+Cookie+check',
    '-L', 'https://SITE/wp-login.php'
], capture_output=True, text=True)

# Get REST nonce
nonce = subprocess.run([
    'curl', '-s', '-b', '/tmp/wp_cookies.txt',
    'https://SITE/wp-admin/admin-ajax.php?action=rest-nonce'
], capture_output=True, text=True).stdout.strip().strip('"')
```

**Never Application Passwords. Never bash heredoc. Always Python.**

**Two auth methods — know which to use:**
| Task | Method |
|------|--------|
| Push Elementor JSON (REST API) | Cookie auth + nonce (above) |
| Edit functions.php | SSH via paramiko (see functions.php Pattern) |

SSH is always preferred for file edits — bypasses Imunify360, no nonces, no cookie dance.

---

## Page Creation

```python
page_data = {
    "title": "Page Title",
    "status": "publish",
    "template": "elementor_header_footer",  # ALWAYS — never omit
    "meta": {
        "_elementor_edit_mode": "builder",
        "_elementor_data": json.dumps(sections)
    }
}
```

---

## Edit Process

No partial update endpoint exists. Every edit:

1. `GET /wp-json/wp/v2/pages/{id}?context=edit`
2. Modify in Python
3. Push full JSON back
4. Clear cache

---

## Push JSON

```python
import json, subprocess

result = subprocess.run([
    'curl', '-s', '-X', 'POST',
    '-b', '/tmp/wp_cookies.txt',
    '-H', 'Content-Type: application/json',
    '-H', f'X-WP-Nonce: {nonce}',
    '--data', json.dumps({"meta": {"_elementor_data": json.dumps(sections)}}),
    f'{WP_URL}/wp-json/wp/v2/pages/{PAGE_ID}',
], capture_output=True, text=True)
print(result.stdout[:300])
```

---

## Clear Cache

```python
subprocess.run([
    'curl', '-s', '-X', 'DELETE',
    '-b', '/tmp/wp_cookies.txt',
    '-H', f'X-WP-Nonce: {nonce}',
    f'{WP_URL}/wp-json/elementor/v1/cache',
], capture_output=True, text=True)
```

Breeze (Cloudways CDN) can't be cleared via API. Always verify with `?nocache=<timestamp>`.

---

## Widget Builders

Use these for every build. Never raw JSON. Never `text-editor` widget — always `heading`.

```python
import random, string

def uid():
    return ''.join(random.choices(string.ascii_lowercase + string.digits, k=8))

def heading(text, tag="h2", align="left", size=None, weight=None,
            color=None, css_class="", line_height=None):
    s = {"title": text, "header_size": tag, "align": align}
    if size:        s["typography_font_size"] = {"unit": "px", "size": size}
    if weight:      s["typography_font_weight"] = str(weight)
    if color:       s["title_color"] = color
    if line_height: s["typography_line_height"] = {"unit": "em", "size": line_height}
    if css_class:   s["_css_classes"] = css_class
    return {"id": uid(), "elType": "widget", "widgetType": "heading", "settings": s, "elements": []}

def button(text, url="#", bg="#ffffff", color="#000000", border_color="transparent",
           border_style="none", css_class=""):
    s = {
        "text": text, "link": {"url": url}, "button_type": "custom",
        "background_color": bg, "button_text_color": color,
        "border_border": border_style,
        "border_width": {"unit": "px", "top": "2", "right": "2", "bottom": "2", "left": "2"},
        "border_color": border_color,
        "border_radius": {"unit": "px", "top": "8", "right": "8", "bottom": "8", "left": "8"},
        "typography_font_size": {"unit": "px", "size": 16},
        "typography_font_weight": "500",
        "padding": {"unit": "px", "top": "14", "right": "28", "bottom": "14", "left": "28", "isLinked": False},
    }
    if css_class: s["_css_classes"] = css_class
    return {"id": uid(), "elType": "widget", "widgetType": "button", "settings": s, "elements": []}

def image(url, alt=""):
    s = {"image": {"url": url, "alt": alt}, "image_size": "full"}
    return {"id": uid(), "elType": "widget", "widgetType": "image", "settings": s, "elements": []}

def col(elements, pct=100, padding=None, bg=None, align=None,
        border_color=None, border_radius=None, border_width=1, css_class=""):
    s = {
        "_column_size": pct,
        "content_position": align or "top",
    }
    if padding: s["padding"] = padding
    if bg:
        s["background_background"] = "classic"
        s["background_color"] = bg
    if border_color:
        s["border_border"] = "solid"
        s["border_width"] = {"unit": "px", "top": str(border_width), "right": str(border_width), "bottom": str(border_width), "left": str(border_width)}
        s["border_color"] = border_color
    if border_radius:
        s["border_radius"] = {"unit": "px", "top": str(border_radius), "right": str(border_radius), "bottom": str(border_radius), "left": str(border_radius)}
    if css_class: s["_css_classes"] = css_class
    return {"id": uid(), "elType": "column", "settings": s, "elements": elements}

def section(cols, bg="#ffffff", padding=None, min_height=None, css_class=""):
    s = {
        "background_background": "classic",
        "background_color": bg,
        "layout": "full_width",
    }
    if padding:    s["padding"] = padding
    if min_height: s["min_height"] = {"unit": "vh", "size": min_height}
    if css_class:  s["_css_classes"] = css_class
    return {"id": uid(), "elType": "section", "settings": s, "elements": cols}
```

---

## CSS Quick Rules

**These apply to every build. Copy them verbatim. Never skip.**

```css
/* 1. Global reset — Hello Elementor defaults */
.elementor-element-populated {
    border: none !important;
    border-radius: 0 !important;
    overflow: visible !important;
    transition: none !important;
}

/* 2. Section container centering */
.elementor-section { width: 100% !important; }
.elementor-section > .elementor-container {
    max-width: 1200px !important;
    margin: 0 auto !important;
    width: 100% !important;
}

/* 3. Page title always hidden */
.entry-title, .page-title { display: none !important; }

/* 4. Admin bar hidden */
#wpadminbar { display: none !important; }
html { margin-top: 0 !important; }
body.logged-in .admin-bar { display: none !important; }

/* 5. Font */
body { font-family: 'Inter', sans-serif !important; }
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');

/* 6. Card column equal width — applies when all card cols share {slug}-card-col class */
.{slug}-card-col {
    flex: 1 1 0 !important;
    width: 0 !important;
    min-width: 200px !important;
}
.{slug}-card-col > .elementor-element-populated {
    height: 100% !important;
    display: flex !important;
    flex-direction: column !important;
}

/* 7. Hero full height */
.{slug}-hero {
    min-height: 90vh !important;
    display: flex !important;
    align-items: center !important;
}
.{slug}-hero > .elementor-container { width: 100% !important; }
```

**BANNED CSS — never write these:**
- `overflow: hidden` on any elementor wrapper
- `body { overflow-x: hidden }`
- `.elementor-column { width: 100% }`
- `.e-con.e-flex { justify-content: center }` (global — targets all containers)
- Any card background or border-radius in CSS (goes in col() JSON)

---

## functions.php Pattern (SSH)

Always use paramiko SSH — never cookie auth for file edits.

```python
import paramiko, base64, re

def read_functions_php():
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    client.connect(SSH_HOST, port=22, username=SSH_USER, password=SSH_PASS, timeout=15)
    _, stdout, _ = client.exec_command(f'cat "{FUNCTIONS_PATH}"')
    content = stdout.read().decode('utf-8')
    client.close()
    return content

def write_functions_php(content):
    encoded = base64.b64encode(content.encode('utf-8')).decode('ascii')
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    client.connect(SSH_HOST, port=22, username=SSH_USER, password=SSH_PASS, timeout=15)
    _, stdout, stderr = client.exec_command(f'echo "{encoded}" | base64 -d > "{FUNCTIONS_PATH}"')
    err = stderr.read().decode()
    client.close()
    return err

# Pattern: strip old build blocks, append new ones
BLOCK_MARKERS = ["// ── {SLUG} STYLES", "// ── {SLUG} NAV"]

def update_functions_php(nav_html, css_block):
    current = read_functions_php()
    # Strip existing blocks for this slug
    for marker in BLOCK_MARKERS:
        current = re.sub(rf'\n*{re.escape(marker)}.*', '', current, flags=re.DOTALL).rstrip()
    # Append new blocks
    new_content = current + "\n\n" + nav_html + "\n\n" + css_block
    write_functions_php(new_content)
```

**Block structure in functions.php:**
```php
// ── {SLUG} NAV ──────────────────────────────────────────────────────────────
add_action('wp_body_open', function() { ?>
<nav id="{slug}-nav" class="{slug}-nav">
  ...
</nav>
<script>/* scroll behavior */</script>
<?php });

// ── {SLUG} STYLES ────────────────────────────────────────────────────────────
add_action('wp_head', function() { ?>
<style>
/* all CSS here */
</style>
<?php });
```

---

## Diagnostic Protocol

When something looks wrong on the live page:

1. **Fetch rendered HTML first** — `curl -s '{URL}?nocache={ts}'`
2. **Check if the CSS class is actually in the HTML** — `grep 'xpo-label-blue'`
3. **Check if functions.php was written** — SSH `cat functions.php | grep -c 'xpo-'`
4. **Check if Elementor cache was cleared** — re-run DELETE cache endpoint
5. **Never assume** — always verify with curl before writing a fix

---

## Verified Patterns — Use These, Don't Reinvent

### Nav transparent-to-white on scroll
```javascript
var nav = document.getElementById('{slug}-nav');
window.addEventListener('scroll', function(){
    nav.classList.toggle('{slug}-nav--scrolled', window.scrollY > 80);
}, {passive: true});
```

### 3-column card grid (equal width, correct)
```python
cards = [
    col(card1_widgets, bg=CARD_BG, border_color=BORDER, border_radius=12, css_class="{slug}-card-col"),
    col(card2_widgets, bg=CARD_BG, border_color=BORDER, border_radius=12, css_class="{slug}-card-col"),
    col(card3_widgets, bg=CARD_BG, border_color=BORDER, border_radius=12, css_class="{slug}-card-col"),
]
card_row = section(cards, bg=SECTION_BG)
```
Then in CSS:
```css
.{slug}-card-col { flex: 1 1 0 !important; width: 0 !important; min-width: 200px !important; }
```

### Hero section (always full height)
```python
hero = section([col([...], pct=100)], bg=HERO_BG, min_height=90, css_class="{slug}-hero")
```
```css
.{slug}-hero { min-height: 90vh !important; display: flex !important; align-items: center !important; }
```

---

## Version History

- v3.5 (March 20, 2026): Complete Braille — crystal clear Content Map & Element Declaration, card gap triple-redundancy
- v3.4 (March 19, 2026): Nav/footer exclusions, image rule, widget ban, Kit CSS ban
- v3.3 (March 19, 2026): Data-id extraction mandatory
- v3.0 (March 19, 2026): Initial split

---

*End of file. Read completely before every build.*

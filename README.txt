================================================================================
  BASE44 → WORDPRESS ELEMENTOR PIPELINE
================================================================================

PURPOSE:
--------
Take a Base44 designed website and convert it to a live WordPress Elementor page.

This pipeline handles:
- Step 0: Content mapping (extracting colors, typography, measurements from reference)
- Step 0.5: Element declaration (planning the widget structure)
- JSON building (Elementor page structure via Python)
- REST API push to WordPress
- Data-id extraction for CSS targeting
- CSS injection via SSH to functions.php

================================================================================
  CREDENTIALS
================================================================================

WORDPRESS (SolidState Studio):
------------------------------
URL:      https://wordpress-1586104-6245423.cloudwaysapps.com
Username: spencer.naismith2@gmail.com
Password: 8u5xz8RGpZ

CLOUDWAYS (Server Access):
--------------------------
URL:      https://platform.cloudways.com
Username: spencer.naismith2@gmail.com
Password: 5366247Jsn##

================================================================================
  SKILL FILES
================================================================================

1. wordpress-pipeline.md
   ---------------------
   Start here. Quick reference for which file to use when.

2. wordpress-elementor.md
   -----------------------
   The main execution file. Step-by-step checklist for building Elementor pages.
   Use this during every build — re-read before each major step.

3. wordpress-elementor-reference.md
   ---------------------------------
   Deep rules and explanations. Why things work the way they do.
   Read once at session start, then reference as needed.

4. post-build-extractor.md
   ------------------------
   Script for extracting data-ids from rendered HTML.
   Run this AFTER pushing JSON, BEFORE writing CSS.
   Critical — column classes don't work, data-ids do.

================================================================================
  WORKFLOW SUMMARY
================================================================================

1. Read wordpress-elementor-reference.md (absorb the rules)
2. Read wordpress-elementor.md (execution checklist)
3. Step 0: Content Map — document every color, font, text, image
4. Step 0.5: Element Declaration — plan every widget and section
5. Build JSON using the widget builders (heading(), button(), col(), section())
6. Push to WordPress via REST API with template: elementor_header_footer
7. Clear Elementor cache
8. Run post-build-extractor.py → get data-ids
9. Write CSS targeting .elementor-element-{data-id}
10. Update functions.php via SSH with nav + CSS
11. Report URL, stop, wait for visual review

================================================================================
  CRITICAL RULES
================================================================================

- NO ICONS EVER — skip icon widgets entirely
- Data-id targeting for structural CSS (columns, hero height, card widths)
- Class-based targeting for typography only
- Card styling in JSON (bg, border, border-radius) — CSS gets overridden
- Never work from memory — re-read the skill section before executing

================================================================================

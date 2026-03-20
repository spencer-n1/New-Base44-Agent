---
type: procedural
force_checklist: true
---

# WordPress Elementor Build Pipeline
**Version:** 3.0 — March 19, 2026
**Status:** Battle-tested, ready for autonomous builds

## ⚠️ THIS FILE HAS BEEN SPLIT

The pipeline is now two files. Use them like this:

| File | When to Use |
|------|-------------|
| `wordpress-elementor.md` | **Every build, mid-build re-reads** — checklists, code snippets, deployment flow, mistakes table |
| `wordpress-elementor-reference.md` | **Session start, diagnosing bugs** — deep explanations, CSS deep dives, ReactBits pipeline, typography/color extraction guides |

### Session Start

Read both files in this order:
1. `wordpress-elementor-reference.md` — absorb the deep rules once
2. `wordpress-elementor.md` — internalize the execution flow

### Mid-Build

Re-read only `wordpress-elementor.md`. It has a Re-Read Map at the top — find the section you need in under 10 seconds.

### Updating the Pipeline

- New rule from a build → add to mistakes table in `wordpress-elementor.md`
- Deep explanation of why → add to `wordpress-elementor-reference.md`
- Never hardcode build-specific values (hex, px, class names) in either file
- Always read the full file before editing — never rebuild from memory

---

*This index file is intentionally minimal. All content lives in the two files above.*
*Last updated: March 19, 2026*

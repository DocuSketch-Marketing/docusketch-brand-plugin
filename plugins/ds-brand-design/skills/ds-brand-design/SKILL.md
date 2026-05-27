---
name: ds-brand-design
description: DocuSketch brand design system — the canonical source of truth for brand, marketing, and visual design decisions. Use whenever working on anything visual for DocuSketch (websites, marketing collateral, brand assets, Figma plugins, design tokens, typography, color, spacing, components). Covers approved color tokens, typography styles, spacing scale, radius scale, component IDs, accessibility rules, and Figma file references. Also use when reviewing or producing design code that must match brand standards.
---

# DocuSketch Brand Design System

## Where the canonical content lives

The full design system reference is bundled with this skill as `DS_Brand_Design_Skill.md`, sitting in this skill's own directory (next to this SKILL.md).

That file is the single source of truth. It is generated from the upstream Figma file (`JR35zTngKUblEKMD0myUyD` — Brand Design Kit) by `figma-sync.py` and shipped inside this plugin, so it travels with the skill on every machine with no extra setup. To pull the latest brand data, update the plugin: run `/plugin marketplace update`, then `/reload-plugins`. The same content also powers the dashboard at https://brand-design-kit.vercel.app.

## How to use this skill

When this skill activates, read the bundled canonical file in full before making design decisions. It lives in this skill's base directory:

```
Read DS_Brand_Design_Skill.md
```

(Resolve it from the same directory as this SKILL.md.) Apply its rules verbatim — color tokens, typography, spacing scale (4px grid, prefer 8px multiples), radius scale (2 / 4 / 8 / 100 only), Figma component IDs, accessibility (WCAG AA), and the "Best Practices" checklist at the top of the file.

## Relationship to other design systems

The DocuSketch product team maintains a separate, deeper system (Phoenix) derived from this brand kit. **Phoenix informs but does not control this brand system.** Do not pull Phoenix tokens, components, or rules into brand work unless explicitly asked. If a brand decision needs to reconcile with Phoenix, ask before adopting Phoenix conventions.

## When NOT to use this skill

- Generic web/code work unrelated to DocuSketch brand
- Backend logic, infra, or non-visual code
- Product UI work where Phoenix is the source of truth (those projects will have their own skill)

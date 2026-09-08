---
name: docusketch-brand
description: DocuSketch Marketing Design System — the canonical source of truth for brand, marketing, and visual design decisions. Use when creating or reviewing ANY DocuSketch-branded asset (PDFs, Word docs, presentations, web/HTML, Figma plugin scripts, dashboards, marketing collateral, print pieces). Covers the immovable Best Practices checklist, logo usage and clear space, colour primitives and semantic tokens with Pantone and WCAG data, the 2026 naming unification, IBM Plex type styles and the plain-zero rule, radius and two-tier spacing scales, shadows, motion canon, Dark Mode pairings, Brand Expressions (Editorial mode and the print type ramp), Material Symbols iconography, and compound-component anatomy. Also use when asked for something "on brand", "in DocuSketch's brand", or when checking design code against brand standards.
metadata:
  author: chris.provins
  installer: nick.keyko
  version: '3.0'
  source: https://github.com/DocuSketch-Marketing/docusketch-brand-plugin
  canon_commit: 156dc46
  figma_file: JR35zTngKUblEKMD0myUyD
  upstream_figma_sync: '2026-08-21'
  synced: '2026-08-26'
---

# DocuSketch Brand Design System

## Where the canonical content lives

The complete design system is bundled with this skill as `DS_Brand_Design_Skill.md`, in this skill's own directory next to this file.

**That file is the source of truth. This file is only a pointer to it.**

It is generated from the Brand Design Kit Figma file (`JR35zTngKUblEKMD0myUyD`) by `figma-sync.py` and published to
[DocuSketch-Marketing/docusketch-brand-plugin](https://github.com/DocuSketch-Marketing/docusketch-brand-plugin).
The same content powers the dashboard at https://brand-design-kit.vercel.app.

This skill is a **mirror** of that repository, kept so the brand system is available to people who don't work in a terminal. It must never be edited in place — see "Keeping this skill current" below.

## How to use this skill

When this skill activates, read the bundled canonical file **in full** before making any design decision:

```
Read DS_Brand_Design_Skill.md
```

Resolve it from the same directory as this SKILL.md.

Apply its rules verbatim. Deliberately, this file restates none of them — every colour token, type style, scale value, and component rule lives in the canonical file only, so there is exactly one place to be right. Start with the **Best Practices** checklist at the top of that file; those are marked immovable and several of them (the logo SVG-only rule, the plain-zero rule) are the brand defects that are hardest to retract once shipped.

Pay particular attention to two sections that override things stated elsewhere in the same file:

- **Naming Convention & Source of Truth — 2026 Unification** declares that it supersedes the token tables that follow it. Where it and a later table disagree, the unification section wins.
- **Dark Mode** carries the committed light/dark pairing for every semantic token. Never invert a light value by hand; use the pairing table.

> **Known discrepancy (as of 2026-08-26).** The unification section and the Dark Mode / Tokens · Text sections assign `text-primary` and `text-strong` to opposite values (`#39381B` vs `#1A1905`). The unification section is newer and self-declares supersession, so prefer it — but flag the conflict rather than guessing if the choice is load-bearing. Raised with the design owner; remove this note once the section-by-section rewrite lands.

## Relationship to other design systems

The DocuSketch product team maintains a separate, deeper system (Phoenix) derived from this brand kit. **Phoenix informs but does not control this brand system.** Do not pull Phoenix tokens, components, or rules into brand work unless explicitly asked. If a brand decision needs to reconcile with Phoenix, ask before adopting Phoenix conventions.

## Keeping this skill current

This is a mirror, not an original. To update it, replace `DS_Brand_Design_Skill.md` with the current copy from the repository and re-upload the skill — do not hand-edit either file. Hand-editing is what caused this mirror to drift three months behind upstream once already.

Current contents track commit `156dc46`, Figma sync `2026-08-21`.

## When NOT to use this skill

- Generic web or code work unrelated to DocuSketch brand
- DocuSketch **product** UI governed by the Phoenix design system
- Anything where the user has specified their own visual direction

---
name: docusketch-brand
description: DocuSketch Marketing Design System — the canonical source of truth for brand, marketing, and visual design decisions. Use when creating or reviewing ANY DocuSketch-branded asset (PDFs, Word docs, presentations, web/HTML, Figma plugin scripts, dashboards, marketing collateral, print pieces). Covers the immovable Best Practices checklist, logo usage and clear space, colour primitives and semantic tokens with Pantone and WCAG data, the 2026 naming unification, IBM Plex type styles and the plain-zero rule, radius and two-tier spacing scales, shadows, motion canon, Dark Mode pairings, Brand Expressions (Editorial mode and the print type ramp), the Product DocuSketch Portal iconography library, and compound-component anatomy. Also use when asked for something "on brand", "in DocuSketch's brand", or when checking design code against brand standards.
metadata:
  author: chris.provins
  installer: sergey.novik
  version: '4.0'
  kind: pointer
  source: https://github.com/DocuSketch-Marketing/docusketch-brand-plugin
  canon_branch: main
  figma_file: JR35zTngKUblEKMD0myUyD
  dashboard: https://brand-design-kit.vercel.app
---

# DocuSketch Brand Design System

**This skill holds no brand rules.** It fetches them, fresh, from the promoted `main` branch of the brand repository every time it activates. There is no bundled copy to fall behind; the branch is the publication.

Why: a bundled copy of this skill drifted three months behind upstream once and answered brand questions from stale values. The fix is structural. Rules live in exactly one place, and this file only knows where.

## URLs

| Name | URL |
|---|---|
| CANON | `https://raw.githubusercontent.com/DocuSketch-Marketing/docusketch-brand-plugin/main/plugins/ds-brand-design/skills/ds-brand-design/DS_Brand_Design_Skill.md` |
| STAMP | `https://raw.githubusercontent.com/DocuSketch-Marketing/docusketch-brand-plugin/main/plugins/ds-brand-design/skills/ds-brand-design/SKILL.md` |
| HUMAN | `https://brand-design-kit.vercel.app` |

## Procedure — run this before answering anything

1. **Fetch CANON.** Do this once per conversation, at first activation. Fetch again only if the user asks for the latest, or if the conversation has been open for more than a day.

2. **Fail closed.** If you have no way to fetch a URL, or the fetch errors, times out, or returns nothing, reply with exactly this and stop:

   > Brand canon unreachable — I won't guess. The rules are published at https://brand-design-kit.vercel.app.

   Do not answer any brand question from memory, from training data, or from a copy held earlier in the conversation. A confident wrong hex is worse than no answer; that is the failure this skill exists to prevent.

3. **Check it is canon.** The response must begin with the line `# DocuSketch Marketing Design System`. If it does not, you have an error page, a moved file, or a redirect. Treat it as unreachable (step 2).

4. **Fetch STAMP and read `canon_commit:` from its frontmatter.** This is non-blocking: if STAMP is unavailable, continue and say "stamp unavailable" in step 5.

5. **State provenance first, in one line, before any brand content:**

   > Brand canon: `main` @ `<canon_commit>` · Figma sync `<last_skill_sync>` · `<size>` KB

   `last_skill_sync` comes from the `## Sync Metadata` JSON block at the end of CANON. The size is the fetched byte length. If a reader questions a value, this line is how they check which rules you were holding.

6. **Apply CANON verbatim.** Every colour token, type style, scale value, and component rule lives in that document only. Deliberately, this file restates none of them.

## Reading CANON

- Start with **Best Practices** at the top. Those are marked immovable, and several (the SVG-only logo rule, the plain-zero rule, no overlapping text) are the defects hardest to retract once shipped.
- **Naming Convention & Source of Truth — 2026 Unification** supersedes any later table that disagrees with it. Where the two conflict, the unification section wins.
- **Dark Mode** carries the committed light/dark pairing for every semantic token. Never invert a light value by hand; use the pairing table.
- **Iconography** indexes every icon by name and size. Use only names from that index; never invent one. Fetch an icon's SVG from `https://brand-design-kit.vercel.app/<file>`, taking `<file>` verbatim from the manifest path the document gives. Icons are read-only: never propose adding, renaming, or altering one.

## What this skill never does

- Edit, extend, or reinterpret canon. A rule that seems wrong is reported to the brand owner (Chris Provins), not patched in the answer.
- Fall back to a remembered value when the fetch fails.
- Pull tokens, components, or rules from Phoenix, the product design system. Phoenix informs but does not control this brand system. If a brand decision needs to reconcile with Phoenix, ask before adopting Phoenix conventions.

## Keeping this skill current

Nothing to do. When the brand owner promotes `main`, every later fetch reads the new rules. Do not re-upload this skill with a bundled copy of the canonical document; that reintroduces the drift this design removes. The only reason to re-upload is if the URLs above change.

## When NOT to use this skill

- Generic web or code work unrelated to DocuSketch brand
- DocuSketch **product** UI governed by the Phoenix design system
- Anything where the user has specified their own visual direction

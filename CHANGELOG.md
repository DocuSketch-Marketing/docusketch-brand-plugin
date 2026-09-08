# Changelog

What changed in the bundled `DS_Brand_Design_Skill.md` between installs.

The bundle is generated from the canonical file by `figma-sync.py` in
`DocuSketch-Marketing/brand-design-kit`; it is never hand-edited here. To pull the
latest, run `/plugin marketplace update` then `/reload-plugins`.

Entries are newest first. `canon` is the `brand-design-kit` commit the bundle was
generated from, where known.

---

## 9efcae0 — 2026-09-08 · light is the default colour set

Publishes the **2026-09-01 immovable rule**: a surface carrying no `data-theme`
attribute renders light on every machine, on first paint, and never inherits dark
from the visitor's OS. Dark is opt-in only — `data-theme="dark"` for a committed
dark surface, `data-theme="auto"` for one that deliberately tracks
`prefers-color-scheme`.

Rewrites the Dark Mode CSS pattern and canonical-surface scoping from
`:root:not([data-theme])` to `:root[data-theme="auto"]`. **The old pattern is the
specific defect the rule exists to prevent** — anything generated against a bundle
older than this commit should be re-checked. `canon 447d9f7`

## 6f39853 — 2026-08-28 · path and radius corrections

Fixes a stale canonical-path reference and nested-radius errors in the concentric
radius guidance.

## 156dc46 — 2026-08-26 · 2026 naming unification, Brand Expressions, Editorial mode

Adds the 2026 naming unification section, which **self-declares supersession over
the token tables that follow it** — where they disagree, the unification section
wins. Adds Brand Expressions (Editorial mode, print type ramp).

Carries a known, unresolved discrepancy: the unification section and the Dark Mode
/ Tokens · Text sections assign `text-primary` and `text-strong` to opposite values
(`#39381B` vs `#1A1905`). Prefer the unification section; flag rather than guess
where the choice is load-bearing.

## 55ee4d2 — 2026-05-28 · Neutral 300 re-tuned

`#E2E0D3` → `#DFDDC8`.

## 0de69c3 — 2026-05-28 · warm-sage Text/Primary

Adds the warm on-swatch neutral overlay.

## 62365f4 — 2026-05-28 · Text/Strong, Text/Accent, canonical-surface scoping

## f52c870 — 2026-05-28 · Dark Mode section

First committed light/dark pairing table. Never invert a light value by hand — use
the pairings.

## 5dba1f5 — 2026-05-27 · initial release

DocuSketch brand design system as an installable Claude Code plugin.

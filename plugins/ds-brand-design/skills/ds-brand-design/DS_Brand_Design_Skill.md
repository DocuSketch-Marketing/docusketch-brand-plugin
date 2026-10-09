# DocuSketch Marketing Design System — Master Skill

> **Source of truth**: the Brand Design Kit Figma file (`JR35zTngKUblEKMD0myUyD`), synced into this document by `figma-sync.py`.
> This document is the canonical rendering of that system. It is versioned in the `brand-design-kit` repo and staged into the `ds-brand-design` plugin automatically — but **published only when a person promotes** `staging` to `main` in `docusketch-brand-plugin`. The claude.ai hosted skill is a separate copy and does **not** update on its own; check its `canon_commit` against the promoted `main` before trusting it.
> Live dashboard: https://brand-design-kit.vercel.app

---

## Best Practices (apply to ALL tasks)

1. **No overlapping text** — every text node must have non-overlapping bounds.
2. **Variants in component frames** — all variants live inside a COMPONENT_SET; designers can reference or edit directly.
3. **Logical, minimal, plain language naming** — use `Property=Value` Figma convention; no generic names like `Variant2`; use `Style=Chartreuse`.
4. **Accessibility, surface and ink** — light web pages sit on `Background/Default` (`Neutral 200`, `#F4F3EA`), with insets one step lighter on `Neutral 100` (`#F9F9F5`). Headings and body copy on light web pages are set in `Neutral 650` (`#39381B`, the Olive ink; 10.7:1 on the page surface). Both rulings are confirmed by the New Home page (2026-10-05); see Colour Tokens → *Web ink and surfaces*. Text must meet WCAG AA (4.5:1 normal, 3:1 large/UI).
5. **Side-by-side Light + Dark mode** — every component group shows both modes in a HORIZONTAL frame.
6. **No scaling** — components display at 1:1. Use `layoutSizingHorizontal='FIXED'` + `layoutPositioning='AUTO'` on children inside COMPONENT_SETs.
7. **`setCurrentPageAsync` required** — ALWAYS call before reading or writing any page. Pages without this return 0 children.
8. **Import + use in same script** — `importComponentSetByKeyAsync` only adds to file registry. Must import AND `createInstance()` in the same plugin call.
9. **Always use DS/Color paint styles — never raw hex fills.** Every brand-coloured node must have `node.fillStyleId` set to an imported `DS/Color/*` style. Raw hex fills (`node.fills = solid(hex)`) are **only** permitted for transparent containers (`fills = []`) and non-brand scaffold elements during layout construction — replace them before the script is finished. Use `tok()` (see Token Application Pattern below) to apply the correct style in one call.
10. **Always use brand kit components — never recreate what already exists.** Check the component table above before building anything. If a matching component exists, import it via `importComponentByKeyAsync` / `importComponentSetByKeyAsync` and call `createInstance()`. Hand-built copies of existing components are not acceptable.
11. **Radius must use the defined scale — never arbitrary values.** Nine values are permitted: `0` (`--radius-none`), `4` (`--radius-sm`), `8` (`--radius-md`), `12` (`--radius-lg`), `16` (`--radius-xl`), `24` (`--radius-2xl`), `30` (`--radius-3xl`), `32` (`--radius-4xl`), `9999` (`--radius-full`). Any other value (e.g. 1, 3, 5, 9, 25, 36) is an error. Use `--radius-full` for all pills, avatars, and circular elements — Figma auto-clamps to 50% of the smallest dimension. When nesting rounded elements, **Outer R = Inner R + Padding** — see Radius Scale.
12. **Spacing must be on the documented scale — never arbitrary values.** Two tiers:
    - **Content tier** (gaps and padding *inside* components): `4 8 12 16 20 24 32 40 48`. Mix of 4-multiples and 8-multiples for fine control of small UI.
    - **Layout tier** (section gaps, viewport-edge padding, page margins, outer chrome): `56 64 80 96 128`. Strict 8-multiples only — at this scale, finer steps don't carry meaning.

    All values are multiples of 4. Layout-tier values are all multiples of 8. Off-scale values like `6 10 14 22 36 44 60 72 100 112 144` are not permitted — choose the nearest documented value or argue to extend the scale (additions go through this rule with a dated note). See Spacing Scale section for the full per-context breakdown.
13. **Text style binding: no post-bind overrides.** After `await node.setRangeTextStyleIdAsync(0, len, styleId)`, setting ANY text property — including `node.textCase`, `node.setRangeTextCase()`, or `node.letterSpacing` — immediately clears `textStyleId` back to `""`. The rule: **bake all text properties into the style itself** (e.g. set `textCase = 'UPPER'` on the style object), then bind the node and touch only non-text node properties (`opacity`, `x`, `y`, `visible`) afterwards. `node.fillStyleId` is safe; it is not a text property.
14. **Load fonts explicitly before text style binding.** `await figma.loadFontAsync({ family: 'IBM Plex Mono', style: 'Regular' })` must be called with the exact family/style of the *target style*, not the node's current font. If they differ, the binding may silently fail.
15. **Logo / wordmark — canonical SVG only, uniform scaling only. Never typeset, never stretch, squash, or skew.** The DocuSketch wordmark must always be rendered from the canonical SVG component (Figma file `JR35zTngKUblEKMD0myUyD`, `Logo / Wordmark` set `266:1521`) at its native aspect ratio (viewBox `0 0 200 25` → **8:1**). The same applies to the DS1 logo, DS° mark, the Pill, and all partner logos in `Logo / *` sets. **SVG-only rule:** anywhere a logo or wordmark is *displayed as a brand mark* — site headers, sidebars, footers, OG cards, login screens, email signatures, watermarks — it must be the inlined or `<img>`-referenced SVG asset. **Never** typeset "DocuSketch°" / "DocuSketch" as live text using a font and a `<sup>°</sup>` (or `°` glyph) as a stand-in for the mark. Type substitution silently breaks: the wordmark's letterforms are custom-drawn (not Plex Sans), the degree symbol's optical placement is bespoke, and live text is at the mercy of the surrounding font stack, weight, tracking, and rendering hints. **Carve-out:** the brand name "DocuSketch" *may* appear as live text when written inside running prose — body copy, headings, button labels, navigation items, page titles, alt text, and ARIA labels. The test: is this *the* logo placement on the surface (one canonical mark per view), or is this the word "DocuSketch" appearing inside a sentence? Logo → SVG. Sentence → live text. **How to scale:** never set both `width` and `height` independently. Pick one (typically height) and derive the other from the aspect ratio — in code, expose a single `height` prop / constant and compute width as `height × ASPECT`; in inline SVG, set `height` (or `width`) in CSS and leave the other as `auto`, and never set `preserveAspectRatio="none"`. In Figma, lock the aspect ratio (`constrainProportions = true`) on every logo instance. Non-uniform scaling, type substitution, or rasterising to a fixed-size box that distorts the glyphs are all violations. **Why immovable:** the wordmark is the most reproduced brand asset; any distortion or text-substitution compounds across deployments and is the hardest brand defect to retract once shipped.
16. **Motion: only animate `transform`, `opacity`, and `filter`.** See the Motion section. Animating layout properties (`width`, `height`, `padding`, `margin`, `max-height`, `max-width`, `top`, `left`, `right`, `bottom`) causes layout thrash on every frame — janky on any device under load. For accordions / expand-collapse, use `display: grid` with `grid-template-rows: 0fr → 1fr`; for reveals use `transform: scaleY()` or `translateY()` paired with `opacity`. Respect `prefers-reduced-motion: reduce` (WCAG 2.3.3) — override all duration tokens to `1ms` and easings to `linear`.
17. **IBM Plex (all variants): plain zero only (immovable).** All `0` glyphs in IBM Plex Sans and IBM Plex Mono must render as the **plain** variant — never dotted, never slashed. The OpenType feature *tags* are identical across the family (`zero`/`ss03` = slashed feature; `ss04`/`salt` = the other alternate), but the *glyph each tag substitutes to* differs per font: **Plex Sans's default `zero` is already plain** — adding `ss04` flips it to DOTTED (the opposite of what you want). **Plex Mono defaults to DOTTED** — `ss04` flips it to plain. So the correct web rule is: *scope `font-feature-settings: 'ss04' 1` to Plex Mono elements only; leave Plex Sans elements at default*. Do not apply a universal `*` selector for `ss04` — it will make Plex Sans dotted. **Figma:** the toggle labels are font-specific — Plex Sans's panel shows \"slashed\" and \"dotted\" (both opt-in; default is plain), while Plex Mono's panel shows \"slashed\" and \"plain\" (default is dotted; toggle \"plain\" ON). **Font source matters:** Google Fonts ships a subset that strips the stylistic sets — self-host or load from jsDelivr (`@ibm/plex-sans@1.1.0`, `@ibm/plex-mono@1.1.0`). Do not use `'salt' 1` as a shortcut; it flips all alternates including single-storey `a` and `g`. See the \"IBM Plex — Zero Glyph\" section for the full per-family substitution table. **Why immovable:** the dotted zero is the second-most-reproduced typographic detail in DocuSketch surfaces (after the wordmark), and inconsistency between dotted/plain across pages is a visible brand defect.
18. **Generative AI: two gates, immovable.** Generative AI shows the product in use and builds the job site around it; synthesized damage (moisture, staining, debris, smoke residue) shown calmly is in scope. (1) **Never develop alarming imagery:** no current or recent weather events or disasters; no people in danger, injuries, collapse, or dread. Test: inform, not alarm. (2) **Never synthesize a real person:** customers, testimonial givers, subject-matter experts — no likeness, voice, or attributed words; anyone presented with a voice in real time is real. Never generate the product, evidence from a real job, or facts in copy. A generated scene may show a realistic job site but is never presented as a specific real job or customer; files carry `gen-`. **Why immovable:** imagery of a live event turns a promise of calm documentation into a false alarm, and a synthesized customer or expert turns a true statement into a fabricated one the moment it is discovered.
19. **No orphans (all type, every breakpoint).** A single word must never sit alone on the last line of a heading, title, card title or paragraph. Fix it in this order: (a) if the heading is meant to read as one line, **reduce its size one step** (or narrow the copy) until it fits on one line; (b) otherwise **give the last word a friend**: bind the final two words with a non-breaking space (`&nbsp;` / U+00A0) or a soft return (Shift+Return in Figma, `<br>` only when the line count is fixed) so they wrap together; (c) on the web also set `text-wrap: balance` on headings and `text-wrap: pretty` on body copy. Check every breakpoint (desktop, tablet, mobile); an orphan at any width is a defect. See Type Styles → Line breaks — no orphans.
20. **Proportional states (hover, press, motion), Chris Provins 2026-10-06.** An interactive state is a small step inside the control's own colour family, never a swap. Every state colour is a DS palette step, never a computed mix: no `color-mix()` and no alpha tint of a brand colour, because a tint over a surface lands off-palette (Olive 6% over a light stage reads as #EFEFEB; Chris Provins 2026-10-07). Each state takes the next palette step in the same family, never a tint. Hover moves fill lightness by one palette step (hard limit 0.10); press takes the next step on (hard limit 0.20 from rest); the label keeps 4.5:1 in every state. **Never on hover or press:** flip dark to light or light to dark; switch hue family (Olive to Chartreuse, an outline to a solid fill); recolour one segment of a joined control (split button, button group, segmented control, toolbar) so the group's silhouette breaks; change a label's colour; scale outside 0.98 to 1.02 or move more than 2px (the card lift is the 2px ceiling); grow a halo past 4px or tint it chartreuse (halos are Neutral 300, then Neutral 400). Recipes: Olive fill to Neutral 800, then Chartreuse 900 `#2A2808`; Chartreuse 300 to 200, then 400; outline and ghost take Neutral 200, then Neutral 300, inside their own shape (on the Neutral 200 page, Neutral 300, then Neutral 400); danger holds its Signal/Error fill and steps the outline (Neutral 800, then Chartreuse 900); selection stays Chartreuse 300 and steps the same way. Entrances start from at least 0.8 scale with no overshoot; pulses are opacity only. **Exception:** the full colour inversion is a signature move reserved for the site's standalone primary marketing CTAs as built (Sticky CTA, hero and closing Book a demo, product-row Learn more, the tour poster); never in a group, never in a system component. Enforced by the state and restraint audits (`scripts/audit/`); components follow `components/STATES.md`.

---

## Naming Convention & Source of Truth — 2026 Unification (canonical)

> **Decision 2026-07-15:** the **DS° Marketing Website 2026** Figma file (`guoAdcJQH5m7hrlyH62OGZ`) is the central source of truth for design-system **naming** — variables and assets. The Brand Design Kit remains the component library; this doc and brand-design-kit.vercel.app are mirrors. Full registry + per-surface impact map: `NAMING_ALIGNMENT.md` in the brand-design-kit repo.

**Grammar.** One canonical token name, `group-role` kebab-case, transformed mechanically per surface:

| Surface | Spelling | Example |
|---|---|---|
| Figma variable | `Group/group-role` | `Text/text-strong` |
| CSS / code | `--ds-` + canonical | `--ds-text-strong` |
| Figma component | `Category / Name` Title Case, variants `Property=Value` | `Logo / Partner / Paul Davis`, `Color=Black` |
| Dev handoff alias | kebab, in component *description* only | `logo-bar` |

Variant property vocabulary (closed list): `Color=Black|White|Chartreuse` (marks) · `Mode=Light|Dark` · `Device=Desktop|Mobile` · `Variant=` (everything else). `Brand=` is retired; values are Title Case.

**Semantic re-mappings that supersede the tables below** (the rest of the legacy `DS/Token/*` names remain valid values with legacy spellings until the section-by-section rewrite lands):

> **Retraction 2026-09-09.** The three text rows below originally inverted `text-primary` / `text-strong` and moved `text-inverse` to pure white. Those three re-mappings were never authored here — they entered through the third-party-built token file and were then mirrored into this section. The authored values stand, and the rows now state them. `text-primary` is the ink for body **and** headlines on light; `text-strong` is the high-contrast body-supporting role whose purpose is holding contrast when it flips to Neutral 200 in dark — it is not an emphasis step above primary. Every other row in this table is unaffected.

| Canonical | Value | Supersedes |
|---|---|---|
| `text-primary` | `#1A1905` | nothing — Neutral 700 remains body and headline ink (17.72:1 vs white). *Superseded 2026-10-05: `text-primary` is now Olive `#39381B`.* |
| `text-strong` | `#39381B` | nothing — Neutral 650 remains the body-supporting role (11.96:1 vs white) |
| `text-inverse` | `#F9F9F5` | nothing — Neutral 100 remains the ink on dark (16.79:1 vs Neutral 700) |
| `text-button-on-accent` | `#010101` | Retired 2026-10-05: button labels on accent use `DS/Token/Text/Primary`, Olive `#39381B` |
| `background-accent` | `#E5DF00` | `DS/Token/Background/Brand` (rename) |
| `brand-white` | `#FFFFFF` | white is reinstated as a brand token |
| `error` | `#FF7575` | any prior signal/negative red |

> **Ruling 2026-10-05 — Olive is the primary dark brand colour.** `Neutral 650` `#39381B` (Olive, Pantone 5747 C) is the brand's dark, everywhere: web and print, ink and fill. It replaces `Neutral 700` `#1A1905` (Brand Black) in every role Brand Black held:
>
> - **Ink:** `Text/Primary` resolves to Neutral 650 on every light surface, web and print alike. This supersedes the same-day web-only ink ruling, which kept `#1A1905` for print.
> - **Brand dark:** the token is renamed `Brand/Olive` (formerly `Brand/Ash`; CSS `--ds-brand-olive`, formerly `--ds-brand-ash`) and resolves to Neutral 650. "Ash" no longer names anything in the system. Dark bands and Campaign dark surfaces (`Background/Inverse`), Editorial hairlines, scrims and shadow tints all follow.
> - **Marks:** DocuSketch logos on light surfaces are Olive in print as well as on the web. On an Olive surface the mark is Chartreuse (White beside a chartreuse CTA). On Chartreuse it is Olive.
> - **On brand:** text on chartreuse fills, button labels included, uses `Text/Primary` (Olive). `Text/OnBrand` and `text-button-on-accent` `#010101` are retired: once Olive, they only duplicated Primary. Dark-theme constraint: Primary flips to sage `#C0BC90` in dark, which is 1.37:1 on chartreuse, so a dark theme must flip the pair (Olive fill, chartreuse text) or pin the text to `Brand/Olive`. This belongs to the open dark-mode ruling. Olive clears AA on every chartreuse step it covers (lowest 5.18:1 on Chartreuse 600). On an Olive band, chartreuse highlights (`Text/Accent`) are Chartreuse 300 (8.48:1).
> - **Contrast:** Olive is AAA against every light surface (11.96:1 on white, 10.74:1 on Neutral 200) and against Chartreuse 300 (8.48:1). One pair breaks: `Text/Secondary` Neutral 500 on an Olive band is 2.83:1, so secondary text on Olive uses Neutral 400 `#C0BC90` (6.17:1).
> - **What `#1A1905` keeps:** it stays a primitive (`Neutral 700`) for the dark theme's deepest surface, which is still an open question (see Dark Mode). It is no longer an ink, a logo fill or a brand surface.
> - **Not yet propagated:** the Brand Design Kit Figma variables (`Brand/Ash`, still to be renamed `Brand/Olive`; `Text/Primary`; `Background/Inverse`), `tokens/ds-tokens.css` (generated from Figma), the `Color=Black` logo variants and the brand kit site's own `:root`. Until Figma changes, bind these roles to Neutral 650 directly.
>
> Source: Chris Provins, 2026-10-05, building on the New Home GA draft (`/dev/new-home-ga`, page `6abae1da24071f5a642f0ffb`), where one Olive ink first replaced the near-black.

Ramp primitives (`chartreuse-100…900`, `neutral-100…800`, `black`, `white`) and the Editorial/print system (`DS/Type/Print/*`, Text/Secondary, Border scale) are **extensions**: absent from the website file, retained here in the same grammar, candidates to upstream.

---

## Brand Expressions

Two modes that DocuSketch design lives in. Same brand, two voices — they share wordmark, chartreuse, the plain-zero rule, type families, and grid discipline; they differ in surface, density, imagery, and accent usage. **Choose the mode by the reader's job**, not by which one "looks more designed."

### The two modes

**Campaign mode — attention in a noisy context.**

- Full-bleed `Brand/Olive` (`#39381B`) surface, scattered chartreuse "+" mark pattern.
- Hero imagery — the 3D iso restoration render (imageHash `f5b72c8ae46eb6e827de28ad588e8ef5be746658`) and its siblings.
- Chartreuse pill tags with IBM Plex Mono caps (`+0.08em` tracking).
- IBM Plex Sans Medium/SemiBold cream headlines, tight tracking, on dark.
- Format reference: full-letter with bleed (`840×1080`).
- Canonical examples in DS Print – One Pagers (`lVyfilyeHp4Vcyc3pq4m1O`) → `DocuSketch 360AI` page: Servpro Vendor Guide (`3361:1003`), Paul Davis Full Page Ad (`3296:429`).

**Editorial mode — honest information in a quiet context.**

- Warm white surface (`Background/Subtle`, `#F9F9F5`).
- **No hero imagery** unless it informs the reader's task — typography carries the work.
- 1px `Brand/Olive` hairlines define the content frame; no shadows, no card borders.
- **One** deliberate chartreuse accent per page (~5×5 square mark), placed semantically.
- IBM Plex Sans Medium headlines (modest sizing, 24–28px); IBM Plex Sans Regular body (13/150); IBM Plex Sans Condensed Medium for caps labels (9–10px, `+12%` tracking).
- **No Plex Mono.** Side benefit: every zero renders plain by default — no manual `ss04` toggle ever needed (cf. BP #17).
- Format reference: true letter (`816×1056`), 72px page-edge margins.
- Canonical example: `Product Update` page in DS Print – One Pagers, frame `3965:76`.

### Choice matrix

| Context | Mode |
|---|---|
| Trade show, OOH, launch poster, hero ad, demo screen | **Campaign** |
| Customer letter, product update, beta cohort note, executive memo | **Editorial** |
| Internal at-a-glance asset (Slack image, status card, dashboard hero) | Campaign (compressed) |
| Whitepaper, longform PDF, technical brief | **Editorial** |
| Full-page ad in a trade magazine, conference banner / booth | **Campaign** |
| In-product release notes, account-facing PDFs | **Editorial** |
| Investor update, partner-facing note, security disclosure | **Editorial** |

If a brief sits between, the mode is the one whose principles serve the **reader's job**, not the one that looks more designed. When in doubt, pick Editorial — under-designed almost never reads as wrong.

### Editorial — the ten principles

1. **Honest over persuasive.** Say the thing. Don't decorate it.
2. **Less, but better.** Every element earns its place — if removing it doesn't lose meaning, remove it.
3. **Typography carries the work.** No imagery unless it informs.
4. **Hairlines, not borders or shadows.** 1px `Brand/Olive` rules define structure; nothing else.
5. **One deliberate chartreuse accent per page.** A 5×5 square, placed semantically (e.g. next to a section identifier). Never two; never bigger.
6. **Sans only.** IBM Plex Sans + IBM Plex Sans Condensed for caps labels. Plex Mono belongs to Campaign — its dotted default zero requires a manual `ss04` toggle the Plugin API cannot set. Editorial side-steps the problem entirely.
7. **Wide margins.** 72px (~0.75 in) from page edge at letter format; scale proportionally for other sizes.
8. **Two-column grid with metadata sidebar.** At letter format: sidebar `144` + gutter `48` + body `480`. Sidebar holds labelled metadata (FROM / SUBJECT / DATE); body holds the letter. The sidebar may end early — the lower negative space is intentional, not under-filled.
9. **Numbered lists, not bullets.** Plex Sans Medium numerals in a fixed-width slot (`22px` at letter), `16px` gap, then item text. No icons, no checkmarks, no "+" markers.
10. **Hierarchy through size and weight, not colour.** Body, signature, labels, footer — all share the ink tone. Differentiation is typographic.

### Tokens an Editorial piece uses

All existing — no new tokens required.

| Role | Token | Hex |
|---|---|---|
| Surface | `Background/Subtle` | `#F9F9F5` |
| Ink (body, headline, signature name) | `Text/Primary` | `#39381B` |
| Labels (sidebar metadata, signature title, footer left URL) | `Text/Secondary` | `#807C5E` |
| Hairlines (top + bottom of content frame) | `Brand/Olive` 1 px | `#39381B` |
| Single accent mark (5×5 square) | `Brand/Chartreuse` | `#E5DF00` |

### Editorial type ramp (letter format, `816×1056`)

| Role | Family | Weight | Size | Line height | Tracking |
|---|---|---|---|---|---|
| Headline | IBM Plex Sans | Medium | 26 | 124% | −2% |
| Body | IBM Plex Sans | Regular | 13 | 150% | 0 |
| Numerals (list) | IBM Plex Sans | Medium | 13 | 150% | 0 |
| Signature name | IBM Plex Sans | Medium | 14 | 120% | 0 |
| Signature title, sidebar subvalue | IBM Plex Sans | Regular | 11.5 | 140% | 0 |
| Sidebar value | IBM Plex Sans | Medium | 12 | 130% | 0 |
| Caps labels (sidebar, header tag, footer right) | IBM Plex Sans Condensed | Medium | 9–10 | 100% | +12% |
| URL (footer left) | IBM Plex Sans | Regular | 10 | 100% | 0 |

**Fonts to load (Plugin API):** `IBM Plex Sans` Regular + Medium, `IBM Plex Sans Condensed` Medium. That's the complete font surface — no other family is permitted in Editorial.

### Canonical divider — print sizing rule

The "+" mark in the canonical divider (`Navigation / Footer`, `266:1311`) has **two fixed sizes**:

| Context | Mark size | Frame widths | Used in |
|---|---|---|---|
| **Web / native** | `26 × 26` | `1466` (canonical) | Website, full-bleed digital |
| **Print** | `18 × 18` | `480`, `672` (Editorial) | Letters, one-pagers, PDFs |

**Marks never scale between these two values.** The frame width adapts to the container; the mark size stays fixed.

**Spacing formula** — marks sit at the left edge, geometric center, and right edge of the frame:

```
left   = 0
center = (W − S) / 2
right  = W − S
```

Where `W` is frame width and `S` is mark size (26 or 18).

**Pre-built Editorial widths** (live in `Document / Divider`):

| Width | Marks (18 × 18) at | Container |
|---|---|---|
| `480` | `0 / 231 / 462` | Body column inside Editorial letter |
| `672` | `0 / 327 / 654` | Letter content area (sidebar + body) |
| `1466` *(native, 26 × 26)* | `0 / 720 / 1440` | Site / banner / full-bleed |

**Implementation.** Drag the `Document / Divider` variant (`Letter` or `Body`) for print, or the canonical `Navigation / Footer / Mode=Light, Scale=Print` variant for native print. **Never call `instance.resize()` on the canonical alone** — it crops content rather than scaling vectors. For uniform geometry scaling between same-scale widths, use `node.rescale(scale)`; otherwise reposition marks per the spacing formula. The decision is: stay within one scale tier (web 26 → web 26 with rescale, or print 18 → print 18 with reposition). Never mix the two within a single piece.

### Canonical mark placement (immovable)

The brand structural mark (canonical vector `266:1319` inside `Navigation / Footer`, currently a "+" cross) **never appears as scattered individual marks** on a layout. It appears only as a **horizontal row of three**, spanning the available frame width.

**The 3-in-a-row rule.** Marks align by the **vertical centerline** of the "+" symbol (not the outer edge of its bounding box). The centerline coincides with the column-perimeter axes — left padding edge, page horizontal center, right padding edge — so the visual rhythm reads as a true grid alignment rather than a "+" hanging inset from the column.

Given content edges at `x = L` (left padding) and `x = R` (right padding), and mark size `S`:

| Position | Centerline target | Vector left edge (= centerline − S/2) |
|---|---|---|
| Left mark | `x = L` | `x = L − S/2` |
| Center mark | `x = (L + R) / 2` | `x = (L + R)/2 − S/2` |
| Right mark | `x = R` | `x = R − S/2` |

Where `S` is mark size (`26` web / `18` print — see the sizing rule above). The mark size stays canonical at all widths; the frame holding the row spans `[L − S/2, R + S/2]` so it contains the marks while the centerlines sit on the column edges.

**Implementation in Figma.** Build the row as a HORIZONTAL auto-layout frame with `primaryAxisAlignItems = SPACE_BETWEEN`. With three children at fixed widths `[S, content_width, S]`, the layout engine distributes the remaining space evenly, putting the outer marks at the row's left/right extremes and the middle item (mark or eyebrow pill) at center.

**Top-row exception when an Eyebrow component sits at center.** If the top row contains an Eyebrow pill (page-header label) at its centerline, **omit the center "+"** — the pill semantically replaces the center mark. The pattern becomes `[Left +, Eyebrow pill, Right +]`. The bottom row keeps all three marks since the footer label (`DOCUMENT FOOTER · NN`) sits *below* the row rather than between the marks.

**Vertical placement.** A row may sit at one of three positions to anchor a layout:

- **Top** — frames the upper edge of content
- **Vertical center** — divides the layout into upper and lower halves
- **Bottom** — frames the lower edge of content

One row per position. A single layout uses one or more positions (never two rows at the same position).

**Forbidden:**

- Scattered individual marks across the page
- Marks at corner positions only (e.g., a single mark at top-right)
- Off-center groupings (marks not anchored to frame left / center / right)
- Vertical or diagonal rows
- Rows at arbitrary y positions (other than top, vertical center, bottom)

**Symbol form (immovable).** The canonical mark is the brand "+" — two perpendicular strokes, defined by vector paths on `266:1319` in the Brand Design Kit. **It is the only valid form.** Never substitute a hyphen, dash, em-dash, dot, or any other simplified shape. When placing the mark, use the canonical SVG vector by cloning `266:1319` (or by instancing `Document / Divider` / `Navigation / Footer / Mode=Light, Scale=Print` and detaching) — never redraw the cross from primitives.

### Column layout typography

**Cap-top alignment between adjacent columns.** When two columns sit side-by-side at the same starting `y` and lead with text of different sizes, the larger-text column appears to start LOWER — because its line-height carries more leading-above the cap-top. To visually align the cap-tops of the first glyphs in each column, shift the smaller-text column DOWN by the leading-above difference:

```
shift = (line_height_larger − font_size_larger) / 2
      − (line_height_smaller − font_size_smaller) / 2
```

Example: Body XL (20 / 150% → leading-above = 5) on the left, Overline (11 / 100% → leading-above = 0) on the right at the same starting `y`. Shift the right column DOWN by ~5-6 px so the Overline's first cap-top sits at the same `y` as the Body XL's first cap-top.

This applies to any side-by-side column layout that mixes type sizes at the first line — Campaign hero + sidebar, Editorial metadata + body, marketing two-column copy.

**Container clipping and descenders.** Auto-layout frames in Figma default to `clipsContent = true`. When the frame wraps text whose line-height is at or near 100% (e.g., `DS/Type/H4`, `H3`, `H2`, `H1`, `Display`, `Overline`, `Button`, `Nav`, `Condensed Label`, `Code/*`, `Quote/Mono`), descenders (`g j p q y`) extend past the line-box bottom and get clipped by the parent frame. **For every auto-layout frame that contains text, set `clipsContent = false` explicitly.** The exception is intentional overflow control (marquee, cropped media) where the behavior is wanted.

### Component spacing — internal vs external

A reusable component owns its **internal** spacing (the gaps between its own children); the parent layout owns its **external** spacing (the gaps to its siblings).

- A component should NOT have outer padding on its frame, **unless** it is explicitly a surface / container variant (Card, Pill, Halo wrapper, Sticky CTA shell, etc.). Surfaces own their inner padding because the surface edge is part of their visual identity.
- An Eyebrow (Overline + Heading + optional Sub) defines its OWN internal `itemSpacing`: 8 px between Overline and Heading, 12 px between Heading and Sub. The Eyebrow does NOT have padding above or below itself — that spacing belongs to whatever places it (e.g., the right column's `itemSpacing`, the body container's gap to siblings).
- A Letterhead, Metadata Block, Numbered Item, Footer, Divider — same rule. They own their internal layout; the page/parent owns the spacing around them.

**Why:** components with built-in outer padding double up when placed in an auto-layout parent (component padding + parent gap = visible double-space). Designers who hit this often detach the component to fix it, which breaks the link to the source. Keeping outer padding at zero on non-surface components means a single component works cleanly across every context.

This is the same principle behind the spacing scale's two tiers (BP #12). Content tier values (`4·8·12·16·20·24·32·40·48`) live INSIDE components; Layout tier values (`56·64·80·96·128`) live in the parent / page context. Padding *of* the component is its content; padding *between* components is the parent's job.

### Layout patterns (multi-page documents)

Patterns surveyed from the "Your Guide to Remote Estimating" ebook in DS Print – One Pagers (`3141:711`). All pages are **612 × 792** — true 8.5 × 11 at PDF native dimensions (1 px = 1 pt → 8.5 × 11 inches at print). Templates of each pattern live on the `Product Update` page (`3929:504`) for visual reference.

| Pattern | Use case | Distinctive elements | Template |
|---|---|---|---|
| **01 — Cover** | Front cover of a multi-page document | Full-bleed dark (`Brand/Olive`) surface, centred chartreuse eyebrow pill, small wordmark, large Display headline, single subtitle line, large hero image filling lower half | `4041:87` |
| **02 — Image-led content** | Narrative pages where one image carries the story | Centred eyebrow + top "+" row, hero image at top (~⅓ height), single-column title left, body paragraph, chartreuse `THE RESULTS` callout box on right, footer | `4042:85` |
| **03 — Two-column header + numbered sections** | Body pages with multiple stepped points | Centred eyebrow + top "+" row, 2-col header (title left ⅔ / intro paragraph right ⅓), numbered sections below (`1.` `2.` …) each with subtitle, body, bullets, optional pull-quote with attribution, footer | `4043:85` |
| **04 — Three-column process grid** | Stepped-process or feature-set pages | Centred eyebrow + top "+" row, 2-col header, sub-section eyebrow + sub-title, three image cards in a row (each with `0X_STEP` numeric overline above an image), three subhead + body columns below the cards, chartreuse outcome callout banner across the full content width, footer | `4044:85` |
| **05 — Data table** | Metrics, before/after, comparison data | Centred eyebrow + 2-col header, 3-column table below with Mono caps header row (`DS/Type/Overline`-style), Olive `Plex Sans Medium` left column (row labels) + `Regular` value columns, 1 px `Neutral 300` hairlines between rows, optional muted footnote paragraph below, footer | `4047:85` |
| **06 — Stat callouts** | High-impact numbers / outcome summary | Centred eyebrow + 2-col header, 2 × 2 grid of large numbers (56 px Plex Sans Medium, −4% tracking) each paired with a `Condensed Medium` caps label in `Text/Secondary`, thin `Neutral 300` separators between cells, chartreuse outcome banner spanning full content width, footer | `4048:85` |
| **07 — Quote / testimonial** | Single-page anchor quote | Centred eyebrow + top "+" row, oversized chartreuse opening quote glyph (`Plex Mono Regular` ~96 px), pull quote in `Plex Mono Regular` ~28 px / 130% on Olive, attribution row below with circular portrait + name (`Plex Sans Medium`) + role (`Text/Secondary`), footer | `4049:85` |

**Frame and grid for body pages (02–04):**

- Page size: 612 × 792
- Padding: 48 px on left/right (content edges at `x = 48` and `x = 564`), 36 px above the top header row, equal below
- "+" centerlines align with column-perimeter axes (`x = 48 / 306 / 564`); the row frame spans `[39, 573]` (width 534) to contain the marks. See *Canonical mark placement* for the formula.
- Centred page eyebrow: instance of `Document / Page Eyebrow` (`Surface=Dark` on dark covers, `Surface=Light` on warm-white body pages); stroked outline (no fill) with surface-inverted stroke and text — Chartreuse on dark, Black on light. Text bound to `DS/Type/Print/Overline`.
- 2-col header proportions: title ~⅔ of content width (left), intro paragraph ~⅓ (right)
- 3-col grid proportions: equal thirds with 6 px gutter (168 + 6 + 168 + 6 + 168 = 516)
- Footer: centred, `DOCUMENT FOOTER · NN`, `DS/Type/Print/Overline`-style metadata caps in muted Eucalyptus tone

**Header pattern — auto-layout, not absolute positioning.** The top row is a HORIZONTAL auto-layout frame at `x = 39, y = 36, w = 534, h = 18`, with `primaryAxisAlignItems = SPACE_BETWEEN` and three children: `[Left + vector (18 × 18), Document/Page Eyebrow instance, Right + vector (18 × 18)]`. The Eyebrow auto-centers between the marks; the centerlines of left and right marks land exactly on the column-perimeter axes (`x = 48` and `x = 564`).

**Body content uses auto-layout containers, never absolute positioning.** Section content, column content, numbered lists — all wrap in vertical or horizontal auto-layout frames (hug primary axis, fixed counter axis). The reason: when type styles change line-height (e.g., switching between `DS/Type/*` and `DS/Type/Print/*`, or editing a placeholder), absolute-positioned content overflows into siblings, while auto-layout content reflows cleanly. Reserve absolute positioning for perimeter elements only — corner "+" marks (anchored to row frames), header (anchored to top), and the closing-zone element (anchored to the bottom zone, see below).

**3-zone vertical rhythm.** A template page composes three zones:

| Zone | Anchor | Role |
|---|---|---|
| **Top** | `y = 100` (below the header row) | Title row — eyebrow + title + optional intro |
| **Middle** | starts after the title row, hugs its content | Main content (sections, grid, table, stats, quote) |
| **Bottom** | bottom edge of the closing element sits at `y = 680` (58 px above the bottom "+" row at `y = 738`) | Closing element — outcome banner, footnote, byline |

The space between middle and bottom is intentional editorial breathing — not dead space. A page with no closing element (T1 Cover, T3 Numbered Sections) lets the middle content extend further and accepts negative space at the bottom as rest.

**Editorial vs Campaign in these patterns.** The four ebook patterns above sit closer to Campaign than Editorial — they use the chartreuse eyebrow pill, chartreuse callout boxes, and "+" rows. Editorial expression (warm-white surface, hairlines, single chartreuse accent, sans-only typography) applies to the **letter** family (one-pager letters, executive notes), while these ebook patterns apply to the **document** family (multi-page ebooks, whitepapers, longform reports). Both expressions share the same `Document / *` primitive components (Letterhead, Metadata Block, Numbered Item, Footer, Divider, Eyebrow) — the difference is which canonical elements (hairlines vs `+` rows, accent density, image use) each surface deploys.

### Editorial anti-patterns

Specifically *not* used in Editorial. Each of these is a Campaign primitive trying to leak in:

- Background patterns (plus marks, camo, gradients).
- Cards with fills, borders, or drop shadows. Cards are a Campaign primitive — Editorial uses hairlines and whitespace to separate content instead.
- Coloured surface backgrounds (chartreuse fills, dark fills) — Editorial is warm white only.
- Hero imagery — the iso render and its siblings belong to Campaign.
- Plex Mono caps labels — use Plex Sans Condensed instead.
- More than one chartreuse element per page.
- Bullet markers (`•`, `✓`, `+`) — use numerals.
- Decorative rules (chartreuse hairlines, dotted lines, double rules) — single 1px `Brand/Olive` hairlines only.
- Drop caps, pull quotes, magazine flourishes — keep it letter-quiet.
- Coloured text for hierarchy — use size and weight.

### Spacing carve-out: letter margin = 72

The Editorial letter page margin (`72`) is **not** on the documented layout-tier scale (`56·64·80·96·128` — BP #12). For print page-edge margins specifically, `72` (= ¾ in at 96 dpi) is added as a permitted Editorial-only value. If we extend further (A4 portrait at `793×1123`, legal at `816×1344`, postcard editorial, etc.), add those page formats and any new margin values here with a dated rationale.

### What inherits from the rest of the system

Editorial doesn't override the brand — it inherits everything except where this section states otherwise:

- **Wordmark** (BP #15): same component, same uniform-scaling rule. The `Black` variant is the default for Editorial pieces on warm white.
- **Plain-zero rule** (BP #17): inherited; Editorial satisfies it effortlessly because it uses no Plex Mono.
- **Spacing scale** (BP #12): use the content tier (`4·8·12·16·20·24·32·40·48`) for internal element gaps; the `72` carve-out above governs page-edge margins only.
- **Radius scale** (BP #11): Editorial mostly avoids radii — its primitives are hairlines and type, not cards. When a radius is needed, use the scale.
- **Motion**: not applicable — Editorial is print-first.

### Where the system pieces live

| Artifact | Location | Status |
|---|---|---|
| This section | `DS_Brand_Design_Skill.md` (you are here) | **Canonical** |
| `Document / *` component set — Letterhead, Metadata Block, Numbered Item, Footer, Divider, Eyebrow (content-area), Page Eyebrow (page-header pill) | Brand Design Kit Figma file (`JR35zTngKUblEKMD0myUyD`), `_Library` page | **Built** — Letterhead `1133:459` · Metadata Block `1134:445` · Numbered Item `1138:438` · Footer `1139:438` · Divider `1161:225` · Eyebrow `1170:229` · Page Eyebrow `1183:221` (Surface=Dark `1183:217` / Surface=Light `1183:219`) |
| `Navigation / Footer` (canonical divider) — `Mode=Light, Scale=Print` variant added | Brand Design Kit (`266:1311`) | **Built** — 2 variants: `Mode=Light, Scale=Web` (26×26) + `Mode=Light, Scale=Print` (18×18) |
| Editorial Letter — canonical example | DS Print – One Pagers (`lVyfilyeHp4Vcyc3pq4m1O`), `Product Update` page, frame `3965:76` | **Built** |
| Editorial Letter template (`816×1056` with hairlines, sidebar grid, footer pre-placed) | DS Print – One Pagers, new page | To build |
| "Two Expressions" reference page (side-by-side examples + choice matrix) | `brand-design-kit.vercel.app` | To build |

---

## Logo Usage

Five canonical marks, each shipped in three colour variants (`Brand Black` / `White` / `Chartreuse`). Since the 2026-10-05 ruling the `Brand Black` variant is filled with Olive (`#39381B`) wherever it is placed; the Figma variant keeps its old name until it is re-filled. All marks live on the `_Library` page of the [Brand Design Kit](https://www.figma.com/design/JR35zTngKUblEKMD0myUyD/Brand-Design-Kit) Figma file under `Logo / *` component sets. Always use the canonical SVG — never typeset, never recolour outside the three approved fills. See **Best Practice #15** for the immovable rule.

### The marks

| Token | viewBox | Aspect | Min size | When to use |
|---|---|---|---|---|
| `DS/Logo/Wordmark` | `0 0 200 25` | **8 : 1** | 80 px web · 0.5 in print | The primary mark. Site headers, sidebars, footers, OG cards, login screens, email signatures. |
| `DS/Logo/DS°` | `0 0 50.9 25` | ~ 2 : 1 | 24 px web · favicon 16 px | Compact mark for tight spaces — favicons, app icons, watermarks, dense lockups. Reach for this *before* shrinking the wordmark below floor. |
| `DS/Logo/DS1` | `0 0 275.7 27.05` | ~ 10.2 : 1 | 160 px web | Long-form horizontal lockup. Document footers, conference banners, trade-show booths — anywhere we have horizontal room to spare. |
| `DS/Logo/Pill` | `0 0 200 44.79` | ~ 4.5 : 1 | 96 px web | Tagged badge mark — wordmark inside a stadium shape. Beta badges, product sub-brands, sticker artwork. |
| `DS/Logo/Insta360 Partnership` | `0 0 440.9 37.5` | ~ 11.8 : 1 | 200 px web | Co-branded lockup with Insta360. **Joint marketing only** — never use to imply partnership where one is not formally agreed. |

The /brand reference exposes per-mark download links for all 15 SVGs (3 colours × 5 marks) at `/brand#logo`.

### Clear space

Every mark sits inside a protected zone equal to **1× its cap-height** on every side. Nothing else lives in this zone — no other type, no photographic elements, no chrome, no horizontal rules. This is the same rule applied to all five marks; cap-height is computed from the mark's full viewBox height (e.g. 25 units for the Wordmark, 44.79 units for the Pill).

### Colour-on-background combinations

Approved fills:

| Background | Approved fill | Notes |
|---|---|---|
| `Neutral 100` / `Neutral 200` / White, **web and print** | `Olive` (Neutral 650 `#39381B`) | Every light surface (ruling 2026-10-05, extended to print the same day): the mark shares the page's one ink. Same canonical SVG, filled with the ink token (`currentColor` or a fill bound to Neutral 650), never a recoloured raster. Figma has no `Color=Olive` variant yet; until it does, apply the fill to the `Color=Black` instance. |
| `Brand/Olive` (#39381B) | `White` *or* `Chartreuse` | Chartreuse on Olive (8.48:1) is the bolder, more on-brand option; reserve White for surfaces where the chartreuse would conflict with neighbouring chartreuse UI (e.g. a chartreuse CTA right beside the mark). |
| `Chartreuse 300` (#E5DF00) | `Olive` | Olive-on-chartreuse is the canonical brand combination (8.48:1) — see Gradients / Hero. |
| `DS/Pattern/Camo` | `Chartreuse 300` | Type on Camo is always chartreuse — see Patterns. |

Never combine: White-on-light, Olive-on-Camo, Brand-Black anywhere, Chartreuse-on-Chartreuse, any fill on a high-detail photo without a scrim.

### Sizing

Always scale uniformly — set **one** of `width` or `height` and let the other auto-derive from the viewBox. Never set both independently (BP #15). In inline SVG, never set `preserveAspectRatio="none"`. In CSS, `width: auto; height: <px>` or vice-versa.

For dense UI (chips, table headers, app-bar overflow), reach for the **DS° Mark** rather than shrinking the wordmark below its floor — below 80 px the wordmark's degree-symbol detail collapses and the brand reads as a generic logotype.

### Don'ts

These violations are not permitted on any DocuSketch surface — internal or external, draft or shipped. If a deployment does any of them, file it as a brand defect.

1. **Don't stretch** — non-uniform horizontal scaling distorts the letterforms.
2. **Don't squash** — non-uniform vertical scaling. Lock the aspect ratio (`constrainProportions = true` in Figma; `width: auto` or `height: auto` in CSS).
3. **Don't rotate** — the wordmark always reads horizontally. No tilts, no verticals, no diagonal lockups. The DS1 and Insta360 marks ship landscape *because* they include rotation in the SVG; consumers receive a landscape asset.
4. **No drop shadow, glow, or bevel** — the mark is flat by design.
5. **Don't outline** — no strokes around the glyphs. If contrast is the problem, pick a fill variant that contrasts the background. If contrast still isn't there, the background is wrong — change it.
6. **Never typeset** — `DocuSketch°` set in Plex Sans (or any other font) plus a `<sup>°</sup>` (or `°` glyph) is **not** the wordmark. The letterforms are custom-drawn, the degree-symbol placement is bespoke, and live text is at the mercy of the surrounding font stack. Always use the canonical SVG. *(Carve-out: the word "DocuSketch" may appear as live text inside running prose — body copy, headings, button labels, alt text, ARIA labels. See BP #15.)*

---

## Colour Tokens

> **Legacy dialect notice (2026-07-15):** the `DS/Color/*` and `DS/Token/*` spellings below predate the 2026 naming unification. Values remain authoritative **except** where the *Naming Convention* section above re-maps them (`text-strong`/`text-primary` swap, `text-inverse`, `text-button-on-accent`, `background-accent`, `brand-white`, `error`). New work should use canonical `group-role` names; full registry in `NAMING_ALIGNMENT.md`.

Two-tier palette mirroring Figma node `119:3`. **Primitives** are the raw colour values — the immutable palette. **Tokens** are semantic aliases that reference primitives — what components actually bind to, and what shifts automatically between Light and Dark mode. Never hardcode hex; never reference a primitive directly from a component when a token exists for that role.

### Primitives · Chartreuse

| Token | Hex | Pantone | Used as |
|---|---|---|---|
| `DS/Color/Chartreuse 100` | `#FFFCC4` | 600 C | Lightest chartreuse — derived as a paler tint of `Chartreuse 200`. Paper-tint backgrounds, washed surfaces. |
| `DS/Color/Chartreuse 200` | `#FFFA37` | 102 C | Brightest chartreuse — nested interactive state inside a chartreuse container (e.g. Sticky CTA icon button). Brighter than `Chartreuse 300` so it stands out against it. |
| `DS/Color/Chartreuse 300` | `#E5DF00` | **396 C** | Brand chartreuse — accent fills, text highlights *(= `Brand/Chartreuse` token)* |
| `DS/Color/Chartreuse 400` | `#C8C200` | 397 C | Muted mid-tone chartreuse |
| `DS/Color/Chartreuse 600` | `#B5AF00` | 398 C | Deeper chartreuse mid-tone |
| `DS/Color/Chartreuse 700` | `#8A8500` | 399 C | Dark chartreuse |
| `DS/Color/Chartreuse 800` | `#4A4510` | 5747 C | Very dark chartreuse / olive crossover |
| `DS/Color/Chartreuse 900` | `#2A2808` | Black 4 C | Darkest chartreuse *(backs `Text/Accent` on light; was `Text/OnBrand`, retired 2026-10-05)* |

*Scale gap at 500 is intentional — the previous step at #C2BC00 was visually indistinguishable from 400 (1.05:1 contrast). 400→600 now reads as a single, clearer mid-tone transition.*

### Primitives · Neutral

| Token | Hex | Pantone | Used as |
|---|---|---|---|
| `DS/Color/Neutral 100` | `#F9F9F5` | — | Common background — page / section warm white. Off-spec for Pantone (warm whites have no clean solid-coated match; print as 0/0/3/0 CMYK or use stock paper). *(= `Background/Subtle` + `Text/Inverse` tokens)* |
| `DS/Color/Neutral 200` | `#F4F3EA` | 9181 C | Common background — testimonial / card / inset surfaces *(= `Background/Default` + `Border/Subtle` tokens)* |
| `DS/Color/Neutral 300` | `#DFDDC8` | 7527 C *(re-verify)* | Dividers, image placeholder fills, warm-neutral text overlay on dark neutral swatches in dark mode *(= `Border/Default`, `Surface/Card`, `Text/Sticky-CTA` tokens — the separate `neutral-warm` `#E2E0D3` primitive that previously backed the latter two was collapsed into this value 2026-09-09; the two sat 1.03:1 apart)* |
| `DS/Color/Neutral 400` | `#C0BC90` | **5793 C** | Light sage / warm grey *(= `Brand/Eucalyptus` + `Text/Muted` tokens — note: 1.49:1 vs white, below AA. Reserve `Text/Muted` for non-essential / decorative text only.)* |
| `DS/Color/Neutral 500` | `#807C5E` | 5777 C | Mid sage / warm grey *(= `Text/Secondary` + `Border/Strong` tokens — note: 4.04:1 vs white, passes AA Large only.)* |
| `DS/Color/Neutral 650` | `#39381B` | **5747 C** | **The brand dark (Olive), ruling 2026-10-05** — body text, headlines, icons and logos on light; dark bands; Editorial ink and hairlines *(= `Brand/Olive`, `Background/Inverse`, `Text/Primary`, `Text/Strong` tokens)*. 11.96:1 vs white. |
| `DS/Color/Neutral 700` | `#1A1905` | Black 2 C | Formerly Brand Black. Since 2026-10-05 a primitive only: the dark theme's deepest surface, pending the dark-mode ruling. Not an ink, logo fill or brand surface. |

*Eucalyptus consolidated into Neutral 400/500/650 (this revision). The former Eucalyptus 100/200/300 (`#C0BC90` / `#807C5E` / `#39381B`) and prior Neutral 400/500/650 (`#B8B5A0` / `#908D68` / `#3D3C2A`) were within ~4 L\* of each other per step — a tonal duplicate scale. Picking the warmer Eucalyptus side as canonical eliminates the redundancy. `Brand/Eucalyptus` token now resolves through `Neutral 400`; the Eucalyptus primitive scale is removed. **Neutral 600** (`#6B6948`) was removed in the same pass — with the new Neutral 500 and 650 in place, the 500 → 650 step is large enough to carry without an intermediate. **Neutral 800** (`#4A4830`) is kept out of the light ramp for the same monotonicity reason, but retained in Figma as a Dark-mode-only primitive (it backs `Background/Subtle` and `Border/Default` in Dark mode).*

### Primitives · Base

| Token | Hex | Pantone | Used as |
|---|---|---|---|
| `DS/Color/Black` | `#000000` | Process Black C | Pure black — sparingly; prefer Olive / `Text/Primary` for body |
| `DS/Color/Base/White` | `#FFFFFF` | — | Pure white — surfaces, panels. No ink for printed white; reference stock paper. Not a brand-identity token (removed from Tokens · Brand) — reference the primitive directly. |

### Primitives · Status

Five light-fill / dark-ink pairs for status tags, pills and messages (ruling 2026-10-07, Chris Provins). Always used as a pair: light fill, dark ink, and on a card a 1px hairline in the dark ink. Each hue also has a **Mid** tone for marks that sit straight on a surface (ruling 2026-10-09). Full rules under *Status colours* below.

| Token | Hex | Pantone | Used as |
|---|---|---|---|
| `DS/Color/Status/Green Light` | `#EBFAC1` | — | Success fill (good news). Dark ink on it 11.6:1 |
| `DS/Color/Status/Green Mid` | `#43944F` | — | Success mark on a surface: status dot, icon, chart series, meter or progress fill. Marks only, never text or a tag fill. 3.56:1 on Neutral 100, 3.37:1 on Neutral 200 |
| `DS/Color/Status/Green Dark` | `#243805` | — | Success ink: the label and icon on Green Light, and the 1px hairline around a success tag on a card. 11.6:1 on Green Light, 12.1:1 on Neutral 100 |
| `DS/Color/Status/Yellow Light` | `#FEF2C3` | — | Warning / caution fill. Dark ink on it 9.7:1 |
| `DS/Color/Status/Yellow Mid` | `#9A7D04` | — | Warning mark on a surface: status dot, icon, chart series, meter fill. A mustard, so never beside a chartreuse accent. 3.75:1 on Neutral 100, 3.55:1 on Neutral 200 |
| `DS/Color/Status/Yellow Dark` | `#5F3100` | — | Warning ink: the label and icon on Yellow Light, and the 1px hairline around a warning tag on a card. 9.7:1 on Yellow Light, 10.3:1 on Neutral 100 |
| `DS/Color/Status/Red Light` | `#FFD6BD` | — | Error tint (peach): the fill behind an error tag, alert or invalid cell. Dark ink on it 11.2:1 |
| `DS/Color/Status/Red Mid` | `#E62C2D` | — | Error mark in data: error dot, failed chart series, over-limit meter segment. Form errors (ring, glyph, message) stay Signal/Error. 4.18:1 on Neutral 100, 3.96:1 on Neutral 200 |
| `DS/Color/Status/Red Dark` | `#4A1404` | — | Error ink: text and icons set inside the Red Light tint (error tag, alert, invalid cell), and its 1px hairline on a card. 11.2:1 on Red Light, 14.3:1 on Neutral 100 |
| `DS/Color/Status/Blue Light` | `#D4E6F4` | — | Info fill. Dark ink on it 5.7:1 (AA, not AAA) |
| `DS/Color/Status/Blue Mid` | `#3E87BC` | — | Info mark on a surface: status dot, icon, chart series, progress fill. 3.69:1 on Neutral 100, 3.49:1 on Neutral 200 |
| `DS/Color/Status/Blue Dark` | `#165D77` | — | Info ink: the label and icon on Blue Light, and the 1px hairline around an info tag on a card. 5.7:1 on Blue Light (AA, keep text 14px or larger), 6.9:1 on Neutral 100 |
| `DS/Color/Status/Purple Light` | `#DFD3D9` | — | Spare category fill (no fixed meaning; label it). Dark ink on it 11.4:1 |
| `DS/Color/Status/Purple Mid` | `#B55F8A` | — | Spare-category mark: dot, icon or chart series for a fifth category with no fixed meaning; label it. 4.00:1 on Neutral 100, 3.79:1 on Neutral 200 |
| `DS/Color/Status/Purple Dark` | `#3D072D` | — | Spare-category ink: the label and icon on Purple Light, and its 1px hairline on a card. 11.4:1 on Purple Light, 15.7:1 on Neutral 100 |

### Pantone matching — note

Pantones above are **best-attempt visual approximations** against the [Pantone Solid Coated library](https://www.pantone.com/connect/Pantone+Solid+Coated). Pantone is a printed-ink standard; RGB / hex is on-screen. The two systems don't map perfectly, and individual lighting, paper stock, and ink batch all shift the printed result. Before producing any high-stakes print collateral (signage, packaging, vehicle wraps), verify the match against a physical Pantone chip under the same lighting conditions as the final deliverable. For day-to-day digital work, the hex value is authoritative; use the Pantone column only when the print vendor asks for a spot reference.

The brand-defining matches — `Chartreuse 300 = 396 C`, `Neutral 400 = 5793 C` (the sage formerly known as Eucalyptus 100), `Neutral 650 = 5747 C` (the brand dark since 2026-10-05; it replaces `Neutral 700 = Black 2 C`) — are the three to pin first if a vendor needs a definitive brand-level reference.

> *Superseded 2026-10-05: the brand dark is now `Brand/Olive` (Neutral 650, Pantone 5747 C); the `Brand/Ash` name is retired. Kept for the record.* **2026-08-05 — Ash re-pinned to Black 2 C.** `Neutral 700` / `Brand/Ash` (`#1A1905`) was previously listed as Black 4 C; it is now canonically **Pantone Black 2 C**, whose olive/yellow undertone matches Ash's cast (Black 4 C reads brown). `Chartreuse 900` (`#2A2808`) keeps Black 4 C pending its own review. Email precedent for the family: the DS1 packaging warm grey was specced as Pantone 2330 U (Insta360 thread, Nov 2024) — an uncoated production reference, not a brand token.

### Tokens · Brand

Semantic aliases for the three colours that carry brand identity. Components should bind to these, not the underlying primitives. Pure white is a primitive (`DS/Color/White`) — it doesn't carry brand identity on its own.

| Token | Resolves to | Hex | Used as |
|---|---|---|---|
| `DS/Token/Brand/Chartreuse` | `Chartreuse 300` | `#E5DF00` | Primary brand fill — CTA pills, accent surfaces |
| `DS/Token/Brand/Olive` | `Neutral 650` | `#39381B` | The brand dark (ruling 2026-10-05; renamed from `Brand/Ash`) — ink on light, dark fills, hairlines, scrims |
| `DS/Token/Brand/Eucalyptus` | `Neutral 400` | `#C0BC90` | Muted sage brand accent (resolves through the Neutral scale; the former Eucalyptus primitive scale has been consolidated in) |

### Tokens · Background

| Token | Resolves to | Hex | Used as |
|---|---|---|---|
| `DS/Token/Background/Brand` | `Chartreuse 300` | `#E5DF00` | Chartreuse-fill sections |
| `DS/Token/Background/Inverse` | `Neutral 650` | `#39381B` | Dark sections and bands (Olive, ruling 2026-10-05). Secondary text on it uses Neutral 400 |
| `DS/Token/Background/Default` | `Neutral 200` | `#F4F3EA` | **Light web page and section background** (ruling 2026-09-16, confirmed on New Home 2026-10-05) |
| `DS/Token/Background/Subtle` | `Neutral 100` | `#F9F9F5` | Insets, testimonial and card surfaces one step lighter than the page; the Editorial print page (warm white) |

> The role text above was corrected 2026-10-05; it previously read Subtle = page, Default = inset. The token *values* never changed. **Still open:** the Dark Mode pairings below label `Background/Subtle` as the deepest (page) surface and `Background/Default` as one step up. Light web now uses the opposite assignment, so the dark mapping needs a ruling before any web surface ships a dark theme.

### Tokens · Text

| Token | Resolves to | Hex | Used as |
|---|---|---|---|
| `DS/Token/Text/Primary` | `Neutral 650` | `#39381B` | Body text and headlines on every light surface, web and print, and all text on chartreuse fills including button labels (Olive, ruling 2026-10-05; 8.48:1 on Chartreuse 300) |
| `DS/Token/Text/Secondary` | `Neutral 500` | `#807C5E` | Captions, supporting copy |
| `DS/Token/Text/Muted` | `Neutral 400` | `#C0BC90` | Tertiary / disabled text |
| `DS/Token/Text/Inverse` | `Neutral 100` | `#F9F9F5` | Text on dark backgrounds |

### Tokens · Border

| Token | Resolves to | Hex | Used as |
|---|---|---|---|
| `DS/Token/Border/Strong` | `Neutral 500` | `#807C5E` | Emphatic dividers, strong borders |
| `DS/Token/Border/Default` | `Neutral 300` | `#DFDDC8` | Standard dividers, hairlines |
| `DS/Token/Border/Subtle` | `Neutral 200` | `#F4F3EA` | Inset edges, subtle separations |

### Gradients

Canonical brand gradients composed from existing primitives. Source: the [DS° Marketing Website](https://www.figma.com/design/guoAdcJQH5m7hrlyH62OGZ/) Figma file. Use the CSS tokens — never recreate the math inline.

| Token | CSS variable | Stops | Use |
|---|---|---|---|
| `DS/Gradient/Hero` | `--ds-gradient-hero` | `0deg`, `Neutral 200` at `38.26%` → `Chartreuse 300` at `100%` | Hero / section-panel background. Dominant chartreuse occupies the upper ~62% with a warm-neutral foot. Used on workflow / feature panels. |

**CSS:**
```css
--ds-gradient-hero: linear-gradient(0deg, var(--ds-neutral-200) 38.26%, var(--ds-chartreuse-300) 100%);
```

The gradient is opaque end-to-end — every pixel of the gradient line carries either a solid Neutral 200 fill (below 38.26%) or an interpolated blend toward Chartreuse 300 (above 38.26%). No transparent fall-through; Figma's `Neutral 300` base layer is functionally redundant in code and omitted from the token definition for clarity.

### Patterns

Generative motifs built from existing primitives. Each pattern resolves to deterministic SVG output for a given seed; designers and engineers should reference the canonical generator rather than re-creating the shapes by hand.

| Token | Mechanism | Palette | Use |
|---|---|---|---|
| `DS/Pattern/Camo` | Ten gaussian-blurred organic blobs (`feGaussianBlur stdDeviation=60`) over a `Chartreuse 400` base, with two stacked grain layers (`feTurbulence` — fine at α 0.42, coarse at α 0.22). Seeded for reproducibility. | `Chartreuse 300 · 400 · 600 · 700 · 800 · 900` (chartreuse family only — Eucalyptus removed; it read cooler than the rest of the camo) | Heavyweight section backgrounds, hero overlays, marketing-page sectional dividers where a richer texture than the flat gradients is wanted. Source: the [DS° Marketing Website](https://www.figma.com/design/guoAdcJQH5m7hrlyH62OGZ/) Figma file. |

The Camo pattern is intentionally **non-token** at the CSS level — it ships as a generator function (`generateCamoSvg(seed, { width, height })`) rather than a `var(--ds-pattern-…)` because the output is a complete SVG document, not a single colour value. The /brand reference exposes a per-instance **Animate** toggle that adds a 22 s ease-in-out drift to each blob via CSS keyframes; static is the default, and `prefers-reduced-motion: reduce` disables the animation entirely.

**Type over Camo — Chartreuse 200 (revised 2026-09-18, Chris Provins).** Any typography, hairline, pill outline or icon fill rendered over `DS/Pattern/Camo` is set in `DS/Color/Chartreuse 200` (`#FFFA37`), the brightest step in the family. It lifts cleanly off both the dark blobs and the Chartreuse 400 base, where Chartreuse 300 read as one flat colour with the pattern. Secondary copy sits at 85% opacity, strokes at 50% alpha, interactive tints at 10–18% alpha of the same colour. Icon glyphs inside a Chartreuse 200 fill are Brand/Black — the canonical primary-fill pairing. **Do not** typeset over camo in Brand/Black or Olive (both were tried on the Capture page and read as a poster, not a brand surface), in Chartreuse 300 (flat against the pattern), or in white, Plex grey or any neutral (foreign, reads as tacked on). Hover/active button fills still flip to solid chartreuse with Brand/Black text. If a piece of type genuinely cannot read on a given seed, regenerate the camo with a new seed rather than changing the colour. The Capture page workflow panel is the reference application; the pre-footer CTA band that used Chartreuse 300 predates this rule and should be brought into line when next touched.

**Accessibility audit notes (WCAG 2.x):**

- `Neutral 100 → 300` are all ≤ 1.4:1 vs each other — intentionally subtle. Use them as adjacent background surfaces (Subtle / Default / Border), not for text differentiation.
- `Neutral 500` (#807C5E) is **~4.0:1 vs Base/White (#FFFFFF)** — passes **AA Large text only**. The `Text/Secondary` token resolves here; do not use it for body copy, only ≥18 pt / 14 pt bold. For body-sized secondary text use `Neutral 650` (#39381B, 11.96:1 vs white).
- `Neutral 400` (`Text/Muted`, #C0BC90) is **~1.5:1 vs Base/White (#FFFFFF)** — below AA. Reserve for non-essential / decorative text only; never load-bearing content.
- `Chartreuse 200` (#FFFA37) at 1.16:1 vs Base/White (#FFFFFF) means *do not* place black-text-on-yellow as the only signal — pair with iconography or weight.

**Text-on-swatch rule (text colour over a solid brand colour):** the light/dark decision is contrast-driven, the light-text *choice* is family-matched.

- **Dark-text test:** if `Text/Primary` (Olive #39381B) clears **AA (≥ 4.5:1)** on the swatch, the background is light enough — use `Text/Primary`. (Covers Chartreuse 100–600, lowest 5.18:1 on Chartreuse 600, and Neutral 100–400.)
- **Otherwise the background is dark/mid and needs light text, picked by family** so the overlay stays inside that family:
  - **Chartreuse swatches → pale `Chartreuse 100` (#FFFCC4).** Keeps the chartreuse identity and gives clean hue separation on the dark olive steps (700 / 800 / 900), where dark-olive-on-olive reads muddy. Applies to **Chartreuse 700, 800, 900**.
  - **Neutral swatches → `Neutral 300` (#DFDDC8).** A warm off-white that keeps the neutral overlay inside the warm-neutral family. Stark pure-white text (Neutral 100) on a warm dark neutral swatch reads as "tech-flat," not brand — Neutral 300 carries DocuSketch's warm identity into the contrast. ~12.85:1 on Neutral 700, ~8.73:1 on Neutral 650, ~3.16:1 on the Neutral 500 mid-tone (same AA Large limit as the other mid-tones — Neutral 500 is the tightest pair, just clearing 3:1). Applies to **Neutral 500, 650, 700** (and the dark `Background/Inverse`, `Brand/Olive` surfaces, which are neutral-family).

The two mid-tone steps near L\*54 — **`Chartreuse 700` (#8A8500)** and **`Neutral 500` (#807C5E)** — can't reach AA with *any* brand text colour (their best lands at AA Large, ~3.7–4.0:1). Treat them as fill / large-display tones, not backgrounds for body-size copy. (The /brand colour page applies this picker automatically and shows the resulting WCAG level on each swatch.)

**Scale cleanup (this revision):**

- Removed `DS/Color/Chartreuse 500` (#C2BC00). The 400→500 step was ~1.05:1 — visually a duplicate of 400. The 400→600 step now reads as a single, clearer mid-tone transition. Scale gap intentional; no renumbering.
- Removed `DS/Color/Neutral 800` (#4A4830). Was lighter than `Neutral 650` (L*=0.063 vs 0.044) and `Neutral 700` (L*=0.009), breaking monotonic darkening. The dark end now reads 600 → 650 → 700 cleanly.

**Removed / renamed / renumbered tokens — migration guide:**

The previous flat `DS/Color/{Black,White,Warm,Default,Chartreuse[…],Eucalyptus[…],Olive,Neutral 300–600}` model is collapsed into the two-tier structure above. Map old names to new:

- `DS/Color/Black` *(legacy meaning, was #1A1905)* → `Neutral 700` (primitive); its semantic roles (`Brand/Ash`, now `Brand/Olive`, and `Text/Primary`) moved to Olive on 2026-10-05. True black `#000000` is now its own primitive: `DS/Color/Black`.
- `DS/Color/Base/White` → `Base/White` primitive only. The `Brand/White` semantic alias was removed — pure white is not part of the brand identity tier.
- `DS/Color/Neutral 100` *(#F9F9F5)* → `Neutral 100` or `Background/Subtle` / `Text/Inverse`
- `DS/Color/Neutral 200` *(#F2F1EA)* → `Neutral 200` or `Background/Default` / `Border/Subtle`
- `DS/Color/Chartreuse 300` *(#E5DF00)* → `Chartreuse 300` or `Brand/Chartreuse`
- `DS/Color/Chartreuse Light` *(legacy #F2EF88)* → discontinued. Chartreuse 200 is now `#FFFA37` (formerly Chartreuse Active); use Chartreuse 100 (#FFFCC4) for paper-tint surfaces.
- `DS/Color/Chartreuse Active` *(#FFFA37)* → `Chartreuse 200` (absorbed into the primitive scale).
- `DS/Color/Chartreuse 900` *(#2A2808)* → `Chartreuse 900` (`Text/OnBrand`, its old semantic, was retired 2026-10-05)
- `DS/Color/Chartreuse 400` *(#C8C200)* → `Chartreuse 400` (unchanged)
- `DS/Color/Chartreuse 700` *(#8A8500)* → renumbered to **`Chartreuse 700`**
- `DS/Color/Eucalyptus 100` *(#C0BC90)* → `Eucalyptus 100` or `Brand/Eucalyptus`
- `DS/Color/Eucalyptus 200` *(#807C5E)* → `Eucalyptus 200` (unchanged)
- `DS/Color/Eucalyptus 300` *(#39381B)* → `Eucalyptus 300`
- `DS/Color/Neutral 300` *(#DFDDC8)* → `Neutral 300` or `Border/Default`
- `DS/Color/Neutral 500` *(was #908D68)* → renumbered to **`Neutral 500`** (Figma's new `Neutral 400` is `#B8B5A0`)
- `DS/Color/Neutral 600` *(was #6B6948)* → renumbered to **`Neutral 600`**
- `DS/Color/Neutral 650` *(was #3D3C2A)* → renumbered to **`Neutral 650`**

**Newly added** (were missing from our doc; sourced from the Brand Design Kit Colour page):
Chartreuse 100, 500, 600, 800; Neutral 400, 800; true Black `#000000`; the entire Tokens semantic layer (16 aliases).

### Web ink and surfaces

Rulings confirmed by the New Home page (New Home GA draft (`/dev/new-home-ga`, page `6abae1da24071f5a642f0ffb`), approved as canon by Chris Provins 2026-10-05):

| Role on a light web page | Value | Contrast |
|---|---|---|
| Page and section background | `Background/Default` · Neutral 200 `#F4F3EA` | — |
| Inset, card, testimonial surface | `Background/Subtle` · Neutral 100 `#F9F9F5` | — |
| Headings and body copy | Neutral 650 `#39381B` (Olive) | 10.7:1 on `#F4F3EA` |
| Dark band | `Background/Inverse` · Olive `#39381B`, type in `Text/Inverse` `#F9F9F5`; secondary text in Neutral 400 | 11.3:1 (Neutral 400: 6.2:1) |
| Hairlines and row dividers | Neutral 400 `#C0BC90` (decorative, never text) | — |

Hierarchy on the page comes from size, weight and space; every heading, paragraph, ink stroke and DocuSketch mark shares the one Olive ink. A near-black `#1A1905` heading beside Olive row titles read as unresolved, which is why the whole page moved to one ink.

### Binding every colour (web)

Every colour a web page draws comes from a DS token: type, strokes, fills, SVG presentation attributes, gradients, shadows and scrims. The New Home page was verified by setting every DS colour variable to magenta and checking all 551 rendered colours followed. Rules that fell out:

- **Never the Relume `default` collection.** Webflow's starter collection (literally named `default`, e.g. `Text Color/text primary`) is not the brand kit. Bind to `DS / Color Tokens` (`--_ds--color-tokens---*`) and `DS / Color Primitives`.
- **SVGs with a baked hex** (crosshair marks, arrows, dot tiles) are drawn as a CSS `mask` filled with a token (`background: var(--token)` or `currentColor`), so they recolour with the system.
- **Shadows and scrims** mix Brand/Olive: `color-mix(in srgb, var(--brand-olive) 12%, transparent)`, never a raw `rgba(0,0,0,…)` on brand pages.
- **The camo generator's colours** are chartreuse primitives (300 / 400 / 700 / 800 / 900 + neutral ink) and get bound to them.
- **Exceptions:** App Store and Google Play badges (store artwork stays unaltered), photography, and product UI shown inside a render (it follows the product design system, not brand accents).

### Text selection

When a user drag-selects copy on any DocuSketch surface, the selection reads as a **brand affordance** — `Chartreuse 300` (`#E5DF00`) background, `Text/Primary` (Olive `#39381B`) text. Same colour pair as primary CTAs, gradient hero, and the chartreuse-on-camo type rule. The OS-default blue is not used.

```css
::selection {
  background-color: var(--ds-brand-chartreuse);  /* Chartreuse 300 — #E5DF00 */
  color: var(--ds-text-primary);                  /* Olive — #39381B */
}
::-moz-selection {
  background-color: var(--ds-brand-chartreuse);
  color: var(--ds-text-primary);
}
```

The Firefox prefixed pseudo (`::-moz-selection`) needs its own rule — Firefox will not honour a comma-grouped selector mixing both. Inside dark-mode surfaces (or any block where the text colour is already chartreuse), authors flip the pair for legibility: `background-color: var(--ds-brand-olive); color: var(--ds-brand-chartreuse);`. In dark mode the flip is required: `Text/Primary` turns sage there and fails on chartreuse. The pair *always* uses these two tokens — never substitute a neutral or a secondary brand colour.

### Error colour (web) — ruling 2026-10-07

`Signal/Error` `#FF7575` is the one error colour on the web: the 2px inset ring on an invalid field **and** the error message text under it. There is no separate darker error ink. In the component kit `--dsc-error-ink` resolves to `var(--dsc-error)`.

**Known contrast exception (accepted).** `#FF7575` text measures 2.47:1 on Neutral 100 `#F9F9F5`, 2.34:1 on Neutral 200 `#F4F3EA` and 2.61:1 on white: below AA for text (4.5:1) and below the 3:1 UI minimum for the ring. Chris Provins accepted this on 2026-10-07. Keep the value; don't swap in a darker red to pass. Pair every error with something besides colour: the message wording, an error glyph, and `aria-invalid` with `aria-describedby` on the field.

#### Real-world context

- **2026-10-07 · Remote Estimating Ebook pages** (`/lp/remote-estimating-ebook` and `/docusketch-guide-to-remote-estimating`, embed "Brand v2 CSS (ebook)"). Restyling the HubSpot form with the v2 kit, the kit's provisional dark-red error ink (off palette, never canon) was replaced by `Signal/Error`, the brand's only red, for both the invalid ring and the message text (Plex Sans 14 / 1.4, sentence case). Chris ruled that the palette red carries the error state on its own, and accepted its sub-AA text contrast as a known exception.

### Status colours — ruling 2026-10-07

Chris Provins' status pairs (designed ~2024 for the Figma DS Brand UI kit, file `iL3MqRVVsyma2D5kL8kZm9`, node `576:952`, and adopted by the product design system as `--{color}-light` / `--{color}-dark` in `colors_and_type.css`) are canon for status on brand surfaces. They shipped first on the marketing dashboard's Expected tags (`--md-signal-*`, 2026-10-07). Values are in *Primitives · Status* above.

| Pair | Role | Light fill | Dark ink | Ink on fill | Olive on fill | Fill vs Neutral 100 |
|---|---|---|---|---|---|---|
| Green | Success, good news | `#EBFAC1` | `#243805` | 11.6:1 | 10.8:1 | 1.05:1 |
| Yellow | Warning, caution | `#FEF2C3` | `#5F3100` | 9.7:1 | 10.7:1 | 1.06:1 |
| Red | Error tint (peach) | `#FFD6BD` | `#4A1404` | 11.2:1 | 8.9:1 | 1.27:1 |
| Blue | Info | `#D4E6F4` | `#165D77` | 5.7:1 (AA only) | 9.4:1 | 1.21:1 |
| Purple | Spare category | `#DFD3D9` | `#3D072D` | 11.4:1 | 8.2:1 | 1.38:1 |

CSS: primitives `--ds-status-{green,yellow,red,blue,purple}-{light,dark}`; component roles `--dsc-{success,warning,info}-bg` / `-ink` and `--dsc-error-bg` / `--dsc-error-bg-ink` (`skill/components/shared/base.css`).

**Usage.**

- **Always the pair.** Light fill with its own dark ink for the label and icon. Olive also reads on every fill (8.2–10.8:1) and is fine for body text inside a status panel, but the tag label takes the pair's ink.
- **Hairline on cards.** The fills sit only 1.05–1.38:1 off Neutral 100 and 1.01–1.30:1 off Neutral 200, so a pill or tag on a card or page carries a 1px inset hairline in its dark ink. Without it green and yellow disappear on the warm whites.
- **Colour never alone.** Every status says it in words (and an icon where there is room): "On track", "Behind", "Failed". Colour is the second signal.
- **Sentiment, not direction.** Green means good news and yellow means caution, whatever way the number moved (a falling cost is green).
- **Caution is yellow, not peach.** The red light fill is the error tint; don't use it for "attention" or "warning".
- **Success is green, not chartreuse.** Chartreuse 300 stays the brand accent: CTAs, selection, highlights. A chartreuse "success" reads as a button or a selected state. `--dsc-success` moved from Chartreuse 300 to the green pair on 2026-10-07.
- **Blue is AA only** (5.7:1). Keep blue-ink text at 14px or larger.
- **Purple has no fixed meaning.** Use it for a fifth category only, and label it.
- **Light theme only.** The pairs are light-surface colours; there is no dark-mode mapping yet.

**Mid tones (ruling 2026-10-09, Chris Provins).** Each status hue has a Mid between its light fill and dark ink, for marks that sit straight on a page or card with no tag around them: status dots, icons, chart series, meter and progress fills, map pins.

| Mid | Hex | OKLCH | On Neutral 100 | On Neutral 200 | On white |
|---|---|---|---|---|---|
| Green Mid | `#43944F` | 0.60 0.130 | 3.56:1 | 3.37:1 | 3.76:1 |
| Yellow Mid | `#9A7D04` | 0.60 0.122 | 3.75:1 | 3.55:1 | 3.96:1 |
| Red Mid | `#E62C2D` | 0.60 0.220 | 4.18:1 | 3.96:1 | 4.41:1 |
| Blue Mid | `#3E87BC` | 0.60 0.109 | 3.69:1 | 3.49:1 | 3.89:1 |
| Purple Mid | `#B55F8A` | 0.60 0.122 | 4.00:1 | 3.79:1 | 4.23:1 |

- **Marks, not text or fills.** Every Mid clears 3:1 for graphics on Neutral 100, Neutral 200 and white, but none reaches 4.5:1, so status words stay in the pair's dark ink. Never use a Mid as a tag fill (the light fill does that job) or as a page or card surface.
- **Text on a Mid** (rare: a label on a swatch or a large figure, 14px or larger) uses the hue's own Light, or its Dark where the Light falls under 3:1. Green, yellow, red and blue take their Light (3.04–3.52:1); purple takes its Dark (3.92:1). AA Large only, so never body copy (ruling 2026-10-09).
- **One lightness.** All five sit at OKLCH lightness 0.60, so no status shouts louder than another in a chart or a row of dots. If a sixth status hue is ever added, set it to the same lightness.
- **Where they come from.** The hues are the product design system's `--{color}-base` tones (`shared/tokens/colors_and_type.css`: green `#64B56E`, yellow `#DFBB3A`, red `#E92F2F`, blue `#549CD3`, purple `#B7618C`), moved to one lightness so they pass on the brand's warm whites. Three of the product bases fail 3:1 on Neutral 100 (yellow 1.76:1, green 2.38:1, blue 2.81:1). Red Mid is the product red to within a hair. Product and brand Mids differ until product adopts these.
- **Yellow Mid is a mustard, not chartreuse.** It sits near Chartreuse 700 in hue, so never place it next to a chartreuse accent or CTA in the same view, and never use chartreuse as a warning mark.
- **Two reds, two jobs.** Signal/Error `#FF7575` stays the form-error colour (invalid ring, error glyph, error message). Red Mid is the error mark in data: an error dot in a list, an error series in a chart, a failed segment in a meter.
- CSS: primitives `--ds-status-{hue}-mid`; component roles `--dsc-{success,warning,info,error}-mark`.

**How the red pair relates to Signal/Error.** `Signal/Error` `#FF7575` stays the error colour for the invalid-field ring, error glyphs and form error text (ruling above). The red pair is the *tint* around an error: the fill of an error tag, alert, toast disc or invalid table cell (`--dsc-error-bg` `#FFD6BD`, which replaces the off-palette `#FFE3E3` tint the component kit used), with `#4A1404` for text set inside that fill. `#FF7575` on `#FFD6BD` is only 1.94:1, so never set error text in Signal/Error on the peach fill; the ring or icon can stay Signal/Error.

## Dark Mode

**Light is the default colour set for every DocuSketch surface.** Every initial deploy renders light, and a surface never inherits dark from the visitor's OS. Dark is reached only by explicit opt-in — `data-theme="dark"` for a committed dark surface, `data-theme="auto"` for the rare surface that deliberately tracks `prefers-color-scheme`. A surface carrying no `data-theme` attribute is light, on every machine, on first paint. *(Immovable — 2026-09-01.)*

Choosing to expose dark at all is still a design decision, not a default: run the *scene sentence* first — "who uses this, where, under what ambient light, in what mood." Where dark mode is wanted, DocuSketch maintains a committed mapping that respects three principles:

1. **Only redefine the semantic layer.** Primitives are immutable; tokens swap. A component that binds to `DS/Token/Background/Default` never needs to know whether it is in light or dark mode.
2. **Depth from surface lightness, not shadow.** Shadows collapse to `none` in dark; elevation is signalled by stepping each surface lighter on the warm-neutral ramp (Neutral 700 → 650 → 800).
3. **Brand colour stays brand.** `Background/Brand` (Chartreuse 300) and `Brand/Olive` do not flip; text on chartreuse in dark is pinned to `Brand/Olive` because `Text/Primary` flips. The brand pair is identity, not chrome.

### Pairings

The full Light / Dark map for every DS semantic token. Light values are unchanged from the Tokens · Background / Text / Border sections above; Dark values are the committed counterparts.

| Token | Light primitive | Dark primitive | Notes |
|---|---|---|---|
| `DS/Token/Background/Brand` | Chartreuse 300 (`#E5DF00`) | **Chartreuse 300** | Brand fill — unchanged |
| `DS/Token/Background/Inverse` | Neutral 650 (`#39381B`) | Neutral 100 (`#F9F9F5`) | Flipped — "inverse" relative to current mode |
| `DS/Token/Background/Default` | Neutral 200 (`#F4F3EA`) | Neutral 650 (`#39381B`) | Card / inset surfaces — one step elevated above page |
| `DS/Token/Background/Subtle` | Neutral 100 (`#F9F9F5`) | Neutral 700 (`#1A1905`) | Page / section background — deepest surface in each mode |
| `DS/Token/Text/Primary` | Neutral 650 | Neutral 400 (`#C0BC90`) | Body, headlines. Dark primary is **warm sage**, not a stark off-white (9.14:1, AAA). Inverting the warm-neutral identity into a tech-flat white misreads the brand — the warm sage carries forward DocuSketch's identity into the dark theme. |
| `DS/Token/Text/Secondary` | Neutral 500 (`#807C5E`) | Neutral 500 (`#807C5E`) | 4.0:1 vs Neutral 700 — AA Large only, same caveat both modes |
| `DS/Token/Text/Strong` | Neutral 650 (`#39381B`) | Neutral 200 (`#F4F3EA`) | High-contrast body-supporting copy. 11.96:1 (light) / 15.91:1 (dark) — AAA both. Not an emphasis step above Primary on light; its purpose is holding high contrast when it flips to Neutral 200 in dark. |
| `DS/Token/Text/Muted` | Neutral 400 (`#C0BC90`) | Neutral 650 (`#39381B`) | Decorative only — ~1.5:1 in both modes. Same semantic across themes: tertiary, non-load-bearing copy. |
| `DS/Token/Text/Inverse` | Neutral 100 | Neutral 650 | Flipped |
| `DS/Token/Text/Accent` | Chartreuse 900 (`#2A2808`) | Chartreuse 300 (`#E5DF00`) | Chartreuse-family highlights NOT on a chartreuse fill: breadcrumb current, in-prose links, group headers, "Show more" expanders. |
| `DS/Token/Border/Strong` | Neutral 500 | Neutral 400 | Lifted off dark bg |
| `DS/Token/Border/Default` | Neutral 300 (`#DFDDC8`) | Neutral 800 (`#4A4830`) | Dark-mode-only primitive — see below |
| `DS/Token/Border/Subtle` | Neutral 200 | Neutral 650 | Barely lifted from page bg |

**Two new semantic tokens were added when canonizing dark mode** to replace patterns that were widely hardcoding primitives:

- `Text/Strong` — solves the *high-contrast body-supporting copy* role that hardcoded `Neutral 650` (11.96:1 vs white, AAA). In dark, Neutral 650 (`#39381B`) drops to 1.4:1 against the page bg (invisible). `Text/Strong` resolves to Neutral 200 in dark, preserving the role's high-contrast intent.
- `Text/Accent` — solves the *chartreuse-family highlight on theme bg* role that hardcoded `Text/OnBrand` (then Chartreuse 900, now retired) for breadcrumbs, in-prose links, and group titles. That token was specifically for text ON a chartreuse fill; on the dark page bg it lands at 1.19:1. `Text/Accent` resolves to Chartreuse 300 in dark (12.56:1, AAA).

### Surface elevation in dark

Three steps, each lighter than the last (impeccable principle: in dark mode, higher elevation reads lighter, not via shadow):

| Role | Primitive | Hex | Used as |
|---|---|---|---|
| Page (lowest) | Neutral 700 | `#1A1905` | Page / section background — `Background/Subtle` |
| Card (mid) | Neutral 650 | `#39381B` | Cards, insets, panels — `Background/Default` |
| Raised (highest) | Neutral 800 | `#4A4830` | Chips, hover surfaces, raised insets — bind directly or via `Border/Default` |

**Neutral 800 (`#4A4830`) is a Dark-mode-only primitive.** It is intentionally outside the light ramp (where it would have broken monotonicity between Neutral 650 → 700) and only resolves through the Dark token map — backing `Border/Default` and any "raised on card" surface.

### Shadows

Drop-shadow tokens (`--shadow-sm` … `--shadow-xl`, `--shadow-floating`) all resolve to `none` in dark mode. Light-mode CSS that uses these tokens degrades gracefully without further changes. Components that need to signal elevation in dark must use a lighter surface from the ramp above — never reintroduce a darker-than-bg shadow, which fails the "depth from lightness" principle.

### Component carve-outs

Brand artifacts that *are* the thing being demonstrated stay canonical regardless of theme:

- **Logo lockup stages** — the cream/white plates that frame the wordmark, DS° mark, Pill, and partner logos are part of the canonical presentation. They show light in both themes; the canonical SVG marks stay on their canonical surfaces.
- **Colour swatches** — show in their actual colour values in both themes. The page chrome around them adapts; the swatch fills don't.
- **Type specimens** — set on light by default; in a dark-mode brand surface, set the specimen plate to canonical light so the type renders as designed.
- **Camo pattern, gradient panels, sticky-CTA halo variants** — each is a designed surface with its own internal palette; theming would obscure the artifact.

### Theme switching

Where dark mode is exposed (e.g. the `/brand` reference site), the pattern is:

1. **Default is light** — no `@media` gate fires on a fresh visit. Declare `color-scheme: light` on `:root` so native controls, scrollbars and form widgets follow. A small inline `<script>` in `<head>` applies a *stored* user preference before paint to avoid FOUC; with nothing stored it does nothing and the page paints light.
2. **Toggle** — a two-segment Light / Dark pill in a stable surface position (sidebar bottom-left, footer, or settings panel). Once the user clicks, the choice is written to `localStorage` and to `:root[data-theme="light" | "dark"]` and wins from then on. Do **not** attach a `prefers-color-scheme` change listener: an OS change must never move a surface off light.
3. **`data-theme="auto"` is the only route to OS-following** — a surface opts in explicitly, typically alongside a three-segment Light / Dark / System control. Most surfaces do not, and shouldn't.

```css
:root { color-scheme: light; }              /* light is the floor */
:root[data-theme="dark"] { /* committed dark, or the toggle's choice */ }
@media (prefers-color-scheme: dark) {
  :root[data-theme="auto"] { /* OS-following — opt-in only */ }
}
```

The CSS pattern duplicates the dark mappings under both selectors so committed dark and opted-in OS dark behave identically. The `[data-theme="auto"]` gate is what keeps the OS preference out of every surface that didn't ask for it — a bare `:root` or `:root:not([data-theme])` inside that media query is the defect this rule exists to prevent.

### Canonical-surface scoping

Canonical artifacts (logo stages, colour swatches, the menu component, the don'ts gallery, etc.) keep a hardcoded light surface in both themes. Their **descendants must also keep light-mode text colours** — otherwise tokens that flip (`Text/Primary`, `Text/Strong`, `Text/Inverse`) will resolve to light values in dark mode and render invisible on the canonical light background.

The pattern: inside `:root[data-theme="dark"]`, scope the canonical containers and re-bind text tokens to their light-mode primitives:

```css
:root[data-theme="dark"] .menu-demo,
:root[data-theme="dark"] .menu-demo *,
:root[data-theme="dark"] .logo-dont,
:root[data-theme="dark"] .logo-dont *,
:root[data-theme="dark"] .colour-item,
:root[data-theme="dark"] .colour-item * {
  --ds-text-primary:    #39381B;   /* Neutral 650, the brand dark */
  --ds-text-secondary:  #807C5E;   /* Neutral 500 */
  --ds-text-strong:     #39381B;   /* Neutral 650 */
  --ds-text-inverse:    #F9F9F5;   /* Neutral 100 */
  --ds-text-muted:      #C0BC90;   /* Neutral 400 */
}
```

Duplicate the block under `@media (prefers-color-scheme: dark) { :root[data-theme="auto"] ... }` so surfaces that opted into OS-following get the same scoping. **The two copies must declare byte-identical values** — they drifted apart once (the committed-dark copy carried the retracted primary/strong inversion while the auto copy carried the authored values), which made a canonical surface render differently depending on whether dark was chosen or inherited. Background tokens are not re-bound — the canonical surface either hardcodes its own background (the typical case) or follows the theme via `Background/Default`.

### WCAG verification

Run a programmatic contrast walk on the dark variant of every brand surface. The pairings above hold AA at minimum across every page-chrome combination:

| Pair | Dark ratio | Threshold | Status |
|---|---|---|---|
| Text/Primary (Neutral 400) on Background/Subtle | 9.14:1 | 4.5:1 | AAA |
| Text/Primary on Background/Default | 6.17:1 | 4.5:1 | AA |
| Text/Strong (Neutral 200) on Background/Subtle | ~14:1 | 4.5:1 | AAA — escalation tier when Primary's warm sage needs reinforcement |
| Text/Secondary (Neutral 500) on Background/Subtle | 4.0:1 | 3:1 (AA Large) | AA Large only — captions / metadata; same caveat as light mode |
| Text/Muted on Background/Subtle | ~1.5:1 | (decorative) | Decorative only — non-load-bearing |
| Text/Primary (Olive) on Background/Brand | 8.48:1 | 4.5:1 | AAA |
| Text/Accent (Chartreuse 300) on Background/Subtle | 12.56:1 | 4.5:1 | AAA |
| Focus ring (Chartreuse 300) on page | 12.56:1 | 3:1 (non-text) | Pass |

**Why Primary is warm sage, not white-ish.** Inverting the warm-neutral identity (Neutral 700 in light → Neutral 100 in dark) would land at 16.79:1 — AAA, yes, but stark off-white on near-black reads as "tech-flat dark mode," not as DocuSketch. Neutral 400 (warm sage) carries the brand identity into the dark theme. It's still AAA (9.14:1). For places that genuinely need MORE contrast — surfaces where the warm sage doesn't have the gravity the role needs — escalate explicitly to `Text/Strong` (Neutral 200, ~14:1).

**Known mid-tone exceptions** (canon-documented, not bugs):
- Text on `Chartreuse 700` / `Neutral 500` swatches lands at ~3.67–4.0:1 — AA Large only. These swatches are documented as "fill / large-display tones, not backgrounds for body-size copy" (see the colour audit notes above). The `/brand` reference renders metadata on these swatches at the canon's recommended sizes; consumers should not put body copy on these fills.

## Radius Scale

Nine values only. Source-of-truth is the Figma Border Radius page (`JR35zTngKUblEKMD0myUyD`, node `119:5`). Any other value is an error — correct to the nearest step.

| Token | Value | Usage |
|---|---|---|
| `--radius-none` | `0` | Full-bleed images, flush banners |
| `--radius-sm` | `4` | Badges, chips, tight tags |
| `--radius-md` | `8` | Buttons, inputs, small cards |
| `--radius-lg` | `12` | Cards, panels, feature tiles |
| `--radius-xl` | `16` | Large cards, drawer panels |
| `--radius-2xl` | `24` | Hero image crops, media embeds |
| `--radius-3xl` | `30` | Nav pill hover, large CTA backgrounds |
| `--radius-4xl` | `32` | Section-level containers |
| `--radius-full` | `9999` | Avatars, pill buttons, circular icons — Figma auto-clamps to 50% of min dimension |

### Nested Radius Rule

**Outer R = Inner R + Padding.** When you nest a rounded element inside another, the outer radius must equal the inner radius plus the padding between them. Same radius on both creates pinched, misaligned corners.

| Outer | Padding | Inner | Context |
|---|---|---|---|
| `--radius-lg` (12px) | 4px | `--radius-md` (8px) | Chip inside a card |
| `--radius-xl` (16px) | 8px | `--radius-md` (8px) | Image inside a card |
| `--radius-2xl` (24px) | 8px | `--radius-xl` (16px) | Card inside section panel |
| `--radius-3xl` (30px) | 6px | `--radius-2xl` (24px) | Inset panel in hero |

### Edge fades on rounded containers

**An edge fade must reach zero opacity before the corner radius begins** (Chris Provins, 2026-09-28, Scope hero transcript strip). A fade that ends at the container edge still shows content at partial opacity where the straight edge turns into the curve, which reads as clipping, not fading. The mask stays fully transparent for the whole radius, then ramps to opaque. Radius `1rem` example:

```css
mask-image: linear-gradient(90deg,
  transparent 0, transparent 1rem,
  #000 3.5rem, #000 calc(100% - 3.5rem),
  transparent calc(100% - 1rem), transparent 100%);
```

Applied on the Scope hero ticker and the card-1 walk strip (radius 28px on the 1200 stage).

### Component Assignments

| Component | Token | Value |
|---|---|---|
| Badges, chips, tags | `--radius-sm` | 4px |
| Buttons, inputs, small cards | `--radius-md` | 8px |
| Cards, panels, feature tiles | `--radius-lg` | 12px |
| Large cards, drawer panels | `--radius-xl` | 16px |
| Hero image crops, media embeds | `--radius-2xl` | 24px |
| Nav pill hover, large CTA bg | `--radius-3xl` | 30px |
| Section-level containers | `--radius-4xl` | 32px |
| Avatars, pill buttons, circular icons | `--radius-full` | 9999px |
| Full-bleed images, flush banners | `--radius-none` | 0px |

**Pill rule**: any element whose `cornerRadius === height / 2` (i.e. a true pill or circle) must use `--radius-full`, not the computed value. This future-proofs resizing. If the element is rounded but NOT clamping to 50% (e.g. a 72px-tall element with a 24px radius), use the explicit value from the scale (24 → `--radius-2xl`), not `--radius-full`.

---

## Motion

Three durations, two easings, one rule. Apply via `var(--ds-motion-*)` in CSS, or by mapping to Figma's prototype interaction settings.

### Durations

| Token | Value | Usage |
|---|---|---|
| `DS/Motion/Duration/Micro` | `100ms` | Hover, focus, button-press — sub-perceptual; user shouldn't notice it as motion |
| `DS/Motion/Duration/Default` | `200ms` | Dropdowns, tooltips, popovers, accordions, tabs — the default for almost everything |
| `DS/Motion/Duration/Emphasis` | `400ms` | Modals, sheets, page transitions, hero reveals — moments the user should perceive as a transition |

> Three tiers for UI, plus one showcase tier below. Sub-50ms is below most users' perception of motion; 600ms+ feels sluggish on controls in a productivity tool. If something doesn't fit, that's a design question, not a token question.

**Showcase tier (ruling 2026-10-05).** Large content expansions on marketing pages (a card opening in an accordion, a photo moving from desaturated to colour) use the Resources page *Meet the Experts* motion, adopted on the New Home testimonials:

| Token (proposed) | Value | Usage |
|---|---|---|
| `DS/Motion/Duration/Showcase` | `600ms` + `Easing/Standard` | Card flex / width changes, open-state image framing, `filter` from inactive to active |
| Copy reveal inside the opened card | `opacity` `500ms`, `150ms` delay, `Easing/Standard` | Text arrives after the card has started opening |
| Inactive image treatment | `filter: grayscale(1) sepia(.2)` → none on hover / active | Peeks and slivers read as one warm-grey family |

Never for buttons, menus or anything a user waits on: those stay on Micro / Default / Emphasis. Reduced motion drops the duration to instant.

**Exception — continuous loops:** spinners and marquees have no perceived start or end. They use:

| Token | Value | Usage |
|---|---|---|
| `DS/Motion/Duration/Loop` | `1000ms` | Spinner full rotation; marquee strip cycle. Permitted **only** with `animation-iteration-count: infinite` — never as a one-shot transition duration. The 400ms ceiling on transitions still applies. |

### Easings

| Token | Curve | Usage |
|---|---|---|
| `DS/Motion/Easing/Out` | `cubic-bezier(0, 0, 0.2, 1)` | **Entrances** — anything appearing (tooltips, modals opening, menus revealing). Fast in, slow settle. |
| `DS/Motion/Easing/Standard` | `cubic-bezier(0.4, 0, 0.2, 1)` | **Persistent / two-way** — accordions, drags, slider changes, state-of-the-world updates. Symmetric ease. |
| `DS/Motion/Easing/In` | `cubic-bezier(0.42, 0, 1, 1)` | **Exits** — for cases where reversed `Out` doesn't read right. Used on the exit half of compound animations like the Sticky CTA arrow loop. |

> **Linear, constant speed** is permitted only for travel along a path (the workflow bracket's pulses run at 50px/s so they don't speed up at corners) and for continuous loops (marquees, progress fills). It is never a transition easing.

> No bounce, elastic, or overshoot curves. These read as AI-generated and don't match brand voice — impeccable explicitly flags them. Use `Easing/In` only for exits where reversed `Out` doesn't read right; default to reversed `Out` otherwise.

### Stagger

For revealing groups of elements (card grids, nav items, list entries), insert a between-item delay so the group reveals in sequence rather than en masse.

| Token | Value | Usage |
|---|---|---|
| `DS/Motion/Stagger/Tight` | `40ms` | Dense lists — table rows, ticker bars, character-level reveals |
| `DS/Motion/Stagger/Default` | `80ms` | The default — card grids, nav menus, settings lists |
| `DS/Motion/Stagger/Loose` | `160ms` | Emphasis sequences — hero feature bullets, marketing reveal moments |

**Compounding math:** total reveal time = duration + (count − 1) × stagger. A 200ms Default reveal across 5 items at Default stagger is 200 + 4×80 = 520ms.

**Cap:** never stagger more than ~8 items at once — the tail feels chaotic. For longer lists, batch into groups or use a single fade.

**Implementation:** set a `--index` custom property per child (inline `style="--index: 3"` or via JS), then:

```css
.item {
  opacity: 0;
  transform: translateY(8px);
  transition:
    opacity   var(--ds-motion-duration-default) var(--ds-motion-easing-out),
    transform var(--ds-motion-duration-default) var(--ds-motion-easing-out);
  transition-delay: calc(var(--index, 0) * var(--ds-motion-stagger-default));
}
.item.is-revealed {
  opacity: 1;
  transform: translateY(0);
}
```

### Permitted properties

Animate only **`transform`**, **`opacity`**, and **`filter`**. These stay on the compositor and don't trigger layout or paint.

**Forbidden** (cause layout thrash on every frame): `width`, `height`, `padding`, `margin`, `top`, `left`, `right`, `bottom`, `max-height`, `max-width`.

**Showcase exception (ruling 2026-10-05, New Home).** Two size changes may animate because the size change *is* the information, and both are contained: an accordion card's `flex-grow` (the testimonial row) and a progress dot's `width` (8px → 40px pill). Both run at Showcase 600ms / `Easing/Standard`, touch at most a handful of elements in one row, and reflow no surrounding text. Nothing else gets this exception: accordions that reveal text still use `grid-template-rows`, and reveals still use `transform`. brand-lint reports the dot's `width` as an error; mark that line `brand-lint: ignore` with a reason.

### Patterns

Nine canonical patterns. All animate only `transform`, `opacity`, `filter`, or `grid-template-rows` on the animated element.

| Pattern | Implementation | When to use |
|---|---|---|
| **Accordion / expand-collapse** | `display: grid` with `grid-template-rows: 0fr → 1fr`; content child has `overflow: hidden`. Default duration, Standard easing. **Disclosure indicator:** canonical `12 × 12` chevron SVG (`<path d="M2 4l4 4 4-4"/>`, `stroke="currentColor"`, `stroke-width="1.5"`, `stroke-linecap="round"`, `stroke-linejoin="round"`); rotates `180°` on open (Default duration, Standard easing). Never use an arrow (`↓`) or a `+/−` toggle. | Disclosure widgets, FAQ items, navigation submenus. |
| **Reveal** | `transform: scale(0.96 → 1)` + `opacity: 0 → 1`. Emphasis duration, Out easing. | Cards appearing in response to a direct user action. |
| **Slide-in / drawer** | `transform: translateX(24% → 0)` (or `translateY(...)`) paired with `opacity: 0 → 1`. Never animate `right` / `left` / `top` / `bottom`. Emphasis duration, Out easing. The shorter travel + fade is intentional — drawer should *arrive*, not *fly in*. Surface: pure white (`#ffffff` — currently `DS/Color/Base/White`; a dedicated `Background/Elevated` token is the right long-term home). Shadow: **`--shadow-lg`** per the [Shadows component map](#component-shadow-map). | Side panels, mobile sheets, notification toasts. |
| **Fade** | `opacity: 0 → 1`. Default duration, Out easing. | When no spatial change is needed — toasts, tooltips, simple state toggles. |
| **Scroll reveal** | `IntersectionObserver` adds `.is-revealed` to the element. CSS transitions `opacity: 0 → 1` + `transform: translateY(…)`. **Two tiers, by target size:** *section-level blocks* (containers, headline groups, hero children) use **Emphasis** duration, Out easing, **14px** travel, no base delay, and trigger at `threshold: 0.2` with `rootMargin: '0px 0px -10% 0px'` (earlier triggers finish below the fold and read as no motion); *small items inside an already-revealed block* (list rows, cards in a grid) may use Default duration and 8px. One-shot per element — unobserve after reveal. Stagger sits one tier above the grid default on marketing pages: Loose for hero children and card grids, Default for slides / FAQ rows (cap 8). `transition-delay: calc(var(--index) * <stagger>)`. Never put `overflow-x: hidden` on `body` — it makes body the scroll container and disables the continuous variant; use `overflow-x: clip` on `html`. The same applies to any ancestor of a scroll-driven element: clip with `clip-path: inset(0)` or `overflow: clip`, never `overflow: hidden`. Opt elements in from JS (e.g. `data-reveal`) so a no-JS page renders everything visible. **Continuous variant for card rows:** where a row of cards should glide in with the scroll instead of firing on a trigger, use a CSS scroll-driven animation (`animation-timeline: view(block)`, `animation-range: entry 5% entry 55%`, each card offset ~7% further), `linear` timing (the scroll is the easing), opacity + `translateY(20px → 0)`, wrapped in `@supports (animation-timeline: view())` with the observer reveal as the fallback and `animation: none` under reduced motion. **Do not tint the reveal.** A chartreuse-wash variant (multiply overlay clearing 1 → 0 instead of an opacity fade) was tried on the Capture page and rejected — see *Real-world context*. Reveals fade from transparent; the accent colour is not a motion device. | Section-level reveals on long pages. Marketing surfaces especially. See *Real-world context* below for where the two-tier rule and the continuous variant came from. |
| **Scroll-scrubbed media** | A muted, inline `<video>` whose `currentTime` is bound to the element's travel through the viewport: 0 when the slot's top reaches 85% of viewport height, the last frame when its bottom reaches 15%. **Linear mapping — the scroll is the easing**; a light per-frame lerp (factor ~0.2) only smooths seek bursts, never adds its own motion. Seek to `duration - 50ms` at most (Safari flags `ended` on the last frame). Gate the `requestAnimationFrame` loop with an `IntersectionObserver` (`rootMargin` ±25%) so it runs only near the viewport. `preload="auto"`, `playsinline`, no controls, `pointer-events: none`, a poster for the pre-load state, `role="img"` + `aria-label` on the wrapper and `aria-hidden` on the video. Reduced motion: seek once to a representative mid frame and stop. Never autoplay-loop the same clip as a fallback — the still frame is the fallback. Encode with dense keyframes (≤0.5s GOP) so seeks land on the frame the scroll asked for. | Product demonstrations that read better as a scrubbed 3D turn or process than as a looping clip: model rotations, before/after builds, step sequences. |
| **Modal entry** | Backdrop `opacity: 0 → 1` (Default, Out). Content `transform: scale(0.96 → 1)` + `opacity: 0 → 1` (Emphasis, Out). Closing reverses both at Default duration. Focus trap on open. | Confirmation dialogs, content sheets, image lightboxes. |
| **Spinner / loader** | `@keyframes spin { to { transform: rotate(360deg) } }`. Loop duration (1000ms), `linear` easing, `animation-iteration-count: infinite`. | "The system is working." Never a one-shot — that's a Fade. |
| **Page transition** | **Clean opacity fade** with a waterfall reveal — no blur. Outgoing `opacity: 1 → 0` → swap content → incoming `opacity: 0 → 1`. On enter, layer a **subtle waterfall**: the incoming view's top-level blocks rise `translateY(8px → 0)` + fade, staggered by the **tight** token (40ms) — first ~6 blocks only (the rest are below the fold). opacity + transform only (compositor-safe). Container fade runs Emphasis duration (≈800ms total out→in); the waterfall children run Default. Sequence leave → swap → enter via `animationend` so it tracks the duration token (and collapses to ~instant under reduced motion; the child waterfall is disabled outright there). | Full-page navigations on the **marketing site**, and section switches on the **/brand reference**. Product surfaces use native routing. |
| **Parallax** *(marketing only)* | `transform: translate3d(0, calc(var(--scroll-progress) * -15%), 0)`. Drive `--scroll-progress` via `IntersectionObserver` or CSS `scroll-timeline`. **Hero and footer only. Max 15% offset. Never in product UI. Disabled at `prefers-reduced-motion: reduce`.** | Visual depth on hero / footer of marketing pages. |
| **Scroll-triggered CTA** | Fixed-position element (typically `bottom-right`) with `transform: translateY(24% → 0)` + `opacity: 0 → 1`. Triggered by `IntersectionObserver` watching an "enter sentinel" and "exit sentinel" in the document flow — CTA appears when the user has scrolled past the enter sentinel and disappears once they've scrolled past the exit sentinel. **Entrance**: Emphasis duration (400ms), `Out` easing. **Exit**: Default duration (200ms), `Out` easing. Surface: `DS/Color/Chartreuse 300` + 1px `Brand/Black` border (matching the existing `Sticky CTA` component). Shadow: `--shadow-lg` per the Component Shadow Map. Radius: `8`. **Never use GSAP `elastic.*`, `bounce.*`, or `back.*` easings here — see Exclusions.** Use `prefers-reduced-motion: reduce` to skip the animation. | Persistent marketing CTAs ("Book A Demo", "Talk to an Expert") that should appear once the user has scrolled past the hero and disappear before the footer. |
| **Hover/leave loop animation** | A CSS `@keyframes` animation that fires once per hover-in AND once per hover-out (same animation, same direction — no reverse on mouse-out). Implemented via a JS-toggled class (`.is-animating`) with a reflow-forced restart so every state-transition re-triggers the animation. Per-keyframe `animation-timing-function`: `Easing/In` for the exit half (accelerate as you leave), `Easing/Out` for the entry half (decelerate as you settle). Triggered on `mouseenter` + `mouseleave` + `focus` + `blur` (a11y parity). Implemented via JS class-toggle because CSS pseudo-classes don't fire on un-hover. **Canonical use:** the Sticky CTA arrow slide-through-loop. | Any "this UI acknowledges both directions of interaction" affordance — primary CTAs, primary buttons with forward semantics, action menu items. |

### Exclusions

What we deliberately do *not* do. These are AI-slop tells, brand-voice mismatches, or solved problems we don't need libraries for. If a designer requests one, the answer is no — it's a system extension request, not a one-off.

- **No smooth-scroll libraries** (Lenis, Locomotive, ScrollSmoother). DocuSketch sites use native browser scroll. Smooth-scroll libs intercept user input, can feel laggy on trackpads, break anchor-link expectations, and disable back-button scroll restoration.
- **No custom Bézier curves beyond `Out` and `Standard`** — no `CustomEase`-style per-component invention.
- **No bounce, elastic, overshoot, or spring curves** (e.g. `cubic-bezier(0.68, -0.55, 0.27, 1.55)`). impeccable explicitly flags these as AI-slop.
- **No cursor-following effects** — magnetic buttons, custom cursors, mouse-tracked highlights, gooey blob cursors. Reads as portfolio-playful, not DocuSketch-professional.
- **No text scramble, typewriter, or decoder-reveal** effects on body or heading copy. AI-slop tells.
- **No atmospheric (>400ms) one-shot transitions.** The 400ms ceiling is intentional. The only legal duration above 400ms is `Loop/Standard` for continuous loops.
- **No parallax in product UI.** Marketing only, restricted to hero and footer — see Patterns.

### Reduced motion

Respect `prefers-reduced-motion: reduce` per WCAG 2.3.3. When set, override all duration tokens to `1ms` and easing tokens to `linear`.

```css
:root {
  --ds-motion-duration-micro: 100ms;
  --ds-motion-duration-default: 200ms;
  --ds-motion-duration-emphasis: 400ms;
  --ds-motion-duration-loop: 1000ms;

  --ds-motion-easing-out: cubic-bezier(0, 0, 0.2, 1);
  --ds-motion-easing-standard: cubic-bezier(0.4, 0, 0.2, 1);

  --ds-motion-stagger-tight: 40ms;
  --ds-motion-stagger-default: 80ms;
  --ds-motion-stagger-loose: 160ms;
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --ds-motion-duration-micro: 1ms;
    --ds-motion-duration-default: 1ms;
    --ds-motion-duration-emphasis: 1ms;
    /* Loop is not collapsed to 1ms — that would still spin, just imperceptibly.
       Instead, components using Loop must explicitly disable their @keyframes
       animation under prefers-reduced-motion (e.g. animation: none; replace
       with static "Loading…" text). */

    --ds-motion-easing-out: linear;
    --ds-motion-easing-standard: linear;

    --ds-motion-stagger-tight: 0ms;
    --ds-motion-stagger-default: 0ms;
    --ds-motion-stagger-loose: 0ms;
  }
}
```

### Decorative motion (ruling 2026-10-05)

Motion must explain a change or a flow. On New Home, animated outcome illustrations went through four rounds (radar sweep, scan lines, point clouds, flowing threads), each made subtler, and were then removed entirely: **illustrations are static**. What stayed is motion that carries information: the workflow bracket showing data flowing into the job record, the testimonial autoplay progress, the DS1 camera scroll-scrub, the camo drift behind the closing CTA (paused off screen), and the client-logo marquee at half speed (`0.5`). Each one has a reduced-motion branch that shows the resting frame.

### Real-world context

Field notes from applying this section to shipped pages. Each entry names the page, what the canonical rule produced, and what changed — so the rule evolves from evidence, not taste.

**Capture platform page (Webflow draft `capture-software-v2`, 2026-09-16, Chris Provins).**

- **Scroll reveal felt too fast.** The canonical Default 200ms / 8px reveal, triggered at 5% visibility, read as a flicker on full-width containers (80vh tool sections, four-up card grids). First pass: Emphasis (400ms), 16px, Out, `threshold: 0.15`, `rootMargin: -12%` bottom. Second pass, same day: 16px travel read as too much movement at that duration, and blocks still arrived too eagerly — travel cut to **10px**, a **Micro (100ms) base delay** added before every reveal, and stagger moved one tier up (grids Loose, rows Default). Third pass (2026-09-17): the base delay and the late trigger made the page feel start/stop — content waited, then moved. Base delay removed, stagger back to Default, trigger brought forward to 5% in view so the reveal overlaps the scroll. For the four how-it-works cards specifically, the one-shot reveal was replaced with a **scroll-linked rise** (`animation-timeline: view()`, per-card range offsets) so they glide in continuously with the scroll; observer reveal remains the fallback. Fourth pass: with the trigger at 5% the rise completed below the fold and read as *no* motion — a reveal must fire where the eye is, so the trigger sits at **20% in view with a 10% bottom margin** and travel is **14px**. Also found: `overflow-x: hidden` on `body` (a common template guard) turns body into the scroll container and silently disables `animation-timeline: view()` and `position: sticky` — use `overflow-x: clip` on `html` only. Fifth pass, tried and rolled back the same day: a *fade from chartreuse* — blocks at full opacity with a Chartreuse 300 multiply overlay clearing as they rise. Chris's verdict on seeing it: "no bueno." The tint read as a colour flash across photos and cream surfaces rather than a brand moment. Reverted to the plain opacity fade; the pattern row now says not to tint reveals. Kept here so nobody re-proposes it. This is now the section-level tier plus the continuous variant in the Scroll reveal pattern above; Default / 8px remains for small items.
- **Exclusions executed, not just flagged.** The page's inherited template shipped Lenis smooth scroll, a mouse-tracking 3D hero tilt, a cursor-tilt on cards, and GSAP scroll animations. All removed; native scroll restored. Accordion moved to `grid-template-rows` at Default / Standard; carousel transition set to Default.
- **Scroll-triggered CTA reference implementation.** Sticky "Book a Demo" now follows the pattern row exactly: `IntersectionObserver` on an enter sentinel (hero button) and exit sentinel (CTA band button), `translateY(24% → 0)` + opacity, Emphasis / Out on entrance, Default / Out on exit, CSS-variable driven, no GSAP. Use this page as the working example when remediating the homepage violation below.
- **Directional testimonial slide.** Layered on Webflow's native Cross animation: outgoing quote exits at Default with `Easing/In`, incoming arrives at Emphasis with `Out`, 24px travel — the drawer pattern's "arrive, don't fly in" ratio applied to a carousel. Class-toggled CSS transitions, no library.
- **Click-to-load embed.** Third-party 360° player is a poster + pill button until clicked (no iframe on load, no scroll capture). Poster hover: scale 1.02 and colour inversion at Default / Out; the loaded frame fades in at Default.

- **Pre-footer CTA on DS/Pattern/Camo with footer-zone parallax.** The inherited template used a flat exported camo JPEG as the section background. Replaced with the canonical generator output (`generateCamoSvg(2026, { width: 1920, height: 720 })`) inlined as an absolutely positioned layer translating −15% → +15% across the section's pass through the viewport. **Tuning lever:** the 15% cap is on the translate, which is relative to the *layer's* height — so the visible travel is set by how much taller the layer is than its section. 130% read as too subtle over a blurred pattern; 200% (50% overshoot each side, ≈ ±30% of section height in pixels) is where it registers. Keep the translate at 15%; change the layer height. — the Parallax pattern's cap, applied to the pre-footer band (the "footer" zone). Scroll-driven (`animation-timeline: view(block)`, `animation-range: cover`) where supported, `requestAnimationFrame` fallback elsewhere so the effect is universal, switched off under reduced motion. Content keeps the existing chartreuse card, so the chartreuse-on-camo type rule is honoured by the card rather than by recolouring type over the pattern. **Gotcha, second sighting:** clipping the section with `overflow: hidden` silently killed the parallax — any `overflow` other than `visible`/`clip` makes that element a scroll container, and `animation-timeline: view()` binds to the nearest one, which then never scrolls. Clip parallax layers with `clip-path: inset(0)` (or `overflow: clip`), never `overflow: hidden`. Same root cause as the body-overflow note above; it applies at every level of nesting.

- **Stats band with a chartreuse glow (2026-09-18).** The business-impact band moved from three free-standing tiles to a Stripe-style proof row: centred condensed eyebrow + two-line H2, then four stats framed not by strokes but by registration marks: + crosshairs at the row's four corners (0.8125rem, half the page's 1.63rem `plus-icon`) and the feature cards' 5×1px dash at the three column boundaries on the top and bottom edges — the same two-mark vocabulary the tool containers use (crosshair at the perimeter, dash between columns). L-shaped crop marks at the corners and crosshairs between the columns were both tried and dropped, 1px, Olive for the crosshairs and Brand/Black for the dashes as on the cards. A first pass with Neutral 300 hairlines top and bottom was replaced at Chris's request — the marks tie the band to the tool containers' tick language instead of introducing a rule, each with a one-line context sentence in body size (not caps). All copy in the band — eyebrow, H2, values, context — is set in Olive `#39381B` (Text/Strong); a first pass with the lead stat in Text/Primary and the rest in Text/Secondary was rejected, so hierarchy in this band comes from size alone. The space above the eyebrow equals the glow space below the row (12rem each). The glow beneath is a translation of the Figma frame `guoAdcJQH5m7hrlyH62OGZ` node `25:189`, which stacks three ellipses (Chartreuse 300 core, `#FFFA37` at 75% then 25%) under a 25% white frosted layer. In CSS that became one `radial-gradient(ellipse at 50% 100%, #E5DF00 0%, rgba(255,250,55,.75) 32%, rgba(255,250,55,.25) 62%, transparent 100%)` sized `closest-side` so the top and side edges resolve to transparent, on an absolutely positioned layer the exact width of the content container, anchored below the row, `filter: blur(48px)`, opacity 0.9. The wrapper carries `overflow: clip` + `isolation: isolate`, so the glow is bounded to the same width as the feature cards beneath it and runs flush to their top edge (the divider that used to sit between the two sections was removed and the section's bottom padding is zero; the glow itself is the spacing). First pass used a 140%-wide ellipse that bled past the cards' edges and stopped short of them — Chris rejected both. **Rules that fell out:** the glow is a surface, not a reveal — it does not animate, and it never sits behind text that has to stay readable at Text/Secondary contrast, which is why the row sits above it on `z-index` and the ellipse rises from below the hairline. Stat copy is centre-aligned in each column. **Hover state per stat:** the copy deepens from Olive to Brand/Black (by inheritance from the tile) and the tile lifts 2px — colour and transform at Default 200ms / Standard easing. No fill, no shadow, no scale; a Neutral 100 tile fill was tried and dropped as too heavy. The same hover now runs on the how-it-works cards (card copy set to Olive at rest, children inherit, card darkens to Brand/Black and lifts 2px). **Gotcha that fell out:** a hover `transform` on an element that is also a reveal target is silently overridden — `.is-revealed { transform: none }` outranks it by source order, and a scroll-driven keyframe with `animation-fill-mode: both` outranks it always. The fix is to move the reveal and glide travel onto the individual `translate` property and keep `transform` for hover; the two compose instead of competing. Reveal and hover should never share a property. **Second gotcha, same root:** the reveal rule's `transition` list replaces the element's class-level `transition` by cascade order, so a revealed tile's hover snapped with no easing until the reveal rule also listed `color` and `transform` at Default / Standard (with `0s` delay for those two, so the stagger delay does not leak into hover). Any page-level reveal rule must carry the hover transitions of every element it can be applied to.
- **Registration marks — three tokens, three jobs (2026-09-18, Chris Provins).** The Capture page settled the vocabulary for the crosshair family, which had drifted into three sizes used interchangeably. Proposed token names, pending Figma:
  - `Mark/Crosshair-L` — the 1.63rem plus (`plus-icon` asset, `☘️ Element / Crosshair` on the New Home). **The default.** Section dividers (the rows above and below the logo marquee), frames around whole blocks (the testimonial block), and the break between the workflow module and the FAQ.
  - `Mark/Crosshair-S` — the 0.8125rem, 1px, Olive plus. Reserved for the perimeter of data rows, where the large mark would out-weigh the numbers. On the page: the four corners of the business-impact stat row, and nowhere else.
  - `Mark/Dash` — the 5×1px dash in the brand dark (Olive since 2026-10-05; Brand/Black before). Column boundaries inside a framed row: between the stat columns on the business-impact band and at the tick rows of the tool containers. Dashes sit at the top and bottom ends of each column boundary, aligned with the row's top and bottom edges, never centred vertically on the content (Chris Provins, 2026-10-05).
  Rule of thumb: L frames sections, S frames data, Dash divides columns. Where a glow already carries the transition between two blocks, the crosshair row on that edge comes off — the Capture page's testimonial block keeps only its bottom row of Crosshair-L because the feature cards' glow tail flows into its top. **Glow as bookends (2026-09-18):** the stats-band glow that rises into the first tool container is mirrored below the last one — same recipe (24rem layer, 8rem overhang, blur 48px, opacity 0.9) with the ellipse anchored at `50% 0%` instead of `50% 100%`, inside a 12rem clipped tail. Entering and leaving the product block through the same light reads as one framed chapter rather than two effects. Never mix two crosshair sizes on the same block. A first pass framed the testimonials with Crosshair-S; Chris moved it to Crosshair-L because the block is a section, not a data row.
- **Workflow value panel on animated Camo (2026-09-18).** The cross-page "Capture / Scope / Estimate" module became a single rounded panel (`--radius-2xl`, 24) on `DS/Pattern/Camo` in its Animate mode — the same `generateCamoSvg` output with the kit's `pattern-blob` drift (22s ease-in-out, 2.2s stagger, off under reduced motion), a new seed per surface so two camo panels on one page never match. Left: eyebrow + one-sentence statement. Right: three hairline-divided rows, title + ↗ (or a "You are here" pill on the current step) + one-line description drawn from the positioning doc's workflow pillars. **Type on the panel is Chartreuse 200 on the standard generator camo** (seed 2027; strokes at 50% alpha, secondary copy at 85%, outlined "You are here" pill, icon boxes filled Chartreuse 200 with Brand/Black glyphs). Chris tried Brand/Black on a bright seed, light neutrals on a dark-base variant and Chartreuse 300 on both bases in one session; Chartreuse 200 was the pairing that separated type from pattern without leaving the family, and it became the rule. An earlier pass used Olive on Neutral 100 cards and read as three product tiles rather than one platform statement. Row states: hover nudges the row `translateX(4px)` (kit list-row rule), turns its top hairline from 35% to solid Brand/Black, brings the description from 80% to full opacity and sends the arrow a further 6px; `:active` settles the row at 2px and the arrow at 3px; `:focus-visible` is a 2px Brand/Black outline at 4px offset. The current step has no link and carries a filled pill — Brand/Black with Chartreuse 300 text, the canonical primary fill — instead of an arrow. The arrow is the same 16px `→` SVG the testimonial slider arrows use, in `currentColor`, so one glyph does all directional work on the page. All at Default 200ms / Standard. Standing rule for this module: a Camo panel is a statement surface — one idea, one palette, no illustrations, no imagery fighting the pattern.
- **Scroll-entry reveals removed (2026-09-21, Chris Provins).** After five passes of tuning, the Capture page shipped without any on-scroll entry animation. Chris's call: the reveals and the continuous card glide added script, observers and per-element transitions without earning their keep, and the page already carried enough motion in the animated camo, the hover states and the marquee. What stayed is motion that either responds to the reader (hover, sticky CTA) or lives inside one bounded surface (camo drift). Field lesson for the Scroll reveal pattern: it is optional, not default — reach for it when a section needs to be *discovered*, not to decorate content that is already legible. **Sticky CTA exit rule, refined:** hide the floating button only while the block carrying the in-page CTA is on screen, and let it return below that block, so the FAQ and footer are never left without a path to convert.
- **Testimonial frame (2026-09-18).** The testimonial block is framed the same way as the stats row — six half-size crosshairs, three on the top edge and three on the bottom (corners and centre) — so every framed block on the page shares one mark. Watch for legacy classes: the old tile class kept a 1px bottom border that drew a stroke under every stat until it was stripped. Clip it with `overflow: clip` on the section (never `overflow: hidden`, which would kill the scroll-driven timelines elsewhere on the page). `#FFFA37` is a highlight mix from the Figma frame, not a token; use it only inside a glow that resolves to Chartreuse 300 at its core.
- **Product feature cards on mobile (2026-09-21, Chris Provins).** Below 768px the four tool containers were first hidden behind an accordion of their detail rows (no headline, no icon, no media). Chris rejected it the same day: the cards carry the product story and the click-to-load 360º tour is the page's one interactive proof, so hiding them on the device most prospects arrive on was the wrong trade. The accordion came out and the how-it-works grid is hidden at that breakpoint instead (its four steps duplicate what the cards say), so the page goes hero → logos → cards → stats. Each card restacks in one column in source order: tool block (2.25rem icon + pill) with the top-right L crop on the same row, the context label beneath it on its own row, the headline at 1.5rem / 1.25, media full-width at **4:3** (not the desktop 16:10 — the tour player needs the height to be usable with a thumb), then the detail rows one per line at 1.25rem gaps, then the bottom row reduced to its two L crops. Card padding 1.5rem / 1.25rem, 1rem between cards; no minimum height. **Rules that fell out:** the Dash token comes off at a breakpoint where the columns it divides no longer exist — the mid row and the bottom middle dash are desktop-only, the corner L crops stay because they frame the card, not a column. Every override lives on the small breakpoint of the existing classes (plus a no-property `is-desktop` combo on the dashes); the tablet and desktop cascades are untouched. Gotcha: the earlier hide had landed on the shared base class `.section-3`, not the page-scoped combo — a breakpoint override on a template class leaks to every page that uses it; hide via a page-scoped combo or the page's own embed. The same template class also ships `display: none` on Webflow's portrait tier (≤479px), which is where a real phone lands, so removing the landscape override alone still left the cards invisible on a phone. Check every breakpoint below the one you edited, and re-show with a page-scoped combo (`.section-3.cap-light { display: block }` at portrait) rather than editing the shared class.
- **Enhanced Sketch media is a scroll-scrubbed video (2026-09-21, Chris Provins).** The card's placeholder became a 9s dollhouse rotation (1220×698 screen recording, hosted as a Webflow asset — the asset store accepts `video/mp4` through the same S3 hand-off as images) whose playback is driven by scroll, per the *Scroll-scrubbed media* row above: one rotation across the slot's travel through the viewport, poster first frame, mid frame under reduced motion. It sits in the same `.cap-tool__media` slot as the click-to-load tour and the photographs, `object-fit: cover` at 16:10 on desktop and 4:3 on mobile, on a white field because the recording's background is white. **Rules that fell out:** the mapping runs across the whole visible travel rather than a pinned range, so the model turns as the card scrolls by instead of holding the page; and the clip is the source of truth for progress — no easing curve is layered on top, only a lerp that damps seek bursts. Gotcha: the source is a screen recording with a 1px window edge and a view-cube widget baked in; a 3% scale hides the edge, the widget stays until a clean export replaces the clip. Prefer exports from the tool over recordings.
- **Mobile logo marquee: two rows, no JavaScript (2026-09-21, Chris Provins).** The ≤767px marquee splits the client logos into two rows scrolling in opposite directions, 30% larger than desktop. The first build cloned the collection list into a second row with a script inside the embed, and Chris could not find it: **the Designer canvas never executes embed scripts**, so anything a script creates is invisible while the page is being designed and only appears on publish. The second row is now static markup — a hand copy of the Marquee-Logos collection, doubled for the seamless loop and started mid-list so it never mirrors row 1 — hidden above 767px, with the section switched to `flex-direction: column` at that breakpoint. **Rule:** on a Webflow page anything structural a designer needs to see in the canvas must exist as elements or static markup; scripts may only enhance (clone for a seamless loop, bind scroll to playback), never create what the layout depends on. The cost is that the static row has to be updated by hand when the collection changes; a note in the embed says so.
- **Closing CTA block replaces the sticky button at page end (2026-09-21, Chris Provins).** Reference: Mews' "Ready for impact?" pre-footer — a display headline, three numbered one-line steps, one button. The DS translation sits between the FAQ's Crosshair-L row and the footer, on top of the pre-footer glow (the glow layer stays at `z-index: -1`, the block at `1`, so the light reads as the surface the invitation sits on rather than an effect behind the footer). Headline in IBM Plex Sans Condensed 500, uppercase, 5.5rem / 0.95 (4rem tablet, 2.75rem phone), Olive — the condensed cut is the brand's display voice for a one-line shout; Plex Sans stays for sentences. Steps are the onboarding promise from the existing site (Free consultation · On-site training · Ongoing support): a 1.75rem Olive disc with a Neutral 100 condensed numeral, then a bold lead (`Lead.`) with the sentence beneath it, three equal columns on desktop (one row of inline steps, Mews-style, does not fit three restoration-length sentences at 1440px), a single left-aligned column on phones. Button is the `Elements / CTA` component, never a re-drawn pill. **Rule that fell out:** the sticky CTA retires for good once this block enters the viewport (or has been scrolled past) — two Book-a-Demo buttons on screen at once is a conversion smell, and the page end should be a place to stop, not something a floating button hovers over. Together with the workflow-panel rule this gives the sticky button three states: hidden over any block that carries its own CTA, visible elsewhere after the hero, retired at the closing block.

### Known violations to remediate

- `https://www.docusketch.com/solutions/360-degree-camera` — `transition: max-height` on at least one element (flagged by `impeccable detect` 2026-05-21). Migrate to the `grid-template-rows` accordion pattern above.
- `https://www.docusketch.com/` — "Book A Demo" scroll-triggered CTA in the inline `stickyDemoInit` script uses **GSAP `elastic.out(0.9, 0.4)` easing** for entrance, violating BP #16 / Motion Exclusions (no elastic). Other deviations: travel is `yPercent: 120` (should be `24%` per the Scroll-triggered CTA pattern), entrance duration `0.7s` and exit `0.35s` (should be `400ms` / `200ms` per the canonical scale), and exit easing `power2.in` (should be `--ds-motion-easing-out`). Replace the GSAP call with `IntersectionObserver` + CSS variable–driven transitions — sample implementation in the Scroll-triggered CTA pattern row.

---

## Shadows

> **Figma source:** [`Shadows` page](https://www.figma.com/design/JR35zTngKUblEKMD0myUyD/Brand-Design-Kit?node-id=119-6) (node `119:6`). Edit Figma to change canonical values; this section mirrors the page verbatim. Last verified: 2026-05-21.

Used sparingly. Always `rgba(0,0,0,α)` — never coloured shadows. Hover steps up one level.

### Shadow scale

Seven levels: six concentric "lift" shadows (light from above) plus one "floating" composite shadow (light from upper-right, designed for elements that hover *over* page content rather than sitting *on* it). Each lift-shadow level is a step up from the last.

| Token | Value | Usage |
|---|---|---|
| `--shadow-none` | `none` | Flat buttons, badges, static cards |
| `--shadow-xs` | `0 1px 2px rgba(0,0,0,0.08)` | Input fields, tight cards |
| `--shadow-sm` | `0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.08)` | Cards, popovers, dropdowns |
| `--shadow-md` | `0 4px 6px rgba(0,0,0,0.10), 0 2px 4px rgba(0,0,0,0.08)` | Feature cards, hover lift |
| `--shadow-lg` | `0 10px 15px rgba(0,0,0,0.10), 0 4px 6px rgba(0,0,0,0.06)` | Modals, drawers, floating nav |
| `--shadow-xl` | `0 20px 25px rgba(0,0,0,0.10), 0 10px 10px rgba(0,0,0,0.06)` | Hero cards, sticky headers |
| `--shadow-floating` | `-1px 3px 3px rgba(0,0,0,0.03), -3px 11px 6px rgba(0,0,0,0.03), -7px 26px 8px rgba(0,0,0,0.02)` | **Sticky CTAs, floating action buttons, persistent floating chips.** Subtle multi-layer shadow with negative X offset (light from upper-right), conveying that the element is *drifting above* the page rather than *pressed up* from it. |

### Usage rules

| Rule | Detail |
|---|---|
| ✅ Hover: step up | On hover, apply the next shadow level up. Card uses `--shadow-sm` → hover applies `--shadow-md`. |
| ✅ rgba only | Always use `rgba(0,0,0,α)`. Coloured shadows are never used — they fail across different background colours. |
| ✅ Buttons: no shadow | Button and card components have no shadow by default. `--shadow-none` is the base state. |
| ✅ Floating elements only | Dropdowns, select panels, modals, and sticky nav use shadows. Static layout elements do not. |
| ❌ Coloured shadows | Never use shadow colours other than black with alpha. Not even brand chartreuse or eucalyptus. |
| ❌ Shadow on shadow | Never stack multiple shadow levels on a single element. Use one level appropriate to the elevation. |

### Component shadow map

Default and hover shadow for each component type. Use these mappings instead of inventing per-component shadows.

| Component | Default | Hover |
|---|---|---|
| Flat buttons, badges | `--shadow-none` | `--shadow-none` |
| Static cards | `--shadow-none` | `--shadow-sm` |
| Input fields | `--shadow-xs` | `--shadow-sm` |
| Cards, popovers | `--shadow-sm` | `--shadow-md` |
| Feature cards | `--shadow-md` | `--shadow-lg` |
| Dropdowns, select panels | `0 1px 3px rgba(0,0,0,0.20)` | — |
| **Modals, drawers** | **`--shadow-lg`** | — |
| Floating nav, sticky headers | `--shadow-xl` | — |
| **Sticky CTA, floating chips** | **`--shadow-floating`** | — |

### CSS implementation

```css
:root {
  --shadow-none: none;
  --shadow-xs: 0 1px 2px rgba(0,0,0,0.08);
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.08);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.10), 0 2px 4px rgba(0,0,0,0.08);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.10), 0 4px 6px rgba(0,0,0,0.06);
  --shadow-xl: 0 20px 25px rgba(0,0,0,0.10), 0 10px 10px rgba(0,0,0,0.06);
  --shadow-floating:
    -1px 3px 3px rgba(0,0,0,0.03),
    -3px 11px 6px rgba(0,0,0,0.03),
    -7px 26px 8px rgba(0,0,0,0.02);
}
```

---

## Type Styles

All type styles are defined in Brand Design Kit (`JR35zTngKUblEKMD0myUyD`). Apply via `node.textStyleId` using the style key. In scripts working within a different file, use `figma.importStyleByKeyAsync(key)` first.

Letter-spacing is part of the type style — set it on the style object, do not override per node (per Best Practice #13). Values marked *(verify)* should be confirmed against the canonical Figma style.

| Style name | Key | Family | Weight | Size | LH | Tracking | Notes |
|---|---|---|---|---|---|---|---|
| `DS/Type/Display` | `3a75c95299f122b3b086669b4b55fe9f9e57ce86` | IBM Plex Sans | Medium | 72 | 110% | `-0.04em` | Hero display headline |
| `DS/Type/H1` | `702f2a8b919b9a6709be73a5e3ccddc2cbbf9e03` | IBM Plex Sans | Medium | 64 | 110% | `-0.04em` | Page title |
| `DS/Type/H2 Large` | `bec7d1e0a0085ae9a0061ca25e7e66aaab2ba99c` | IBM Plex Sans | Medium | 54 | 110% | `-0.04em` | Large section heading |
| `DS/Type/H2` | `08bdd97ec2f346dc2387b8051f544ef8e9fd765c` | IBM Plex Sans | Medium | 36 | 110% | `-0.03em` *(verify)* | Section heading |
| `DS/Type/H3` | `2645d388038ba4ce0f22edaa3564e356766140eb` | IBM Plex Sans | Medium | 28 | 110% | `-0.02em` *(verify)* | Sub-section heading |
| `DS/Type/H4` | `f675dfcc94a08e379ce2e9e1afe1f448c0d2bdde` | IBM Plex Sans | Medium | 24 | 100% | `-0.04em` | Card heading. |
| `DS/Type/Body XL` | `12b0f0afadeca79ea17fda5611083986416cf4ae` | IBM Plex Sans | Regular | 20 | 150% | `0` | Large body / intro paragraph |
| `DS/Type/Body LG` | `f59d671d93870260c6eb051c3989afc1cb914c69` | IBM Plex Sans | Regular | 18 | 150% | `0` | Body large |
| `DS/Type/Body` | `650dc559a21dfc3c6690e3106e5a4d992144e78d` | IBM Plex Sans | Regular | 16 | 150% | `0` | Default body copy |
| `DS/Type/Body SM` | `582baf79f949ab128001a9ba6b8aa181c132b262` | IBM Plex Sans | Regular | 14 | 150% | `0` | Small body / supporting copy |
| `DS/Type/Button` | `6d12e030fd57b0d53c6b79cdf3277fcbc37c5a4c` | IBM Plex Sans | Regular | 18 | 100% | `-0.01em` *(verify)* | CTA button text |
| `DS/Type/Nav` | `a0c44689d935eae742c01cbbb4ff64e409dc3865` | IBM Plex Sans Condensed | Regular | 16 | 100% | `0` | Navigation items |
| `DS/Type/Condensed Label` | `52a12981aaa70f9b1201c8b8c8cbcc51d9976512` | IBM Plex Sans Condensed | Medium | 18 | 100% | `0` | Compact label / tag |
| `DS/Type/Caption` | `fa647c4dc406168ce570791c185e7f92a4f8fa4f` | IBM Plex Sans | Regular | 12 | 150% | `0` | Fine print, footnotes |
| `DS/Type/Overline` | `fdc61f952a620bf320986e341ed689cc8bfa5c4a` | IBM Plex Mono | Regular | 11 | 100% | `+0.08em` | **Primary label style.** ALL CAPS. Eyebrows, stat headers, section markers, column labels |
| `DS/Type/Quote/Mono` | `51edfbaf43855b980e869aac20fb108260b7ea79` | IBM Plex Mono | Regular | 38 | 110% | `-0.01em` *(verify)* | Pull quote — mono style. Use at Desktop (1580px) and Tablet (768px) breakpoints. |
| `DS/Type/Quote/Mono SM` | `f10a2d20aeb3fa82b66f7be12b93b63991956057` | IBM Plex Mono | Regular | 20 | 130% | `-0.01em` | Pull quote — mono mobile. Use instead of Quote/Mono on 375px breakpoint |
| `DS/Type/Code/SM` | `e78f4975e84c8ab8a8e371ac21fc57151fdc3dc4` | IBM Plex Mono | Regular | 12 | 150% | `0` | Inline code, small data |
| `DS/Type/Code/MD` | `706c886590a638336a4673c3146310fd0bb22321` | IBM Plex Mono | Regular | 14 | 150% | `0` | Code blocks, technical data |

> **DS/Type/Label has been deleted.** `DS/Type/Overline` now supersedes it as the standard label style for all eyebrows, stat headers, column labels, and section markers. Any file that previously used Label should be rebound to Overline via `figma.importStyleByKeyAsync('fdc61f952a620bf320986e341ed689cc8bfa5c4a')`.

**Web eyebrow — ruling 2026-10-07.** A page or section eyebrow on the web is the kit's `Document / Page Eyebrow` (Brand Design Kit `JR35zTngKUblEKMD0myUyD`, set `1183:221`, `Surface=Light` `1183:219`) at web size: `DS/Type/Overline` text (Plex Mono 11, line-height 1, `+0.08em`, caps, `ss04` for the plain zero) in Olive, inside a 1px Olive inset hairline (`box-shadow: inset 0 0 0 1px`), `--radius-sm` (4), padding 8px vertical / 12px horizontal, no fill. On an Olive surface use `Surface=Dark` (Chartreuse stroke and text). This settles the Overline vs Plex Sans Condensed 14 question (AUDIT-2026-10-05 #4): no Condensed eyebrows on the web. Condensed Medium caps stays for Editorial caps labels, product pills and in-control column heads. First built on the Remote Estimating Ebook pages (2026-10-07, embed "Brand v2 CSS (ebook)").

### Print parallel ramp — `DS/Type/Print/*`

Print collateral designed at **PDF-native scale** (1 px = 1 pt, page size `612 × 792` = 8.5 × 11 in) renders type at the same point size on screen as it will print. Web/digital styles (the `DS/Type/*` ramp above) are sized for screen reading and look oversized at print scale; `DS/Type/Print/*` is a parallel ramp at ~60–65% scale that lands as print-realistic body / headline sizing.

**Decision rule:**

- **Web scale page** (96 dpi: `816 × 1056`, `840 × 1080`, web/digital surfaces) → use `DS/Type/*`. Export to PDF at 75% to land at letter.
- **PDF-native scale page** (72 dpi: `612 × 792`, `552 × 840`, print-native templates) → use `DS/Type/Print/*`. No export scaling; what you see is what prints.

Line-heights and letter-spacing are preserved across both ramps so relative rhythm is identical — only the font size scales.

| Style name | Key | Family | Weight | Size | LH | Tracking |
|---|---|---|---|---|---|---|
| `DS/Type/Print/Display` | `bd81e2a5f06e3b401884dbe24f15b5ab963716fe` | IBM Plex Sans | Medium | 42 | 110% | `-0.04em` |
| `DS/Type/Print/H1` | `fd595c7b624d942e985357ce31b593228ffcbeb2` | IBM Plex Sans | Medium | 36 | 110% | `-0.04em` |
| `DS/Type/Print/H2 Large` | `6322a0c84c277de86110e8abc0269671103f9603` | IBM Plex Sans | Medium | 30 | 110% | `-0.04em` |
| `DS/Type/Print/H2` | `83640340c931b1351e15e1b96217486d3cbe3ae6` | IBM Plex Sans | Medium | 22 | 110% | `-0.04em` |
| `DS/Type/Print/H3` | `788f651db0ce11c6f35196ec834be9c92bf61719` | IBM Plex Sans | Medium | 18 | 110% | `-0.04em` |
| `DS/Type/Print/H4` | `92c877b15b7f6886193d0bb355a30ebe49438e47` | IBM Plex Sans | Medium | 14 | 100% | `-0.04em` |
| `DS/Type/Print/Body XL` | `c35f411b1df11ec8e403eaac00b5f8eaaa3fc7e9` | IBM Plex Sans | Regular | 14 | 150% | `0` |
| `DS/Type/Print/Body LG` | `00bb7a70afd86ae73db44bfd1d0e4de016f5fe76` | IBM Plex Sans | Regular | 12 | 150% | `0` |
| `DS/Type/Print/Body` | `b1f7dd86a42852c8463514da98a09cac56326142` | IBM Plex Sans | Regular | 11 | 150% | `0` |
| `DS/Type/Print/Body SM` | `5b39091b3e769456eb07a14f04a90ec001be8fc4` | IBM Plex Sans | Regular | 10 | 150% | `0` |
| `DS/Type/Print/Caption` | `d1749c1fd7340d91bec429b79b4ddcbcde8c02a0` | IBM Plex Sans | Regular | 8 | 150% | `0` |
| `DS/Type/Print/Overline` | `657edcfe4792207a68fe087c2b410048b8a35c4c` | IBM Plex Mono | Regular | 8 | 100% | `+0.08em` |
| `DS/Type/Print/Quote/Mono` | `1ceadb10e8c2a63537c0abb1ef82fe39924a658b` | IBM Plex Mono | Regular | 24 | 110% | `-0.04em` |
| `DS/Type/Print/Quote/Mono SM` | `5036626409334c4df9bcd75c4c27bca469a137c5` | IBM Plex Mono | Regular | 14 | 130% | `-0.01em` |
| `DS/Type/Print/Quote/Serif` | `7d6f850f0a8e81f5c1600231cba975431c2aba48` | IBM Plex Serif | Regular | 24 | 110% | `-0.04em` |
| `DS/Type/Print/Code/MD` | `b776fb0adc5b13ca1485988bc5ad0b8179e86fc7` | IBM Plex Mono | Regular | 9 | 150% | `0` |
| `DS/Type/Print/Code/SM` | `400296190e44c142c4f5425a3aef3d0a700b0b70` | IBM Plex Mono | Regular | 8 | 150% | `0` |
| `DS/Type/Print/Button` | `c5757a30e69159997173e8496fa79e9fe1363aac` | IBM Plex Sans | Regular | 11 | 100% | `0` |
| `DS/Type/Print/Condensed Label` | `fa7df856eb31cf4d66201e97245b5b2f79e8bfab` | IBM Plex Sans Condensed | Medium | 11 | 100% | `0` |
| `DS/Type/Print/Nav` | `f023c3252752bd87aabf7ecf797360b2877a7eab` | IBM Plex Sans Condensed | Regular | 11 | 100% | `0` |

Same plain-zero rules apply (BP #17): Plex Sans defaults plain, Plex Mono needs `ss04`. `DS/Type/Print/Overline` (Plex Mono) requires the same toggle as the web Overline.

### Line breaks — no orphans

An **orphan** is one word left alone on the last line of a block of text. It breaks the reading rhythm and makes a heading look
unfinished. Rule: Best Practice #19, applies to every surface (web, Figma, print, decks, email).

| Situation | Fix |
|---|---|
| Heading intended as **one line** that wraps by a word or two | Step the size down (e.g. `H2 Large` → `H2`, or a page-scoped size override) or tighten the copy until it holds on one line at that breakpoint |
| Multi-line heading that ends on one word | Bind the last two words with a non-breaking space so they travel together (`get&nbsp;paid`) |
| Body / description paragraph | `text-wrap: pretty` on the web; in Figma, nudge the frame width or add a non-breaking space |
| Fixed-format artwork (ads, print, decks) | Soft return (Shift+Return) before the last two words |

**Web implementation:** headings get `text-wrap: balance` (evens out line lengths, which removes most orphans), paragraphs get
`text-wrap: pretty`. Neither is a guarantee at every width, so still bind the final two words with `&nbsp;` in the copy of any
heading that can wrap, and check desktop, tablet and mobile.

**Page-wide safety net (New Home, 2026-10-02):** one typesetting embed per page sets `text-wrap: balance` on `h1–h6`, `[class*="heading-style"]`, quotes and attributions, and `text-wrap: pretty` on `p`, `li` and `[class*="text-size-p"]`. Copy that can still orphan carries `&nbsp;` between its last two words inside the text itself (component instance props included). A hyphenated last word needs a **word joiner** (U+2060) after the hyphen (`estimate-⁠ready`): IBM Plex Sans has no non-breaking hyphen (U+2011) glyph.

**One-line fit:** where a title must hold on one line (the problem / solution card titles), size it to its container: `font-size: min(2rem, (100cqi - 84px) / 17); white-space: nowrap` from 1200px up, balanced wrap below. Put `container-type: inline-size` on an ancestor with a **definite width** (a grid track), never on a shrink-to-fit or flex-aligned element: containment drops its intrinsic width and the element collapses.

**Verification:** scan each text block's last line for a single word at 11 widths from 1600 to 360px; New Home ships with zero orphans at every width.

**Figma:** there is no automatic orphan control. Inspect every text frame at each breakpoint frame; use a non-breaking space or
Shift+Return, never extra spaces or a narrower frame that breaks the grid.

#### Real-world context

- **2026-10-02 · docusketch.com New Home GA draft, problem section.** The title "If it's not in the file, you don't get paid" wrapped
  so that "paid" sat alone on the second line. Chris: "Either we reduce the typeface size or we give it a friend." Preferred outcome:
  both problem titles on one line each (size stepped down for this instance); fallback: `get&nbsp;paid` so the pair wraps together.
  Rule promoted to Best Practice #19 the same day: no orphans, universally.


### IBM Plex — Zero Glyph (plain, not dotted, not slashed)

All three IBM Plex families ship **three** zero glyphs each (`zero` / `zero.alt01` / `zero.alt02`) with the **same OpenType feature tag structure**. The visual design assigned to each glyph **differs per family** — the same feature tag produces different results in each font.

#### Per-family substitution table

| Family | Default `zero` | `ss04` / `salt` → ... | `ss03` / `zero` feature → ... | Figma toggle labels |
|---|---|---|---|---|
| **IBM Plex Sans** | **PLAIN** | `zero.alt01` = **DOTTED** | `zero.alt02` = **SLASHED** | "slashed number zero", "dotted number zero" (both opt-in) |
| **IBM Plex Mono** | **DOTTED** | `zero.alt02` = **PLAIN** | `zero.alt01` = **SLASHED** | "slashed number zero", "plain number zero" |

**Read this table carefully:** the same `ss04` feature tag activates the DOTTED glyph in Plex Sans but the PLAIN glyph in Plex Mono. They are *not equivalent* across the family. The Figma panel labels make this clear — Plex Sans's panel offers "slashed" and "dotted" as opt-in alternates to the plain default, while Plex Mono's panel offers "slashed" and "plain" as opt-in alternates to the dotted default.

#### Web — scope `ss04` to Plex Mono only

Plex Sans's default zero is already plain. Applying `ss04` to it substitutes the default with `zero.alt01` = dotted — the opposite of what you want.

```css
/* Body uses Plex Sans — default is plain. NO font-feature-settings. */
body {
  font-family: 'IBM Plex Sans', system-ui, sans-serif;
}

/* Every Plex Mono rule explicitly carries ss04. */
code, .mono, pre {
  font-family: 'IBM Plex Mono', monospace;
  font-feature-settings: 'ss04' 1;
}
```

**Do not** use `*, *::before, *::after { font-feature-settings: 'ss04' 1 }`. It propagates into Plex Sans descendants and makes them dotted — Plex Sans's default zero is already plain.

#### Figma — manual application, per family

The action depends on which family the text style uses:

| Text style group | Family | Action |
|---|---|---|
| Display, H1, H2 Large, H2, H3, H4, Body XL, Body LG, Body, Body SM, Caption, Button, Nav, Condensed Label | Plex Sans | Confirm both "slashed" and "dotted" toggles are **OFF** (default = plain) |
| Code/SM, Code/MD, Quote/Mono, Quote/Mono SM, Overline, Testimonial | Plex Mono | Toggle **plain number zero** **ON** |

**⚠️ Plugin API limitation:** The Figma Plugin API v1.0.0 does NOT expose `setRangeOpenTypeFeatures` or a writable `openTypeFeatures` property. These toggles must be applied manually:

1. **Assets panel** (⌥2) → Local styles → Text
2. Right-click each affected style → **Edit style**
3. In the style editor → **Advanced type** → **Stylistic sets**
4. Apply the action from the table above
5. Save — all bound text nodes inherit the change immediately

#### Font source matters

Google Fonts ships a subset of IBM Plex that strips the stylistic sets — `'ss04' 1` does nothing against that subset. Always self-host or load the full font from a CDN that ships the unsubsetted woff2:

```html
<link rel="preconnect" href="https://cdn.jsdelivr.net" crossorigin>
<style>
  @font-face {
    font-family: 'IBM Plex Sans';
    font-style: normal;
    font-weight: 500;
    src: url('https://cdn.jsdelivr.net/npm/@ibm/plex-sans@1.1.0/fonts/complete/woff2/IBMPlexSans-Medium.woff2') format('woff2');
    font-display: swap;
  }
  @font-face {
    font-family: 'IBM Plex Mono';
    font-style: normal;
    font-weight: 400;
    src: url('https://cdn.jsdelivr.net/npm/@ibm/plex-mono@1.1.0/fonts/complete/woff2/IBMPlexMono-Regular.woff2') format('woff2');
    font-display: swap;
  }
</style>
```

**Why not `'salt' 1`?** `salt` (Stylistic Alternates) is a meta-feature that flips *all* alternates simultaneously — single-storey `a`, single-storey `g`, the zero alternate, alt-eszett, etc. In Plex Mono it produces plain zero as a side-effect, but it also changes letters in ways you don't want for brand text. In Plex Sans it makes the zero dotted. Always use the specific `ss04` flag where you need it, scoped to the right family.

---

#### Real-world context — Webflow-hosted Plex (2026-10-02, New Home)

Webflow's hosted IBM Plex Mono (`Ibmplexmono`) keeps its stylistic sets and defaults to the **dotted** zero, so `ss04` turns on its plain zero; hosted Plex Sans and Condensed default to plain. The page applies `font-feature-settings: "ss04" on` to every Mono selector by name (eyebrows, attributions, stats, quotes, chips, code), never to `*`.

## Photo Card Pattern (Mobile)

When photography is used inside a card at mobile breakpoints, **never overlay text directly on the image**. Use the split-card pattern from Material Design v3, Apple HIG, and Airbnb:

```
┌─────────────────────────┐
│                         │
│   IMAGE ZONE (3:2)      │  scaleMode: FILL — image always covers zone
│   327 × 218px           │
│                         │
├─────────────────────────┤
│  01. CAPTURE            │  DS/Type/Overline · DS/Color/White @ 65% opacity
│  Field truth, captured  │  DS/Type/Body · DS/Color/White
│  quickly and consist.   │  Fill → DS/Color/Chartreuse Dark
└─────────────────────────┘
```

**Dimensions (at 375px mobile, 24px L/R padding):**
| Zone | Width | Height | Fill token | Notes |
|---|---|---|---|---|
| Image | 327px | 218px | — | scaleMode=FILL, imageHash on rectangle |
| Text zone | 327px | 96px | `DS/Color/Chartreuse 900` | pT=12 pB=12 pL=16 pR=16, gap=4 |
| Full card | 327px | 314px | — | cornerRadius=8, clipsContent=true, layoutMode=NONE |

**Token bindings:**
| Node | `fillStyleId` | `textStyleId` | Opacity |
|---|---|---|---|
| text-zone frame | `DS/Color/Chartreuse 900` | — | 1 |
| Label (overline) | `DS/Color/Base/White` | `DS/Type/Overline` | 0.65 |
| Body copy | `DS/Color/Base/White` | `DS/Type/Body` | 1 |

**Why not text-on-image:**
- Portrait photography cropped to landscape loses subjects unpredictably
- Scrim opacity sufficient for light images makes dark images look over-processed
- Text-on-image fails WCAG contrast requirements without per-image tuning
- Split-card works on every photo regardless of brightness or crop

**Precedents:** Google Material Design v3 "Media card", Apple App Store cards, Airbnb property listings, Spotify Now Playing cards.

---

## Generative AI

Generative AI shows the product in use and builds the job site around it. It never stands in for a real person, real evidence, or the product's own output.

### The two gates (immovable — Best Practice #18)

Published or not. Never "tone it down."

1. **Never develop alarming imagery.** No current or recent weather events or disasters — they read as news and prompt people to act. No people in danger, injuries, collapse, or dread lighting. Synthesized damage — moisture, staining, debris, smoke residue — shown calmly, as found, is fine. Test: *inform, not alarm.*
2. **Never synthesize a real person.** Customers, testimonial givers, subject-matter experts — no likeness, no voice, no attributed words. Anyone presented with a voice, in real time, is real. Generated people are always fictional.

**Context.** Generated imagery *may* show a realistic job-site scenario — that is its purpose. It *may not* be presented as a specific real job, site, or customer. Captions and alt text describe the scene, never a real address, date, or name. Generated files carry `gen-` in the filename.

### Where it belongs

Campaign mode only. Editorial carries no imagery.

| Use | Generative? |
|---|---|
| Technician using the DS1 or the DS° app on site | **Yes** |
| Synthesized job sites with realistic damage | **Yes** — calm framing |
| Rooms, offices, vehicles, exteriors | **Yes** |
| Repeatable fictional cast across a campaign | **Yes** |
| Social, ads, trade-show graphics, one-pagers | **Yes** |
| Moodboards, placeholders (internal) | **Yes** |
| Current or recent weather events, disasters | **No** — gate 1 |
| People in distress, injuries, collapse, dread | **No** — gate 1 |
| Customers, testimonial givers, SMEs — face, voice, or words | **No** — gate 2 |
| Evidence from a real job — before/after, claim docs, tour stills | **No** |
| Product UI, floor plans, estimates, tour views | **No** — composite real captures |
| Real customers' properties, partner logos, named people | **No** |
| Editorial documents — letters, updates, PDFs | **No** |
| Staff and SME headshots, press, events | **No** — photograph it |
| Statistics, claims, names, quotes in copy | **No** |

### Copy

- Lexicon is binding — see `/messaging#lexicon`.
- No invented statistics, names, dates, or claims.
- No generated words for a real person — gate 2.
- Honest over persuasive. Cut the superlatives.
- A person owns every shipped sentence.

### Review log

- **2026-09-14** — Section created. Gates set: no alarming imagery; no generative support for real testimonials.
- **2026-09-14** — Gates sharpened: gate 1 names weather events and anything current; synthesized damage shown calmly is in scope. Gate 2 broadened to any real person (customers, testimonial givers, SMEs) with the real-time-voice rule.
- **2026-09-14** — Cut to three parts: gates with disclosure context, where-it-belongs table, copy. Fuser notes, video/voice, and pre-ship check removed. Every point one to two sentences.

---

## Marketing Page Patterns

Approved page-level patterns, each taken from a shipped or approved page and stated as a rule another page can reuse. Section-level rulings (ink, surfaces, motion, line breaks, touch targets) live in their own sections; this section holds the patterns themselves.

**Source for every pattern below unless noted:** New Home GA draft (`/dev/new-home-ga`, Webflow page `6abae1da24071f5a642f0ffb`), approved as brand canon by Chris Provins on 2026-10-05. Measured from the page's live embed code on that date. Values marked *approved deviation* sit off the documented scales on purpose; keep them for this pattern and don't generalise them.

### Product rows (Capture / Scope / Estimate)

A stacked set of two-column rows, one per product, used where a page introduces the three products in sequence.

| Part | Spec |
|---|---|
| Row | Two equal columns; media and content alternate sides (Scope flips). Rows divided by a 0.5px Neutral 400 hairline, with a 1px Neutral 400 centre line at 60% opacity |
| Media | 24px padding; image **3:2**, `object-fit: cover`, radius 12 (`--radius-lg`) |
| Content | Vertically centred; padding 64 / 80 desktop, 40 tablet, 32 / 24 / 48 mobile |
| Head | Icon tile + pill, 2px apart |
| Icon tile | 36px, 8px padding, radius 8, fill Neutral 650 `#39381B`, the brand-kit product mark in Chartreuse 300 (see *Product marks*) |
| Pill | 36px tall, padding 0 14 (*approved deviation*), 1px ink stroke, radius full; Plex Sans Condensed Medium 14/16.8, +1.12px, uppercase |
| Title | Plex Sans Medium 46.8 / 50.5, −1.5px (*approved deviation* between H2 and H2 Large); 36 / 40 tablet; 30 / 33 mobile; Olive; balanced |
| Description | Plex Sans 18 / 28 (Body LG), Olive, pretty wrap, max 620px |
| Feature chips | 28px tall, padding 0 12, 1px Eucalyptus (Neutral 400) stroke, no fill, radius full; Plex Mono Medium 12 / 16, +0.06em, uppercase, `ss04`; 8px gap, wrap |
| Learn more | See below |

**Learn more button (row link).** 36px tall (the icon tile's height), radius 8, fill and stroke Neutral 650, label in `Text/Inverse` Plex Sans 16 Regular, 20px Material `arrow_forward` drawn as a mask in `currentColor`. Hover / focus: chartreuse fill and stroke, ink label, arrow nudges 4px right; all at Default 200ms / `Easing/Out`. Press: Eucalyptus fill, `Text/Secondary` label. Focus ring 2px ink, 2px offset. 44px tall on touch screens. Five earlier variants were tried and rejected: chartreuse at rest, a subtle fill that read as a chip, an outline-only button, a boxed arrow and a looping arrow.

**Mobile:** one column, media first in every row (the flip is removed), centre line hidden.

### Testimonial accordion

Desktop and tablet: one row of photo cards where one card is open and the rest are slivers; built on the *Meet the Experts* motion (Motion → Showcase tier).

| Part | Spec |
|---|---|
| Width | Capped at 74.5rem (1192px), matching the page's content width |
| Heading | The label style used by the logo band ("Trusted by…"): Plex Mono 16, centred, Olive |
| Row | Flex, 8px gap; open card `flex: 6.5`, slivers `1`, hovered sliver `1.6`; the row's `aspect-ratio` is set so the **open card is 16:9** (five cards → about 2.95:1) |
| Images | Inactive `grayscale(1) sepia(.2)`, colour when open or hovered; per-card focal points: the face centred in a sliver, the person in the right third when open |
| Copy | Quote and attribution shown only on the open card, over a left-to-right Brand/Olive scrim (72% → 40% → 0 at 62%); copy fades in 500ms after 150ms |
| Framing | The page's 26px crosshair, centred 32px outside each end of the row, vertically centred; hidden at 991px and below |
| Dots | One per card: inactive 8px at 25% opacity, active a 40px Olive pill that fills over 7s while autoplay runs |
| Autoplay | Advances every 7s and loops; pauses on hover, focus, off screen and hidden tab, then resumes with the time left; a manual pick restarts the timer; off under reduced motion |
| Build | No JavaScript for the accordion itself: transparent radio inputs laid out with the same flex rules are the hit areas, and `:has(#card-k:checked)` opens card k. Works in Webflow preview and with arrow keys. Autoplay is a progressive enhancement |

**Phone (767px and below):** one card at a time in a **1:1** square framed on the face; a Brand/Olive scrim on the bottom 45%; quote 16 / 1.35, attribution 12; Previous / Next buttons below the square, 44px, radius 8, 1px Olive stroke, CTA arrows.

**Content rules:** quotes are verbatim from the source (trim with `…`, never insert words in `[ ]`); every card is a real, approved customer with their real photo.

### Outcome columns ("Built for")

Four highlight columns that connect the dots after the product rows. They don't re-walk the workflow.

| Part | Spec |
|---|---|
| Heading | One H2 spanning the columns, centred, max 16em, balanced |
| Grid | Four equal columns, padding 0 40 (first column flush left); two columns at 991px and below, stacked at 767px |
| Column | Label (Plex Mono 14, +0.4px, `ss04`) → statement (Plex Sans 26 / 1.22, −0.5px, max 10em so every title runs two lines) → one-line body (17px) → illustration pinned to the bottom |
| Frame | Crosshair rows above and below the columns only, inside the grid: 13px + at the ends, 5px dashes at the three column boundaries, centre + hidden; 56px above the columns and 64px below |
| Illustrations | Equal rendered width (crop each `viewBox` to its own artwork); static (Motion → Decorative motion) |

**Illustration style.** Reference: Figma `guoAdcJQH5m7hrlyH62OGZ` node `2038:6693` (the Capture drawing). Open line work at about 2px, **no fills**, fine dotted secondary lines with round caps, at most one solid dark element as the focal point. Instant Sketch is drawn as the floor plan only. Product marks in an illustration keep their crop marks.

### Client logo band

- Background: a repeating dot tile (r 0.88, Eucalyptus at 75%, pitch 21.333 × 17.531) drawn as a token-filled mask so it recolours with the system.
- The logo row sits 96px from the crosshair centre lines above and below; the label sits 46px above the logos.
- Marquee at half speed (`0.5` / `−0.5`).
- The crosshair row under the hero drops its centre point where it would collide with the hero icon (spacing kept).

### Workflow bracket

The Capture / Scope / Estimate → Connected Job Record bracket is inline SVG. A 2px Chartreuse 200 (`#FFFA37`, bound as `background/accent-active`) segment at 85% opacity travels each line into the stem at a constant 50px/s on a 7s loop. Use Chartreuse 200 here because the section background turns chartreuse, where Chartreuse 300 would vanish. The tip is the CTA's open, square-cut arrow head. Reduced motion shows the static bracket.

### Hero: product-artifact nodes

The hero's floating nodes are real product artifacts (360° Tour, Scope Walkthrough, Instant Sketch, Scope Sheet). Threads from each node converge on one mark that stands for the DocuSketch job record. Rules:

- Product UI inside a node follows the **product** design system (Field Note pins, walkthrough pill, real Instant Sketch plans), never brand accents. Match a real product screenshot over any token inference.
- Show only states the product has. No invented status cards, live transcripts or flows.
- No prices in any estimate or scope render (costing comes from Xactimate).
- Motion inside a node (tour pan, recording pulse) honours reduced motion.

### Image ratios

| Use | Ratio |
|---|---|
| Default for new image frames, cards and generated assets | **16:9** landscape |
| Product row media | 3:2 |
| Open testimonial card | 16:9 (desktop / tablet) · 1:1 (phone) |

### Product marks

Capture, Scope and Estimate icons are the brand-kit marks from the workflow pills: a corner-bracket frame plus ring (Capture), 3×3 dots (Scope) or three bars (Estimate), 24×24, `currentColor`. The crop-mark frame is part of the mark. Never strip it; scale the whole mark to fit a tile or circle. Never substitute the chartreuse 360 / claim-summary / estimate image assets.

### Pre-footer CTA on animated camo

The closing CTA band uses `DS/Pattern/Camo` in Animate mode (kit generator, drift keyframes), with the drift paused while off screen. Type over camo follows Colour Tokens → Patterns (Chartreuse 200).

---

## Compound Component Anatomy

Some components are layered — an outer wrapper, an inner primary surface, and one or more nested interactive elements. Each layer has its own token assignments, and the layered relationship is part of the design (not an implementation detail). Compound components are documented with explicit per-layer anatomy so that any consumer renders the same visual hierarchy.

### Pattern: Halo wrapper

A semi-transparent tint of the inner surface color, sitting just outside the primary surface. Creates a soft "halo" of color around the element that conveys elevation through tone rather than shadow.

| Property | Value |
|---|---|
| Background | Same hue as inner surface, ~25% opacity |
| Padding | `8` |
| Border radius | `32` (`--radius-4xl`) = inner `24` + padding `8`, per the Nested Radius Rule |
| Shadow | none (the halo *is* the elevation cue) |

**When to use**: floating action buttons, sticky CTAs, prominent notifications where a soft surround reads better than a sharp drop shadow.

**Do not stack with `--shadow-*` tokens.** Halo wrapper OR drop shadow, not both. The halo is the elevation treatment for this class of element.

### Component: Sticky CTA (canonical compound-component example)

> **Figma source:** the `Sticky CTA` component set in the [DS° Marketing Website](https://www.figma.com/design/guoAdcJQH5m7hrlyH62OGZ/) Figma file. The chartreuse `Variant2` is what's currently live on docusketch.com.

Three layered elements. Each layer gets its own token assignments:

```
Sticky CTA  (compound component)
├─ Outer (halo wrapper)
│   - background: rgba(26, 25, 5, 0.5)            translucent black
│   - backdrop-filter: var(--ds-effect-glass)     blur(20px) saturate(140%) — frosted glass
│   - padding: 8
│   - border-radius: 32 (--radius-4xl)
│
├─ Inner (pill surface)
│   - background: DS/Color/Black            #1a1905
│   - padding: 8 / 8 / 8 / 16                     (top / right / bottom / left)
│   - border-radius: 24 (--radius-2xl)
│   - box-shadow: --shadow-floating
│   - gap: 24                                     (between text and icon button)
│
└─ Icon button (nested interactive)
    - background: DS/Color/Chartreuse       #e5df00 (regular Chartreuse 300, not the brighter Chartreuse 200)
    - size: 48 × 40
    - border-radius: 16 (--radius-xl)
    - padding: 10 / 12                            (vertical / horizontal)
    - icon: Material Symbols `arrow_forward` at 24 × 24
    - icon colour: #1C1B1F (≈ Brand/Black)
```

**Viewport positioning:** the Sticky CTA sits `bottom: 56` from the viewport bottom, horizontally centred (`left: 50%` + `translateX(-50%)`). `56` is the first value on BP #12's layout tier — the right semantic choice for viewport-edge positioning.

**Halo glass effect**: the outer halo wrapper carries `backdrop-filter: var(--ds-effect-glass)` where `--ds-effect-glass` resolves to `blur(20px) saturate(140%)`. Underlying page content is blurred and saturation-boosted behind the tint, producing the frosted-glass look. The blur abstracts content into soft colour fields without losing scale; the saturation boost keeps the underlying colours vivid. Same `backdrop-filter` across all three interaction states — only the tint colour changes between Default / hover / press.

*Browser support*: `backdrop-filter` is unprefixed in Chrome/Edge (76+), Firefox (103+), and modern Safari (18+); older Safari needs `-webkit-backdrop-filter` alongside. Emit both forms.

**Text inside the pill**: IBM Plex Sans Regular at 24px, line-height 110%, tracking `-0.04em` (= `-0.96px` at 24px). Colour is **`Neutral/300`** in the Default (black) state and **`Brand/Black`** in the hover (chartreuse) state — see the Interaction states table. Note this is **Regular** weight — the `DS/Type/H4` token is Medium; the Sticky CTA's "Book A Demo" label uses Sans Regular at H4 size. *This is a documented exception; do not generalise — most surfaces use the canonical type styles.*

**Interaction states**:

> **Olive ruling 2026-10-05.** Where this component spec says `Brand/Black` (`#1a1905`), the brand dark is now Olive `#39381B`. On the web, bind the pill fill and hover text to `Brand/Olive`. The Figma component keeps the old fill until it is re-filled, so treat the values below as Figma's current state, not the target.

| Layer | Default (rest) | hover | press (active) |
|---|---|---|---|
| Halo | `rgba(26,25,5,0.5)` (black-tinted) | `rgba(255,250,55,0.25)` (chartreuse-tinted) | `rgba(249,249,245,0.25)` (bg-warm-tinted) |
| Pill | `Brand/Black` `#1a1905` | `Brand/Chartreuse` `#e5df00` | `Brand/Eucalyptus` `#c0bc90` |
| Text | `Neutral/300` `#dfddc8` | `Brand/Black` | `Neutral/500` `#807c5e` |
| Icon button bg | `Brand/Chartreuse` `#e5df00` *(regular, not the brighter Chartreuse 200)* | `Chartreuse 200` `#fffa37` | `Neutral/500` `#807c5e` |
| Arrow colour | `#1C1B1F` (≈ Brand/Black) | `#1C1B1F` (unchanged from rest) | `Olive` `#39381b` *(verify against Figma asset)* |
| Figma node | `25:233` (Figma "hover" variant) | `25:235` (Figma "Default" variant) | `25:597` |

**The hover is a full colour inversion.** Black pill lights up to chartreuse; text flips from Neutral 300 to Brand/Black; the icon button shifts from regular Chartreuse to the brighter Chartreuse 200 (formerly Chartreuse Active). The press state mutes to eucalyptus.

**Transition rules:**
- **Colour transitions** (background, text, icon arrow color): `var(--ds-motion-duration-default)` (200ms), `var(--ds-motion-easing-out)`. Applied between all three states.
- **Arrow loop on hover and leave**: a CSS keyframe animation runs once per state-transition in either direction. The arrow drifts right with an opacity fade, teleports off-screen during invisibility, fades back in from the left, and settles at center. Travel is **20px** — most of the "exit" work is done by opacity, matching the drawer's `translateY(24%) + opacity` pattern. **Per-keyframe easing**: the exit half uses `Easing/In` (arrow accelerates as it leaves); the entry half uses `Easing/Out` (arrow decelerates as it settles). Duration: `var(--ds-motion-duration-emphasis)` (400ms total).

  **Bidirectional trigger:** the same animation fires on `mouseenter` AND `mouseleave` (and `focus` / `blur` for keyboard parity). Both directions of interaction get acknowledged with the same forward-motion gesture. Implementation pattern follows the Motion → Patterns → "Hover/leave loop animation" row:

  ```js
  // JS — toggle .is-animating with reflow-forced restart so the animation
  // re-triggers on every state transition (not just mouseenter).
  const arrow = cta.querySelector('.cta-icon svg');
  let timeout = null;
  function play() {
    arrow.classList.remove('is-animating');
    void arrow.offsetWidth;  // force reflow — restarts the animation
    arrow.classList.add('is-animating');
    clearTimeout(timeout);
    timeout = setTimeout(() => arrow.classList.remove('is-animating'), 400);
  }
  cta.addEventListener('mouseenter', play);
  cta.addEventListener('mouseleave', play);
  cta.addEventListener('focus', play, true);
  cta.addEventListener('blur', play, true);
  ```

  ```css
  @keyframes cta-arrow-loop {
    0%   {
      transform: translateX(0);
      opacity: 1;
      animation-timing-function: var(--ds-motion-easing-in);    /* exit half */
    }
    49%  { transform: translateX(20px); opacity: 0; }
    50%  {
      transform: translateX(-20px);
      opacity: 0;
      animation-timing-function: var(--ds-motion-easing-out);   /* entry half */
    }
    100% { transform: translateX(0); opacity: 1; }
  }
  .sticky-cta:hover .icon svg,
  .sticky-cta:focus-visible .icon svg {
    animation: cta-arrow-loop var(--ds-motion-duration-emphasis) linear;
    /* Overall TF is linear; per-keyframe animation-timing-function does the easing. */
  }
  ```

  **Why per-keyframe easing:** exits feel right with ease-in (accelerate as you leave) and entrances feel right with ease-out (decelerate as you settle). Per-keyframe `animation-timing-function` lets each half use the right curve.

  **Why a loop, not a slide-and-reverse:** a reverse animation on mouse-out reads as "the arrow is going backward" — wrong affordance for a forward-pointing CTA. A loop preserves the forward momentum even when the hover ends mid-animation. If mouse-out happens while the animation is running, the animation simply stops and the arrow snaps back to center (its rest position) — no jarring reverse.

  Triggered on hover AND focus-visible (keyboard parity). The icon button's parent must have `overflow: hidden` so the arrow clips at the boundaries during the loop.
- **`prefers-reduced-motion: reduce`**: arrow loop animation is disabled and entrance translate is bypassed; colour transitions still apply (colour shifts aren't motion in the WCAG 2.3.3 sense).

**Motion (entry/exit)**: see the Motion section → Patterns → Scroll-triggered CTA. Entry is `transform: translateY(24% → 0)` + opacity fade, Emphasis duration, `Out` easing. Never elastic / bounce.

### Convention for documenting future compound components

Each compound component documented in this section should have:

1. A Figma source link at the top (node ID + clickable URL)
2. A layered anatomy diagram (outer → inner → nested) using ASCII tree
3. Explicit token assignments per layer (paint, padding, radius, shadow, gap, size)
4. Text styling spec if different from the canonical type styles
5. Variant differences as a table
6. A pointer to the motion pattern that governs entry/exit

---

## Spacing Scale

Base unit: **4px**. Prefer **8px multiples** for layout-level gaps. Scale is split into two tiers (see BP #12).

### Content tier — gaps and padding inside components

**Permitted values:** `4 · 8 · 12 · 16 · 20 · 24 · 32 · 40 · 48`

Use for: gaps between elements within a component, padding inside a card / button / panel, distances between paired UI (icon ↔ label, title ↔ meta, etc.).

### Layout tier — viewport-edge padding, section gaps, page margins

| Value | Usage |
|---|---|
| `56` | Tight viewport-edge padding (mobile-tablet bridge cases) |
| `64` | Standard viewport-edge padding for full-width content; medium section gap |
| `80` | Desktop L/R padding (per Website Section outer margin standard); large section gap |
| `96` | Extra-large section gap, division between major page regions |
| `128` | Hero gaps, top-of-page / bottom-of-page padding, division between distinct page surfaces |

Use for: outer layout, viewport-edge clearance, section-to-section gaps. Components themselves should not consume these values internally — only the page / layout context.

### Touch targets (ruling 2026-10-05)

Interactive controls are at least **44px** in each direction on touch screens. Grow the hit area, not the visual: under `@media (pointer: coarse)` add padding (with a matching negative margin where layout must not move) so an 8px dot or a 36px button keeps its look and gains a 44px target. Desktop keeps the visual size. Applied on New Home to the testimonial dots, the menu button, the Learn more buttons and the CTA.

### Forbidden values

`6 · 10 · 14 · 22 · 36 · 44 · 60 · 72 · 100 · 112 · 144` — and any other value not in either tier above. The scale is intentionally restrictive; if a design needs a value not listed, that's a design question, not a system question. To extend the scale, add the value here in the same PR with a dated rationale.

### Why two tiers

Content-tier values need fine control because small UI is sensitive to small steps (4px difference between 12 and 16 is meaningful in a 24px-tall input). Layout-tier values don't — at 56 vs 64 vs 80, an 8px difference is the smallest meaningful step, and finer steps just add noise. The split keeps the small end precise without bloating the large end with values designers don't actually need.

### Component patterns (codified from audit)

| Component | Gap | Padding |
|---|---|---|
| Stat (label ↕ value) | `8` | — |
| Bullet (icon ↔ text) | `8` | — |
| Header (title ↔ meta) | `8` | — |
| Testimonial block | `16` | `T24 R28 B24 L28` |
| Badge / pill | — | `T4 R8 B4 L8` |
| One-pager page margin | — | `T40 R48 B40 L48` |

### Website Section outer margin standard

All Website Section components exist at **three breakpoints**. Each breakpoint has its own component named `Section Name / Tablet` and `Section Name / Mobile`.

| Breakpoint | Canvas | L/R Padding | Content width |
|---|---|---|---|
| Desktop | `1580px` | `80px` | **1420px** |
| Tablet  | `768px`  | `40px` | **688px**  |
| Mobile  | `375px`  | `24px` | **327px**  |

**Layout rules by breakpoint:**
- **Desktop → Tablet**: Outer padding only changes. Horizontal side-by-side layouts are preserved where content fits; inner fixed-width frames resize to content width.
- **Tablet → Mobile**: Structural changes apply — `HORIZONTAL` layouts become `VERTICAL` (stacked). Decorative elements (nav arrows, secondary mockups) are hidden (`visible = false`). Each column becomes full content width.

**Per-section responsive decisions:**

| Section | Tablet | Mobile |
|---|---|---|
| Testimonial Copy Only | HORIZONTAL (arrows stay) | VERTICAL — arrows removed, quote centred |
| Feature Card Slider | VERTICAL, header resizes | VERTICAL, header stacks title/arrows |
| Page Hero | 2-col at 360/296px split | Single col, mockup hidden |
| Focus CTA | HORIZONTAL, card fills gap | VERTICAL, decorative vectors removed |
| App + Portal Hero | Both mockups scaled down | Mobile device only (desktop hidden) |
| Workflow Visual Model | 3-col triptych ~213px/panel | Panels stack VERTICALLY, full width |

**Other rules:**
- Full-bleed IMAGE fills expand to cover the full frame regardless of padding — padding governs content positioning only
- Vertical padding is section-specific but must stay on the 8px grid
- Component naming convention: `{Section Name}` / `{Section Name} / Tablet` / `{Section Name} / Mobile`
- **Mobile image max height: 320px.** No image zone on a Mobile breakpoint may exceed 320px in height. A user should never scroll past a full-screen image. Implementation: set `clipsContent = true` on the image container and resize it to ≤320px. For component INSTANCE children (which can't be directly resized), resize the INSTANCE to the target height and enable `clipsContent` — the component image zone is cropped to the instance bounds. Hidden frames inside VERTICAL auto-layout still consume space; set `layoutPositioning = 'ABSOLUTE'` on any hidden decorative frame to remove it from the flow. Always recalculate section height as `paddingTop + contentHeight + gap + imageHeight + paddingBottom`.

**Tablet-specific audit rules (768px breakpoint):**
- **Content width is 688px** (768 − 40pL − 40pR). Every text node and inner layout frame must be ≤688px wide. Overflow from desktop (typically 1420px-wide descendants) must be explicitly resized.
- **Type scale stays close to desktop.** H1 (64px) is correct for hero headings. H2 Large (54px) is preferred for section headings (Workflow, etc.). H2 (36px) for sub-headings. Body LG (18px) for panel body text. Quote/Mono (38px) for testimonial quotes.
- **SPACE_BETWEEN layout overflow.** If a HORIZONTAL auto-layout uses `primaryAxisAlignItems = 'SPACE_BETWEEN'` and the sum of child widths exceeds the frame width, children overlap. Diagnosis: `childA.x + childA.width > childB.x`. Fix: switch the row to VERTICAL layout and stack title above CTA button, or reduce the title frame width to `frameWidth − gap − buttonWidth`.
- **Right-aligned attribution rows.** `primaryAxisAlignItems = 'MAX'` on a HORIZONTAL frame means children right-align. Resizing such a frame causes children to reposition automatically — no manual x-coordinate update needed.
- **Content-zone centering causes negative x.** A child frame that is wider than its VERTICAL auto-layout parent will be centred, producing a negative x offset (e.g. 746px child in 688px parent → x = −29). Fix: resize the child to ≤688px and the centering places it at x = 0.
- **Card padding scale.** Desktop: 64px. Tablet: 48px. Mobile: 32px. Use these when adapting a section across breakpoints.
- **Tablet hero heading scale.** The Page Hero uses `DS/Type/H2 Large` (54px) at the 768px breakpoint. The full H1 (64px) produces 4 lines on a 360px column, pushing the section past a single viewport height. H2 Large still produces 4 lines but with 10px less per line, bringing the section to ~595px.
- **Tablet body text scale.** A 27px body style (library `d8579677...`) wraps to 6+ lines on a 360px column. At tablet, bind to `DS/Type/Body LG` (18px) instead — it wraps to ~4 lines, saving ~90px of height. Rule: if desktop body at 27px is used on a column narrower than ~480px, step down to Body LG (18px).
- **Tablet image aspect ratio — preserve desktop ratio.** Image frames must maintain the same aspect ratio as the desktop version across breakpoints. Formula: `tabletImgH = round(tabletImgW × (desktopH / desktopW))`. Page Hero: desktop image is 456×345 (ratio 1.322:1 landscape); tablet image at 296px wide → `296 × (345/456) = 224px` tall. The image sits at the top of its column and the section background fills below — this is the correct responsive behaviour, not a sign of imbalance. A portrait image at tablet (height > width) is **wrong** and means column height was used instead of the desktop ratio.

---

## Page Architecture

```
Cover
_Library          ← slot 1 — all COMPONENT_SET nodes live here
How to Use
╌╌ Foundations ╌╌
  Colour / Typography / Spacing / Border Radius / Shadows
╌╌ Brand ╌╌
  Logo & Identity / Iconography
╌╌ Components ╌╌
  Navigation / Buttons & CTAs / Forms & Controls / Cards & Content
╌╌ Guidelines ╌╌
  Accessibility / Changelog
```

---

## _Library Component Sets

Run the rename script in `plugin-queue/rename-components-*.js` to apply all naming fixes.

**Figma source URLs.** The Set Key column gives the node ID. Construct the full URL by combining it with the source file from "Source File Component Keys" below:

- BDK rows (Set Keys starting `266:*` / `271:*` / `274:*`) → file key `JR35zTngKUblEKMD0myUyD` (Brand Design Kit). Example URL: `https://www.figma.com/design/JR35zTngKUblEKMD0myUyD/Brand-Design-Kit?node-id=266-997`
- Marketing-Website-2026 components — separate file key `guoAdcJQH5m7hrlyH62OGZ`. Example: the Sticky CTA at `25:234` lives in that file, documented under Compound Component Anatomy → Component: Sticky CTA above.

The `Sticky CTA` row below (Set Key `266:997`) references the BDK definition. The canonical implementation currently live on docusketch.com is at `25:234` in the Marketing-Website-2026 file — these may have diverged. Reconciling them is tracked as Tier 2 #10 (Consolidate component sources).

| Current Figma Name | Target Name | Variants | Set Key |
|---|---|---|---|
| `DocuSketch° wordmark` | `Logo / Wordmark` | Color=Chartreuse · Dark · White | 266:1521 |
| `DocuSketch° DS1` | `Logo / DS1` | Color=White · Black · Chartreuse | 266:1535 |
| `DS°` | `Logo / DS°` | Color=White · Black · Chartreuse | 266:1542 |
| `DocuSketch Pill` | `Logo / Pill` | Color=Black · Chartreuse · White | 266:1553 |
| `Insta360 \| DocuSketch° DS1` | `Logo / Insta360 Partnership` | Color=White · Black · Chartreuse | 266:1528 |
| `Logos` | `Logo / Industry Partners` | Brand=Dark · Light — full-width 1530px; dot grid 1510×144; logos overflow ±311px (marquee) | 266:1277 |
| `Main Navigation` | `Navigation / Main Navigation` | Variant=Default · Platform · Solutions | 266:1023 |
| `Announcement Bar Top` | `Navigation / Announcement Bar` | Variant=Default · Chartreuse | 266:1322 |
| `Sticky CTA` | `Navigation / Sticky CTA` | Style=Dark · Chartreuse · Muted | 266:997 |
| `Navigation / Nav Item` | `Navigation / Nav Item` ✓ | Variant=Static · Hover · Active | 256:1906 |
| `Frame 1321316425` | `Navigation / Footer` | (single) | 266:1311 |
| `Divider` | `Content / Divider` | Mode=Dark · Light | 266:1300 |
| `Feature Cards` | `Cards / Feature Card` | Card=DS1 360 Camera · 360 Tour · Timelines · Add Scope · Accurate Sketches · Estimate Creation · Collaboration · Field Camera Kit; Style=Image · Detail | 274:1834 |
| `Next` | `Layouts / Website Sections` | Feature Card Slider · Page Hero · Focus CTA · App + Portal Product Hero · Workflow Visual Model · Testimonial Copy Only | 271:985 |
| `Workflow Icons` | `Icons / Workflow` | Icon=Capture · Scope · Estimate; Size=32 · 24 | 266:1560 |

**All variants currently use `Property 1=X` naming — renames needed:**
- Feature Cards: `Property 1` → `Card`, `Property 2=Default` → `Style=Image`, `Property 2=Variant2` → `Style=Detail`
- Workflow Icons: `Property 1=Capture` → `Icon=Capture, Size=32`, `Property 1=Capture 24` → `Icon=Capture, Size=24` (same for Scope, Estimate)
- Logo sets: `Property 1` → `Color`
- Navigation sets: `Property 1` → `Variant` or `Style`

---

## Source File Component Keys

> **`DS-Brand-UI-kit` (`iL3MqRVVsyma2D5kL8kZm9`) is retired.** It previously appeared here with
> component keys for the Wordmark, DS1, DS°, Pill and Insta360 Partnership marks. That file is
> being deleted; all five are canonically keyed in the **Brand Design Kit** below, per Best
> Practice #15. Never key a component from a file that no longer exists —
> `importComponentByKeyAsync` fails on it.

### Brand Design Kit (`JR35zTngKUblEKMD0myUyD`)

Document / Page Eyebrow (`_Library` page, set `1183:221`):
- Surface=Dark (Chartreuse stroke + Chartreuse text): `45e29a8b9f1d7b7b4db64b5a2818abdbeee60aa8`
- Surface=Light (Black stroke + Black text): `63446974658e668edd0b3db6091178c137d28beb`

Feature Cards (Cards & Content page, node 274:1834 — needs moving to _Library):
- DS1 360 Camera / Style=Image: `274:1515` · Style=Detail: `274:1532`
- 360 Tour / Style=Image: `274:1548` · Style=Detail: `274:1565`
- Timelines / Style=Image: `274:1576` · Style=Detail: `274:1593`
- Add Scope / Style=Image: `274:1609` · Style=Detail: `274:1626`
- Accurate Sketches / Style=Image: `274:1642` · Style=Detail: `274:1659`
- Estimate Creation / Style=Image: `274:1675` · Style=Detail: `274:1692`
- Collaboration / Style=Image: `274:1708` · Style=Detail: `274:1725`
- Field Camera Kit / Style=Image: `274:1741` · Style=Detail: `274:1758`

Forms & Controls (Forms & Controls page, node 186:11):
- 24px/arrows · 24px/checkbox · 24px/radio · 24px/timeline · 24px/compare
- 24px/close · 24px/search · 24px/copy link · 20px/switch · 24px/info
- 24px/AI · 24px/AI stroke

### DS°-Marketing-Website-2026 (`guoAdcJQH5m7hrlyH62OGZ`)
- Main Navigation: `3f712908dcc60bf41783d3327f55f2626f99ce83`
- Announcement Bar: `6b0d667f4ec095afe7cbc56d88c714a6f91ff636`
- Sticky CTA: `76e3bbf33c439991ffccca1f7b1dc8b62dcc49b8`
- Nav Item: `0b03724ffd6bd8435196c9c27e06ac96f73012b8`
- Footer: `ebffb93a53641449b7cdd7b08a4e9bda894e839d`
- Trust Bar: `de6e0f9cf5256ec5047200ef6b61aa0b5ecadb1b`
- Social Proof Strip: `5a695f894dfcc57da80b00e2201d83f0bb1b8179`
- Divider: `07a9e3e950e7be9da80b00e2201d83f0bb1b8179`

---

## Token Application Pattern

**`tok(node, styleKey)`** — the standard way to apply any brand colour. Import once, apply everywhere.

```javascript
// Pre-import all needed styles at the top of every script (Promise.all for speed)
const KEYS = {
  // Backgrounds
  bgWarm:    'beb776cec309fb787d227fb1029944c18003013c',  // DS/Color/Warm      #f9f9f5
  bgDefault: '691aa8e40e607784c4c4e404482edb4154c19dc3',  // DS/Color/Default   #f2f1ea
  bgDark:    'b13030801ac6bfede8476d73fbcbcdc81624a63b',  // DS/Color/Black      #1a1905
  bgChart:   'a5dd296082af737698a18e2d3d03dc08cdea4acc',  // DS/Color/Chartreuse #e5df00
  // Brand
  black:     '79e83948b8311ff1b1c1c0e9f928f6b204949570',  // DS/Color/Black          #1a1905
  white:     'd97f64ea6c0e44f4fa3f6e9be1b5abf7e88ad719',  // DS/Color/White          #ffffff
  chart:     '84537c0a3d84b13e7c36fc93eab3bcfdddbf68d6',  // DS/Color/Chartreuse     #e5df00
  chartDk:   'e2ffcb6a6fe50a91050f1fc54ed0fc12158ae7c7',  // DS/Color/Chartreuse Dark #2a2808
  eucalyptus:'e72d9522dcb925a76af912eb4a61173aabb848be',  // DS/Color/Eucalyptus     #c0bc90
  // Neutrals
  n300:      '2da31f3779da151d1d45fdc1a86dfc980f2483ad',  // DS/Color/Neutral 300    #dfddc8 (re-tuned from #e2e0d3 for chroma progression — re-publish the Figma style)
  n400:      'b790aa71fab53fae6a4361aa787bf922ebe69bde',  // DS/Color/Scale/Neutral/400  #c0bc90
  n500:      'df6f1bdac791116d5ae8105c412ea01246a3a607',  // DS/Color/Scale/Neutral/500  #807c5e
  n600:      '0719226c9ad3c8ef20da9e7267f207039e59cc82',  // DS/Color/Scale/Neutral/600  #39381b (→ Neutral 650)
};

const S = {};  // populated styles cache
await Promise.all(
  Object.entries(KEYS).map(async ([k,v]) => { S[k] = await figma.importStyleByKeyAsync(v); })
);

// Apply a paint style to any node
function tok(node, key) { node.fillStyleId = S[key].id; }

// Usage examples
tok(frame, 'bgWarm');     // frame background → DS/Color/Warm
tok(textNode, 'black');   // text colour → DS/Color/Black
tok(rect, 'chart');       // accent bar → DS/Color/Chartreuse
```

**Rule**: every `solid(hex)` fill on a brand element must be replaced with a `tok()` call before the script is done.

---

## Core Helper Functions

```javascript
function hexRgb(h){return{r:parseInt(h.slice(1,3),16)/255,g:parseInt(h.slice(3,5),16)/255,b:parseInt(h.slice(5,7),16)/255};}
// ⚠️  solid() is for transparent containers (fills=[]) and layout scaffolding ONLY.
// Brand-coloured nodes must use tok() — see Token Application Pattern above.
function solid(hex,op=1){return[{type:'SOLID',color:hexRgb(hex),opacity:op}];}

// Pass styleKey (from KEYS above) as the colour — tok() is called internally after append.
async function sans(parent,str,size,weight,styleKey,opts={}){
  const t=figma.createText();
  await figma.loadFontAsync({family:'IBM Plex Sans',style:weight});
  t.fontName={family:'IBM Plex Sans',style:weight};
  t.characters=str; t.fontSize=size;
  if(opts.width){t.textAutoResize='HEIGHT';t.resize(opts.width,t.height);}
  parent.appendChild(t);
  tok(t, styleKey);  // ← applies DS/Color paint style
  return t;
}

async function mono(parent,str,size,styleKey,opts={}){
  const t=figma.createText();
  await figma.loadFontAsync({family:'IBM Plex Mono',style:'Regular'});
  t.fontName={family:'IBM Plex Mono',style:'Regular'};
  t.characters=str; t.fontSize=size;
  if(opts.width){t.textAutoResize='HEIGHT';t.resize(opts.width,t.height);}
  parent.appendChild(t);
  tok(t, styleKey);  // ← applies DS/Color paint style
  return t;
}

function frm(parent,name,dir,pT,pR,pB,pL,gap){
  const f=figma.createFrame();
  f.name=name; f.layoutMode=dir;
  f.paddingTop=pT; f.paddingRight=pR; f.paddingBottom=pB; f.paddingLeft=pL;
  f.itemSpacing=gap;
  f.primaryAxisSizingMode='AUTO'; f.counterAxisSizingMode='AUTO';
  f.fills=[]; parent.appendChild(f); return f;
  // After appending, call tok(f, 'bgWarm') etc. if frame needs a brand fill.
}
```

---

## Local Component Creation Pattern

```javascript
async function localComponentFromSource(sourceComp, variantName, libPage) {
  const inst = sourceComp.createInstance();
  libPage.appendChild(inst);
  const frame = inst.detachInstance();
  const comp = figma.createComponent();
  libPage.appendChild(comp);
  comp.resize(frame.width, frame.height);
  if (frame.layoutMode && frame.layoutMode !== 'NONE') {
    comp.layoutMode = frame.layoutMode;
    comp.paddingTop=frame.paddingTop; comp.paddingRight=frame.paddingRight;
    comp.paddingBottom=frame.paddingBottom; comp.paddingLeft=frame.paddingLeft;
    comp.itemSpacing=frame.itemSpacing;
    comp.primaryAxisSizingMode=frame.primaryAxisSizingMode;
    comp.counterAxisSizingMode=frame.counterAxisSizingMode;
    comp.primaryAxisAlignItems=frame.primaryAxisAlignItems;
    comp.counterAxisAlignItems=frame.counterAxisAlignItems;
  }
  comp.fills=[...frame.fills];
  if(frame.cornerRadius!==undefined)comp.cornerRadius=frame.cornerRadius;
  if(frame.clipsContent!==undefined)comp.clipsContent=frame.clipsContent;
  for(const child of [...frame.children])comp.appendChild(child);
  frame.remove();
  comp.name=variantName;
  return comp;
}
```

---

## Critical Gotchas

- **`counterAxisAlignItems='STRETCH'` is invalid** — only `'MIN'|'MAX'|'CENTER'|'BASELINE'`.
- **`layoutPositioning='AUTO'` required** — children inside a COMPONENT_SET with auto-layout must have this set or they float (absolute position) and overlap.
- **`layoutSizingHorizontal='FIXED'`** — set on each child after enabling auto-layout to prevent stretching.
- **Pages show 0 children** without `setCurrentPageAsync`.
- **Imported components not findable** across separate plugin calls — import AND use in same script.
- **Multi-pass detach**: run 5–6 passes to fully detach nested instances.
- **Never resize a VECTOR non-uniformly** — Figma scales vector PATH COORDINATES when you call `node.resize(w, h)`. Resizing only one axis (e.g. width) while the other stays fixed compresses circular dot grids into ellipses. Always resize vectors proportionally: `node.resize(w * scale, h * scale)`. After any component width change, re-check dot aspect ratio (`path_w / path_h ≈ 1.0`).
- **Full-width components must be 1530px** — don't let `counterAxisSizingMode='AUTO'` on the COMPONENT_SET shrink a full-page section component. Full-width components (trust bars, dividers, footers) are designed at 1530px. Set `paddingLeft = paddingRight = 0` on their COMPONENT_SET.
- **Never touch `primaryAxisSizingMode` or `counterAxisSizingMode` on component children** — these are the designer's internal sizing decisions. Only set `layoutPositioning`, `layoutSizingHorizontal`, and `layoutSizingVertical` on children when applying COMPONENT_SET auto-layout.
- **`clipsContent=false` required on COMPONENT_SETs** — Figma auto-sets `clipsContent=true` when auto-layout is applied. Explicitly reset to `false` on every COMPONENT_SET to preserve intentional overflow (marquee strips, wide shadows, etc.).

---

## Iconography

DocuSketch draws from **two** icon sources. Which one you use is determined by what the icon
*means*, not by which surface you are building — brand and product share one vocabulary.

| The icon represents | Source | Example |
|---|---|---|
| A named DocuSketch product feature or domain concept | **Universal icon set** (Product DocuSketch Portal) | 360° tours, Timeline, Compare, room list, water/mold/fire |
| A generic UI affordance with no product meaning | **Material Symbols Outlined** | arrows, close, search, download, external link |
| A product concept the universal set does not yet cover | Material Symbols Outlined, as a declared fallback | see *Self-serve and gaps* |

A product concept always prefers the universal set, **including on marketing surfaces**. The
pricing page is the reference: it ships the universal `360°` and `timeline` icons beside Material
Symbols `download`, `contract` and `language`.

---

### The universal icon set

The icon set is the one the DocuSketch product ships, and it serves marketing surfaces too.
One library, two consumers — not a product asset that brand borrows.
It is not a product asset that brand borrows — it is one vocabulary with two consumers.

**Source of truth**

| | |
|---|---|
| Figma file | `uWPtoRCtbBAnOqzJVNhYr3` — *3.0 Design system* |
| Page | `33:2227` — *Particles* |
| Section | `5422:33452` — *Icons* |

| Subsection | Node | Notes |
|---|---|---|
| 24px icons | `13463:4310` | **Default.** 188 symbols; use this set unless you have a reason not to |
| 24px icons dark | `18035:25077` | `Dark` suffix; for dark surfaces |
| 16px icons | `5422:33541` | `Small` suffix |
| 16px icons dark | `18041:36050` | |
| 32px icons | `18541:4328` | |
| iOS SF Symbols | `5422:33569` | Native iOS only |

**Naming convention.** `lowerCamelCase` semantic names — `noteOutline`, `sketchRoom`,
`claimSummary`, `waterOutline`. Suffixes: `Dark` (dark variant), `Small` (16px set),
`Filled` / `Outline` (weight pairs, e.g. `equipmentFilled` / `equipmentOutline`).

---

### Ownership and change control

- **Product self-serves.** The product team adds icons to the universal set as needed. There is no
  brand gate on an ordinary addition.
- **Chris owns it when it is a new product or a positioning move.** Those decisions route to him
  first; the icon still lands in the universal set afterwards.
- **Upstream adoption is handled by Chris offline.** It is not part of any automated pipeline.

#### Brand consumes read-only

Brand **pulls and mirrors. It never alters or edits** the source file, and it never changes how
product documents or accesses the set.

> **Hard rule for anyone using this skill in Claude: never publish or add an icon.**
> Do not write to the universal set. Do not add files to the brand mirror. Do not add rows to the
> manifest or to the component icon map. Do not open a PR proposing an icon. Use what exists,
> fall back where documented, and report the gap in your response — nothing more.

**Read-only allowlist for `uWPtoRCtbBAnOqzJVNhYr3`.** The Figma MCP exposes write tools next to
read ones. Against this file, only these four are permitted:

`get_metadata` · `get_design_context` · `get_screenshot` · `download_assets`

Plus two read-only REST endpoints: `GET /v1/files/{key}` and
`GET /v1/images/{key}?ids=…&format=svg`.

Never `use_figma`, `create_new_file`, `upload_assets`, `add_code_connect_map`,
`send_code_connect_mappings`, or any other mutating call.

> The brand token's role on this file is **editor**. Read-only is a policy, not a permission —
> this allowlist is the only thing standing between the skill and an edit to product's source.

---

### The brand mirror

Brand keeps pinned copies so builds do not depend on live Figma access, exactly as
`tokens/ds-tokens.css` mirrors the DS° Tokens library 1:1.

| | |
|---|---|
| Location | `brand-design-kit/assets/icons/{16,24,32}/` — plus seven still in `assets/menu/` |
| Registry | `brand-design-kit/assets/icons/manifest.json` |
| Drift guard | `figma-sync.py` — read-only (`GET /v1/files/{key}`), same three-way freshness check it already runs for the doc |

**Filenames match upstream verbatim** — `waterOutline.svg`, not `water-outline.svg` — so a
filename is a lookup key back into the source set. Brand-side aliases belong in the manifest, never in
the filename.

**Normalization is mirror-side and must be recorded.** Pull on the **upstream viewBox**, never the
tight-bbox variant, so every icon shares one optical grid.

Use the REST images endpoint — it batches many nodes in one read and returns a clean single-path
SVG with no background rect and no stray artboard paths:

```
GET /v1/images/{fileKey}?ids=<nodeId,nodeId,…>&format=svg
```

1. svg root: `fill="none"` → `fill="currentColor"` so the icon inherits `DS/Color/*`
2. Drop literal `fill="#1A1905"` from child shapes so they inherit
3. Drop the root `width`/`height`; size via CSS
4. Keep the upstream `viewBox` verbatim — usually `0 0 24 24`, but not always
   (`docuSketchMini` is `0 0 26 24`; size non-square marks by height)

> The MCP's `download_assets` also works, but its node export carries a `#F5F5F5` background rect
> plus artboard paths that must be stripped from the innermost `<g id="…">`, and it handles one
> node per call. Prefer the images endpoint.

The mirrored file is therefore **not** byte-identical upstream. The manifest records the upstream
identity and the transform applied so any copy can be regenerated and diffed. A local copy is
never evidence of what is upstream.

> Taking the tight-bbox variant is what produced the per-icon hand-tuned scale CSS in commit
> `88e7bce`. Icons on a shared 24×24 grid need no per-icon sizing.

---

### Verify the glyph, never trust the name

**Always render an icon and look at it before committing to it.** Names in the universal set do
not reliably describe the drawing:

- `report` and `reportOutline` are octagon-with-exclamation **alert** icons, not documents
- `promoEsx` is a badge/seal, not a page

Pull the candidates, render them side by side, and choose from the drawings.

---

### Icon System: Material Symbols Outlined

Material Symbols Outlined (Google Fonts variable font) covers generic UI affordances across all
digital products, design tooling, and internal dashboards.

#### Loading (web / Vercel)
```html
<link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@24,400,0,0" rel="stylesheet">
```

Do **not** append `&display=swap` — the font falls back to rendering the ligature name as literal
text ("picture_as_pdf") until it loads.

#### CSS baseline
```css
.icon {
  font-family: 'Material Symbols Outlined';
  font-variation-settings: 'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24;
  font-size: 24px;
  line-height: 1;
  user-select: none;
}
```

#### Standard axis settings
| Axis | Value | Notes |
|---|---|---|
| `FILL` | `0` | Outlined (unfilled). Use `1` for emphasis or active states only. |
| `wght` | `400` | Regular weight. Match surrounding text weight when inline. |
| `GRAD` | `0` | No grade adjustment. |
| `opsz` | `24` | Optical size. Use `20` for 16–18px inline icons. |

#### As inline SVG
Where the webfont is impractical — a single glyph, or a page already inlining universal-set icons —
fetch the official SVG at the standard axis config:

```
https://fonts.gstatic.com/s/i/short-term/release/materialsymbolsoutlined/<name>/default/24px.svg
```

`default` **is** `FILL 0 / wght 400 / GRAD 0 / opsz 24`. Lighter official weights exist as path variants: `…/<name>/wght300/24px.svg` and `wght200`, and the 20px optical size as `…/<name>/wght300/20px.svg` (verified 2026-10-06; `wght400/` and `wght300fill0/` 404). The brand kit's UI chrome (block tabs, preview-width toggle) uses `wght300` at 20px for a lighter line next to Plex.
Set `fill="currentColor"` and keep Google's `0 -960 960 960` viewBox — mixing it with 24×24
universal-set icons is fine, since each is sized to the same box.

#### Arrow set — standard usage
| Icon name | Use case |
|---|---|
| `arrow_outward` | External links, open in new tab, launch — **primary arrow** |
| `arrow_forward` | Navigate right, next step, proceed |
| `arrow_back` | Navigate left, previous, back |
| `arrow_upward` | Expand, scroll to top, increase |
| `arrow_downward` | Collapse, scroll down, decrease |
| `north_east` | Diagonal emphasis (decorative, large display) |
| `open_in_new` | Inline "open in new" in dense text contexts |

**Rule.** Always use `arrow_outward` for external link indicators. Never substitute Unicode arrows
(↗ ↑ →) or custom SVG arrows — they break visual consistency across products.

---

### Component icon map

Every component documented in `_Library Component Sets` that contains an icon has a canonical icon
name. Do not substitute a different icon for a documented component — use the one listed here.

| Component | Variant / part | Icon | Source | Size |
|---|---|---|---|---|
| Sticky CTA | icon button (all variants) | `arrow_forward` | Material Symbols | 24 |
| Navigation / Nav Item | Active state | `arrow_forward` | Material Symbols | 20 inline |
| External link (inline text) | — | `arrow_outward` | Material Symbols | 20 inline |
| Cards / Feature Card | "Learn more" affordance | `arrow_forward` | Material Symbols | 24 |
| Footer / social or partner links | external destinations | `arrow_outward` | Material Symbols | 24 |
| Workflow icons (Capture, Scope, Estimate) | custom — see `Icons / Workflow` set | — | Brand | 24 / 32 |

**Platform feature icons** — for pages and decks describing what a project unlocks:

| Feature | Icon | Source |
|---|---|---|
| 360° tours | `360°` | Universal |
| Timeline Tours | `timeline` | Universal |
| Compare Mode | `compare` | Universal |
| Field Notes | `noteOutline` | Universal |
| Structured room list | `sketchRoom` | Universal |
| Enhanced photo report | `picture_as_pdf` | Material Symbols — **fallback**, see below |

**Rule.** Pick from these tables when a matching pattern exists. If the pattern is genuinely new,
choose per the precedence table at the top and **report the gap** — do not add a row here. See the
hard rule under *Brand consumes read-only*.

**Sizes.** Components default to `24px` icons. For inline-text contexts (within 16–18px copy), use
`opsz: 20` with 20×20 dimensions so the icon aligns optically with the surrounding lowercase
x-height.

---

### Self-serve and gaps

Brand may self-serve where appropriate: when the universal set has no fitting icon, use Material
Symbols for the deliverable at hand rather than blocking on an upstream addition.

Two obligations come with it:

1. **Record the fallback** in the manifest under `fallbacks`, so the gap is visible rather than
   rediscovered each time.
2. **Report it in your response** so it can be picked up for upstream adoption offline. Never file
   it upstream yourself.

**Known gap.** The universal set has no PDF/file/export icon — zero of its 444 symbols match
`pdf`, `file`, `export`, or `print`. `picture_as_pdf` from Material Symbols is the standing
fallback for document-export concepts.

---

### Icon index — resolving a name to real markup

**Never invent an icon name, and never draw a substitute.** Every icon available to you is listed
below. If the concept you need is not here, say so in your response and fall back per the
precedence table above — do not improvise a glyph.

**How to get the actual SVG.** The registry
`assets/icons/manifest.json` maps every name to a file, a size and a viewBox. Resolve in this
order:

1. **Bundled with the skill** — `assets/icons/<size>/<name>.svg` relative to the skill directory.
   This is the offline path and the one to prefer.
2. **The published brand** — `https://brand-design-kit.vercel.app/<file>`, taking `<file>`
   verbatim from the manifest. Publicly readable, no auth. Use when the bundle is unavailable.

**Using one.** Inline the SVG rather than referencing it with `<img>`; every icon is normalised to
`fill="currentColor"`, so inlined it inherits the surrounding text colour and follows light and
dark automatically. An `<img>` cannot. Keep the upstream `viewBox` verbatim and size with CSS —
never set both `width` and `height` independently.

**Two exceptions that are multi-colour by design.** `check` and `cross` are two-tone status badges
carrying their own fore and background; they do not follow `currentColor`. Do not recolour them.

**One stroke icon.** `360aiLogo` (the 360AI logo, a modified 360 mark; brand-drawn, Chris Provins
2026-10-06) is a 1.25 stroke on `currentColor` in a `0 0 22 22` viewBox, not a fill: size it with CSS
and never add a fill. It is also the job-record hub at the centre of the hero's four nodes. It is
proposed as the full DocuSketch logo, but until that is ruled it is an icon, not the logo: the
Logo Usage rules (canonical wordmark, DS° mark) are unchanged.

**Sizes.** `24` is the default. `16` for dense UI and inline text — note the set uses a `Small`
suffix at that size. `32` for large display, and it covers only five concepts.

**24px — 201 icons**

`360aiLogo` · `360cam` · `360camConnected` · `360camDisconnected` · `360camFilled` · `360°` · `account`
`actions` · `add` · `add360°` · `addComment` · `addFilled` · `affected` · `alert`
`approval_delegation` · `archive` · `arrowDown` · `arrowLeft` · `arrowRight` · `arrowUp`
`barrier` · `batteryAlmostFull` · `batteryFull` · `batteryHalf` · `batteryHalfLow`
`batteryLow` · `billing` · `boxes` · `cabinets` · `calendar` · `card` · `carpentry`
`ceiling` · `check` · `checkmark` · `chevronDown` · `chevronLeft` · `chevronRight`
`chevronUp` · `circle` · `claimSummary` · `cleaning` · `close` · `colComment` · `comment`
`commentPrivate` · `commercial` · `compare` · `contentsFilled` · `contentsOutline`
`contract` · `copy` · `copyFrom` · `cross` · `cubicImage` · `delete` · `device_hub`
`deviceLaptop` · `devicePhone` · `deviceTablet` · `docuSketchMini` · `door`
`doubleChevronLeft` · `doubleChevronRight` · `download` · `drag` · `edit` · `electricity`
`emergency` · `environment` · `equipAirMover` · `equipAirScrubber` · `equipDehu`
`equipmentFilled` · `equipmentOutline` · `estimate` · `estimateInstant` · `existingDamage` · `feedback`
`filter` · `filterApplied` · `fire` · `flag` · `flash` · `flashOff` · `flood` · `floor`
`freehand` · `freeline` · `gears` · `geo` · `help` · `helpFilled` · `history` · `home_work`
`hvac` · `idea` · `image` · `information` · `kitchen` · `label` · `ladder` · `lidar` · `logOut`
`mapArea` · `mask` · `menu` · `menuNotification` · `message` · `messageBadge` · `micOff`
`micOn` · `migration` · `minus` · `mold` · `more` · `networkConnect` · `networkError`
`networkOffline` · `networkToUpload` · `networkUpload` · `networkUploading`
`networkUpToDate` · `noteOutline` · `notes` · `notification` · `notificationBadge`
`opening` · `overnighAlert` · `overnight` · `photoCamera` · `placeholder` · `playMedia`
`plumbing` · `plus` · `private` · `projectsList` · `promoEsx` · `promoEsxSow` · `promoGift`
`public` · `reconstruction` · `recordMic` · `recordPlayFilled` · `recordStopFilled`
`recordWave` · `rectangle` · `report` · `reportOutline` · `reshoot` · `residential`
`residentialFilled` · `scissors` · `scissorsCut` · `search` · `select` · `send` · `settings`
`settingsFilled` · `shareAndroid` · `shareOutline` · `shower` · `sketch` · `sketchInstant` · `sketchRoom`
`sofa` · `soundOff` · `soundOn` · `sow` · `star` · `stMinus` · `stPlus` · `success`
`supportMessage` · `text` · `timeline` · `transcribe` · `trauma` · `trolleyFilled`
`trolleyOutline` · `tutorial` · `unaffected` · `unpinned` · `usersList` · `vehicleImpact`
`visiblityOff` · `visiblityOn` · `wall` · `wallCut` · `wallMoisture` · `warningOutline`
`washingMachine` · `waterFilled` · `waterOutline` · `wifi` · `window` · `workflow-capture`
`workflow-estimate` · `workflow-scope` · `zoomIn` · `zoomReset`
**16px — 57 icons**

`360imageSmall` · `360°Small` · `affectedOutlineSmall` · `affectedSmall` · `audioNotesSmall`
`changeSmall` · `chevronDownSmall` · `chevronLeftSmall` · `chevronRightSmall`
`chevronUpSmall` · `clearSmall` · `closeSmall` · `commentsSmall` · `commercialSmall`
`contentsSmall` · `contractSmall` · `copySmall` · `cotalitySmall` · `deleteSmall`
`downloadSmall` · `editSmall` · `emergencyMitigationSmall` · `environmentSmall`
`equipmentFilledSmall` · `estimateSmall` · `fireOutlineSmall` · `flagSmall`
`informationSmall` · `messageBadgeSmall` · `messageSmall` · `notesSmall` · `openLinkSmall`
`overnightSmall` · `photoCameraSmall` · `placeholderSmall` · `playMediaSmall` · `plusSmall`
`predamagedSmall` · `privateSmall` · `promoGiftSmall` · `publicSmall`
`reconstructionGeneralSmall` · `reportSmall` · `reshootSmall` · `residentialSmall`
`shareOutlineSmall` · `sketchSmall` · `sowSmall` · `traumaSmall` · `unpinnedSmall`
`uploadSmall` · `vehicleImpactSmall` · `veriskSmall` · `warningFilledSmall` · `warningSmall`
`waterFilledSmall` · `waterOutlineSmall`
**32px — 5 icons**

`contentsFilledLarge` · `equipmentFilledLarge` · `notesLarge` · `reportLarge`
`waterFilledLarge`

#### Optical size check
Universal-set icons fill roughly **60–90%** of the 24×24 box. A Material Symbols glyph beside them
should land in that range — `picture_as_pdf` measures 83%, so it needs no rescaling. If a mixed row
looks uneven, measure the ink extents before reaching for a per-icon transform.

---

## Sync Metadata
<!-- AUTO-UPDATED BY figma-sync.py — DO NOT EDIT THIS SECTION MANUALLY -->
```json
{
  "figma_file_key": "JR35zTngKUblEKMD0myUyD",
  "sync_user": "provins",
  "sync_user_email": "chris.provins@docusketch.com",
  "last_figma_sync": "2026-10-07T22:27:05.802465+00:00",
  "last_skill_sync": "2026-10-09T15:04:29.596500+00:00",
  "figma_last_version": "2407744953440500267"
}
```

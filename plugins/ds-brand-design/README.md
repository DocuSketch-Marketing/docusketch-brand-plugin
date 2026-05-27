# DocuSketch Brand Design (Claude Code plugin)

The DocuSketch brand design system as an installable Claude Code skill. Once installed, Claude automatically applies brand tokens, typography, spacing, radius, component IDs, accessibility rules, and the Best Practices checklist whenever you work on anything visual for DocuSketch, and you can also invoke it explicitly with `/ds-brand-design:ds-brand-design`.

The full reference (`DS_Brand_Design_Skill.md`) is bundled inside the skill, so it works on any machine with no extra setup. It is generated from the Brand Design Kit Figma file by `figma-sync.py` and shipped with the plugin.

## Install

```
/plugin marketplace add ds-provins/brand-design-kit
/plugin install ds-brand-design@docusketch-brand
```

Replace `ds-provins/brand-design-kit` with this repository's GitHub `owner/repo` if it differs. The part after `@` (`docusketch-brand`) is the marketplace name, defined in `.claude-plugin/marketplace.json` at the repo root.

## Update to the latest brand sync

```
/plugin marketplace update
/reload-plugins
```

## What you get

- Color tokens (primitives + semantic), with Pantone and WCAG data
- Typography styles (IBM Plex families, sizes, weights, plain-zero rule)
- Spacing scale (4px grid), radius scale, elevation tokens
- Motion tokens and canonical patterns
- Figma component IDs and file references
- The Best Practices checklist

Live dashboard: https://brand-design-kit.vercel.app

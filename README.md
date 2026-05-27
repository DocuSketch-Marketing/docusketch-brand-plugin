# DocuSketch Brand Design — Claude Code plugin

This is the public install source for the DocuSketch brand design system as a Claude Code plugin. Install it once and Claude applies the brand (color tokens, typography, spacing, radius, components, accessibility, and the Best Practices checklist) automatically, in any project.

No coding required. The full reference is bundled in the plugin, so it works on any machine with nothing else to set up.

## Install (paste into Claude Code)

```
/plugin marketplace add ds-provins/docusketch-brand-plugin
/plugin install ds-brand-design@docusketch-brand
```

That's it. Claude now follows the brand on its own. You can also call it directly:

```
/ds-brand-design:ds-brand-design
```

## Update later

When the brand kit changes, pull the latest:

```
/plugin marketplace update
/reload-plugins
```

## What's inside

| Path | What it is |
| :--- | :--- |
| `.claude-plugin/marketplace.json` | The marketplace catalog (name: `docusketch-brand`). |
| `plugins/ds-brand-design/` | The plugin: manifest, skill, and the bundled brand reference. |
| `plugins/ds-brand-design/skills/ds-brand-design/DS_Brand_Design_Skill.md` | The canonical brand doc, generated from Figma and shipped with the plugin. |

The brand doc is generated from the Brand Design Kit Figma file and kept in sync by `figma-sync.py` in the (private) `brand-design-kit` repo. Live dashboard: https://brand-design-kit.vercel.app

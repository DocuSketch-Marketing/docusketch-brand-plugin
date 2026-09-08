# DocuSketch Brand Design — Claude Code plugin

This is the public install source for the DocuSketch brand design system as a Claude Code plugin. Install it once and Claude applies the brand (color tokens, typography, spacing, radius, components, accessibility, and the Best Practices checklist) automatically, in any project.

No coding required. The full reference is bundled in the plugin, so it works on any machine with nothing else to set up.

## Install (paste into Claude Code)

```
/plugin marketplace add DocuSketch-Marketing/docusketch-brand-plugin
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
| `plugins/ds-brand-design/skills/ds-brand-design/DS_Brand_Design_Skill.md` | The brand doc itself — generated, never hand-edited here. |
| `plugins/ds-brand-design/org-skill/SKILL.md` | Wrapper for the uploaded org skill (`docusketch-brand`), versioned here so it does not live only inside an installed copy. |
| `CHANGELOG.md` | What changed in the bundled doc between installs. |

## How it gets published

The doc is hand-authored (tokens, rules) plus a Figma-generated component
inventory, and lives canonically at
`~/Library/Application Support/docusketch/sync/DS_Brand_Design_Skill.md` on the
design owner's machine. `figma-sync.py` in the `brand-design-kit` repo runs every
15 minutes and fans it out:

| Consumer | Published by |
| :--- | :--- |
| `brand-design-kit` repo + dashboard | committed with the daily health snapshot |
| **this repo** (plugin installs) | committed and pushed automatically by the sync |
| org skill `docusketch-brand` (claude.ai → Settings → Skills) | **by hand — no API exists.** The sync stages a ready-to-upload bundle and writes a `PUBLISH_OWED` marker when it falls behind. |

Run `figma-sync.py --publish-status` to see canonical vs. every published copy.
Live dashboard: https://brand-design-kit.vercel.app

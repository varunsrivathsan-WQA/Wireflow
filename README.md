# Wireflow Wireframe

A Claude Code / Cowork plugin that builds lo-fi wireframes as matched **Figma + HTML/CSS** deliverables from a plain-English brief, using WQA's 66-primitive Wireflow design system (grayscale + one link blue, real Lucide icons, shadcn/ui-parity radii).

## What it does

Ask Claude for a wireframe ("wireframe a checkout flow", "mock up a SaaS pricing page") and this plugin's `wireflow` skill will:

1. Confirm which Figma file to build into, and check that file's component library and the bundled CSS agree with each other before building anything.
2. Map every element in your brief to one of Wireflow's 66 documented primitives (never an invented shape or class).
3. Build a real Figma frame out of actual component instances — proper auto-layout, real variant properties, nothing hand-drawn.
4. Build a matching self-contained HTML file using the same primitives' CSS classes.
5. For a multi-screen flow ("wireframe the checkout flow"), link the screens into a real click-through prototype — actual `href`s between the generated HTML files plus matching Figma prototype connections, no JavaScript involved on either side.
6. Apply layout-sizing discipline so nothing overflows its container — text fills its card instead of spilling past the edge, a row of cards renders equal-height, a button/toggle group is sized to fit its parent instead of blowing past it, and an image gallery is built as one composed pattern (Carousel-plus-thumbnails, or a single static image) rather than both at once.
7. Report back both deliverables, flagging anything it wasn't sure how to map.

## Requirements

This skill needs the Wireflow component library available to whatever Figma file you're building into — all 66 primitives plus the Icon component set. It checks for this automatically and picks one of two paths:

- **If Wireflow is published as a team/org library** (Figma → Assets panel → Publish, a one-time action, private to your org — not the public Figma Community), the skill pulls primitives directly from the library into any file as needed. No manual duplication required.
- **If it isn't published yet**, the skill falls back to pointing you at the canonical reference file and asking you to duplicate/save a copy before it builds the Figma side (it can still produce the HTML/CSS side on its own either way).

Publishing the reference file as a library once means every future build skips the duplication step entirely — worth doing if you'll be using this plugin regularly.

## What's included

- `skills/wireflow/SKILL.md` — the full build process: primitive vocabulary, token rules, icon handling, Figma structural-quality requirements (auto-layout, true variant component sets), and the hard rules that keep both outputs in sync.
- `skills/wireflow/references/wireflow-tokens.css` — the canonical design tokens (colors, type scale, spacing, radius) that every primitive references.
- `skills/wireflow/references/wireflow.css` — the full primitive stylesheet (one `.wf-*` class per primitive).
- `skills/wireflow/references/catalog-artifact.html` — a self-contained, standalone preview of all 66 primitives. Open it directly in a browser (or download and double-click it) — no other files needed.

## Installing

Drag the `.plugin` file into Claude Code or Cowork, or add this repo as a marketplace source and install `wireflow` from it.

## Updating

This plugin bundles its own copies of the two CSS files so it works standalone. If the canonical Wireflow token/primitive files change, refresh the copies under `skills/wireflow/references/` and bump the version in `.claude-plugin/plugin.json` before re-publishing.

## Marketplace `source` field format

If this repo is ever used as a template for another self-hosted marketplace, note that `.claude-plugin/marketplace.json`'s `plugins[].source` field must be the nested object form:

```json
"source": {
  "source": "github",
  "repo": "owner/repo"
}
```

A bare `"owner/repo"` string, a bare `"."`, or `{"repo": "."}` all fail marketplace sync with a generic "Marketplace sync failed. Check the repository URL and try again." error that gives no indication the `source` field's shape is the problem. This nested shape was confirmed by inspecting `~/.claude/settings.json`'s `extraKnownMarketplaces` entries for already-working marketplaces (e.g. `claude-plugins-official`), which use this exact structure — it isn't documented anywhere obvious, so don't rediscover this the hard way again.

Also worth knowing: Cowork's marketplace importer reads whichever branch GitHub reports as the repo's **default branch** (same as an unauthenticated `git clone` with no branch specified), not necessarily `main`. If you ever rename a branch, confirm the new one is actually set as the default under **Settings → Branches** — a rename alone doesn't move the default pointer, and a stale old default branch will make every sync attempt silently validate against outdated content.

---
name: hermes-dashboard-fonts
description: Replace the Hermes Agent dashboard's stylized display fonts with system fonts; for the web UI looking jarring or hard to read
---

# Hermes Agent dashboard — replace the display fonts

**TL;DR:** Append a font override to the end of the dashboard app's own `web/src/index.css`, then rebuild — overriding at the `@font-face` level, not chasing individual selectors.

## Symptom

The official Hermes Agent web dashboard (`hermes dashboard`) renders headings, navigation, and UI chrome in stylized display fonts — an over-wide "expanded" heading face, a pixel-style face, etc. It reads as a retro/terminal aesthetic that some find jarring or hard to scan. The fonts load fine; the issue is purely the typeface choice.

## Cause

The display fonts are not in the dashboard app's own CSS. They ship with NousResearch's design-system package, `@nous-research/ui`. The app's `web/src/index.css` actually defines clean system-font defaults (`--theme-font-sans`), but near its top it does `@import '@nous-research/ui/styles/globals.css'`, which brings in `@font-face` declarations for faces like `Collapse`, `Rules Expanded`, `Rules Compressed`, and `Mondwest`, plus a `ThemeProvider` that applies them.

So the weird fonts are an intentional design-system choice — not a bug, and not a missing-font fallback. "Fixing" means overriding the design system, not repairing it.

## Fix

Append an override block to the **end** of `web/src/index.css` (default location `~/.hermes/hermes-agent/web/src/index.css`). Because `index.css` imports the design-system CSS near its top, anything appended later wins the cascade.

Override in two layers, because the display fonts are referenced two different ways:

1. **CSS variables** — catches components that use `var(--font-sans)` etc.
2. **`@font-face` remapping** — re-declare each display-font family so the *name itself* resolves to a local system font. This is the robust layer: it does not matter which selector or component uses `font-family: "Rules Expanded"`, the family now resolves to a normal font.

Leave `JetBrains Mono` alone — it is a normal, readable monospace face and the embedded terminal uses it.

```css
/* ─── font override ─── system fonts over @nous-research/ui display fonts */
:root {
  --theme-font-sans: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif !important;
  --theme-font-display: var(--theme-font-sans) !important;
  --font-sans: var(--theme-font-sans) !important;
  --font-display: var(--theme-font-sans) !important;
}
@font-face { font-family: "Collapse";         src: local("Helvetica Neue"), local("Segoe UI"), local("Roboto"), local("Arial"); font-weight: 100 900; font-style: normal; }
@font-face { font-family: "Rules Expanded";   src: local("Helvetica Neue"), local("Segoe UI"), local("Roboto"), local("Arial"); font-weight: 100 900; font-style: normal; }
@font-face { font-family: "Rules Compressed"; src: local("Helvetica Neue"), local("Segoe UI"), local("Roboto"), local("Arial"); font-weight: 100 900; font-style: normal; }
@font-face { font-family: "Mondwest";         src: local("Helvetica Neue"), local("Segoe UI"), local("Roboto"), local("Arial"); font-weight: 100 900; font-style: normal; }
```

Rebuild the dashboard's static assets and restart it:

```bash
cd ~/.hermes/hermes-agent/web && npm run build
hermes dashboard --stop   # then relaunch it (directly, or restart the service it runs under)
```

Verify in the browser DevTools console:

```js
getComputedStyle(document.documentElement).getPropertyValue('--theme-font-sans')  // -> the system stack
document.fonts.check('20px "Rules Expanded"')                                     // -> false (display face no longer used)
```

## Notes

- **Why edit `web/src/index.css` and not the build output or the package.** Editing the app's own source means the override is re-baked on every `npm run build`. Patching the built `web_dist/` or `node_modules/@nous-research/ui` instead is wiped by the next build or `npm install`.
- **It does not survive a Hermes version update.** A version bump that checks out new code (`git checkout <tag>`, `hermes update`) replaces `index.css`. Re-append the block and rebuild afterward. Mark it with a comment and `grep` for that marker before appending, so re-runs do not duplicate it.
- **`@font-face` cascade.** When two `@font-face` rules declare the same family with overlapping weight/style, the **last one wins** — so the override must come after the design-system `@import`, which appending to the end of `index.css` guarantees. `font-weight: 100 900` is a range covering every weight the design system defines.
- **Graceful degradation.** If `local()` matches none of the listed fonts (e.g. a Linux browser without them), the family resolves empty and the browser falls back to the generic in the declaration (`sans-serif`) — still a normal font.
- **Future-proofing.** Font names verified against Hermes Agent v2026.5.x. NousResearch may rename or add display faces in later releases; if a weird font reappears after an update, run `[...document.fonts].map(f => f.family)` in DevTools to read the current family names and extend the `@font-face` list.

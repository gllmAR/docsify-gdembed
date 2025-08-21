(# docsify-gdembed)

Docsify plugin that finds embed markers in your Docsify pages and inserts interactive Godot demo iframes (with mobile-friendly fullscreen, pop-out and expanded-view controls).

## What this plugin does

- Scans page content for HTML comment markers such as `<!-- embed-{$PATH} -->`, `<!-- embed-{project/path} -->`, and legacy `<!-- embed-projectName -->` markers.
- Builds a best-effort URL pointing at a project's `exports/web` folder (or the shared gdEmbed exports), then injects a demo container with an iframe and overlay controls.
- Adds mobile-friendly fullscreen behavior (CSS-based expanded view), native fullscreen attempts on desktop, a pop-out button and touch gestures (double-tap) for mobile.

## Quick install

1. Copy `docsify-gdembed.js` into your Docsify project's assets and include it on your Docsify page (before Docsify init):

```html
<script src="/path/to/docsify-gdembed.js"></script>
```

The script auto-registers as a Docsify plugin on load; no additional config is required.

## Marker formats and examples

The plugin recognizes several marker styles. Insert any of these as HTML comments in your Markdown where you want the demo to appear.

- Embed current scene/project using the current page path:

```html
<!-- embed-{$PATH} -->
```

This resolves the current Docsify hash and attempts to construct a sensible demo URL for either a shared `gdEmbed` layout or an individual project layout.

- Embed a specific project path (direct path to project root or repo subpath):

```html
<!-- embed-{some/repo/path} -->
```

This will be turned into `<baseUrl>/some/repo/path/exports/web/`.

- Legacy/simple embed form (keeps backward compatibility):

```html
<!-- embed-projectName -->
<!-- embed-projectName: gdEmbed/scenes/category/scene_name -->
```

The plugin supports a few legacy patterns but prefers explicit `{...}` or `{$PATH}` markers when possible.

## How the path is determined

The plugin tries multiple strategies (see `getCurrentScenePath()` in `docsify-gdembed.js`):

- Recognize `gdEmbed/scenes/<category>/<scene>` patterns.
- Recognize common three-segment paths like `repo/category/project` and map to `category/project`.
- Fallback: use the last two path segments from the current hash.

It normalizes hashes (removes `.md`, `README`/`index`, query/fragments) and can append `?scene=` to exports URLs when configured.

## Options

- ALLOW_URL_SCENE_ARG (in `docsify-gdembed.js`): boolean (default: true)
	- When true the plugin appends `?scene=<category/scene>` to certain constructed demo URLs so the exported web build can open a specific scene.

If you need different behavior, edit the top of `docsify-gdembed.js` (the configuration section) before including it.

## Controls and UX

- Desktop: overlay buttons for expanded view, native fullscreen (tries iframe then container) and pop-out (new window).
- Mobile: expanded view (CSS-based mobile fullscreen), pop-out opens new tab, double-tap toggles fullscreen.
- Escape key exits mobile-style fullscreen when active.

## Troubleshooting

- Blank iframe / 404: confirm the constructed demo URL points to an `exports/web/` directory that contains an exported Godot HTML5 build (index.html). Check the browser console logs printed by the plugin (it logs its decisions).
- Hash formats: this plugin expects Docsify-style hashes (the part after `#`). If your Docsify setup rewrites routes or you use pretty URLs, the automatic extraction might fail—use explicit `<!-- embed-{owner/repo/path} -->` markers instead.
- Cross-origin: ensure the export hosting allows being framed (X-Frame-Options header) or open as a pop-out.

## Internals (short analysis)

- Entry: plugin registers via `window.$docsify.plugins` and runs `initializePlugin` on `doneEach` and on `hashchange`.
- Marker discovery: uses an XPath query to find HTML comment nodes that contain `embed-`.
- Creation: builds a `.demo-container` with an `<iframe>` and overlay controls; `setupDemoControls()` wires buttons and touch listeners.
- Fullscreen handling: attempts native iframe fullscreen APIs (`requestFullscreen`, vendor prefixes) and falls back to a CSS mobile-style fullscreen that toggles `.mobile-fullscreen` on the container.

Contract (inputs/outputs, error modes)

- Inputs: Docsify page HTML with embed comment markers; current window.location.hash; ALLOW_URL_SCENE_ARG boolean.
- Output: injected DOM `.demo-container` containing iframe pointing at a guessed `exports/web` URL.
- Error modes: unable to resolve scene path (plugin inserts a small inline error), iframe blocked by X-Frame-Options, or resource 404.

Edge cases to be aware of

- Non-standard routing / pretty URLs may break scene path extraction.
- Multiple identical markers on the same page: dedup logic uses a coarse marker id; markers very close in time may not dedupe perfectly.
- Cross-origin framing restrictions may prevent embedding—pop-out is the fallback.
- Very strict mobile browsers may prevent programmatic scroll/resize to hide address bars.

## Development & debugging tips

- Open the browser console: the plugin logs extraction steps, constructed URLs and warnings.
- To force a particular demo URL during development, use the explicit `<!-- embed-{path} -->` marker.
- If you change `docsify-gdembed.js`, be sure to clear Docsify cache (or reload with cache disabled) and navigate to the affected page.

## License

This repository includes a `LICENSE` file. Follow the project license for reuse and distribution.

## Status

- Analysis of plugin behavior: Done
- README created and added to repository: Done

If you want, I can also:

- add a simple example page into the repo showing all marker types,
- or add a small test harness that validates URL construction logic for several hash inputs.


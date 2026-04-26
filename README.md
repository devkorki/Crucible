# Crucible

A recipe graph visualizer for [Infinite Craft](https://neal.fun/infinite-craft/). Drop your `.ic` save file in and see your discoveries as a connected node map — colour-coded by depth from the four base elements.

![screenshot placeholder — replace with your own]()

## What it does

- Reads the **gzipped JSON** `.ic` save file the game exports from Settings
- Resolves the integer-ID recipes back to ingredient names
- Lays out the result as a **force-directed graph** (drag, zoom, pan)
- Persists everything to `localStorage` so it survives reloads
- Round-trips back to `.ic` so you can re-import into the game

## Deploy to GitHub Pages

The whole app is a single self-contained `index.html`. No build step.

1. Create a new repository on GitHub (public, since Pages needs that on free accounts).
2. Upload `index.html` to the root of the repo.
3. Repo → **Settings** → **Pages** → **Source**: pick `main` branch, `/ (root)` folder. Save.
4. Wait ~30 seconds. Your app is live at `https://<your-username>.github.io/<repo-name>/`.

That's it. Push updates to `main` to redeploy.

## Or run it locally

Just open `index.html` in a browser. Everything's on CDN — no install.

If your browser blocks the file due to mixed-content rules, serve it over a tiny local server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## How it's built

`index.html` includes everything in one file:

- **React 18** + **ReactDOM** from `unpkg.com` (UMD globals)
- **D3 v7** from `jsdelivr.net` for the force simulation
- **Tailwind Play CDN** for styling (JIT-compiles classes in the browser)
- **Babel standalone** to transpile the JSX in a `<script type="text/babel">` block
- **Lucide-style icons** inlined as small SVG components (no extra requests)
- **Fraunces / Geist / JetBrains Mono** loaded from Google Fonts

The page is ~70 KB; total over the wire after CDN payloads is around 4 MB on first load (mostly Babel and Tailwind). Subsequent loads are cached.

## Going production

The Babel + Tailwind Play CDN combo is fine for personal use but not ideal for a public app — Babel transpiles every page load. To upgrade:

1. Move the React component (the inline `<script type="text/babel">` block) into a real React project (Vite, Next.js, etc.).
2. Install `tailwindcss`, `lucide-react`, `d3` properly.
3. Replace the `window.storage` shim with direct `localStorage` calls (or whatever storage layer fits your app).

The component itself doesn't need any other changes.

## Notes on the `.ic` format

The save file is gzipped JSON with this shape:

```json
{
  "name": "Save 1",
  "version": "1.0",
  "created": 1777196910083,
  "updated": 1777196932427,
  "instances": [],
  "items": [
    { "id": 0, "text": "Water", "emoji": "💧" },
    { "id": 4, "text": "Lava", "emoji": "🌋", "recipes": [[1, 3]] }
  ]
}
```

`recipes` is an array of `[ingredientIdA, ingredientIdB]` pairs — an item can have multiple. Base elements (Water, Fire, Wind, Earth) have no `recipes` field. The `instances` array is for the Infinite Canvas mode (positions of placed items) and is ignored by this tool.

When exporting back to `.ic`, items are renumbered in topological order (by depth, then name) so every recipe references IDs lower than its result — required by the format.

## License

MIT. Do whatever.

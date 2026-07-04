# WordTally — Agent / Contributor Guide

## What this is

WordTally is a **standalone**, single-purpose Astro site: a live word,
character, and sentence counter with reading time, speaking time, and
keyword density. It was split out of the multi-tool "anyconvert" site.

**This is deliberate.** The SEO strategy for this tool family is one
keyword-matched domain per tool — a visitor searching "word counter" lands
on a site that is ONLY a word counter, not a grab-bag utility dump with
unrelated tools cluttering the nav. Do not add other tools to this repo.
Do not add a nav to sibling tools. If a new tool is needed, it gets its
own standalone project under `~/iCloud/website/<tool-name>/`.

## Stack

- **Astro** (static output, no SSR adapter) — `astro build` emits to `dist/`.
- **Tailwind v4** via `@tailwindcss/vite` (no `tailwind.config.js` — v4 is
  CSS-first; theme tokens live in `src/styles/global.css` under `@theme`).
- **@astrojs/sitemap** for `sitemap-index.xml` (site URL set in
  `astro.config.mjs`).
- 100% client-side logic. No backend, no database, no API calls. The word
  counter runs entirely in a `<script>` block in `src/pages/index.astro`.
- No extra runtime dependencies beyond Astro + Tailwind — this tool needs
  none of anyconvert's heavier deps (ffmpeg, transformers, etc.).

## Design system (do not deviate)

| Token | Value | Use |
|---|---|---|
| Background | `#FAF6EC` | page background (warm cream) |
| Surface | `#FFFFFF` | cards |
| Border | `#E8DFC8` | warm border on all cards/inputs (never gray) |
| Heading text | `#2B2013` | warm espresso brown |
| Body text | `#5C4F3D` | warm brown-gray |
| Accent | `#C9982E` | links, CTAs, active/focus states |
| Accent hover | `#B8860B` | darker gold on hover |

- Headings: **Fraunces** (soft-serif/display), weight 600-700, slightly
  negative letter-spacing. Loaded via Google Fonts `<link>` in
  `Layout.astro` head with `display=swap`.
- Body/UI/buttons: **Inter**.
- Rounded-xl corners, soft shadows only (`shadow-sm`), no gradients, no
  dark mode. Cream-only aesthetic is intentional — do not add a theme
  toggle or dark variant.
- Should read as a small, confident, premium product — not a generic
  free-tool dump.

## Pages

- `/` — the tool itself (`src/pages/index.astro`)
- `/about` — what/why
- `/privacy` — privacy policy (AdSense eligibility requirement)
- `/faq` — FAQ (AdSense eligibility requirement)
- `/404` — custom not-found page

All four utility pages are linked from the homepage footer (and `/` also
deep-links to about/privacy/faq inline) — required for AdSense approval
across this whole tool family.

## Security standard (applies across the tool family)

Any user-controlled text that gets inserted into the DOM must be passed
through an `escapeHtml()`-style helper first, even when using
`textContent` (kept consistent as a defensive standard, not because
`textContent` alone is unsafe). See the `escapeHtml` function in
`src/pages/index.astro`'s inline script.

## iCloud sync hygiene

`node_modules` is renamed to `node_modules.nosync` with a symlink named
`node_modules` pointing at it, so iCloud does not attempt to sync
hundreds of thousands of small package files (which chokes iCloud sync
and corrupts state across devices).

**Known gotcha:** `astro check` / the TS server will crash trying to walk
into `node_modules.nosync` unless it's excluded. `tsconfig.json`
`exclude` includes BOTH `"node_modules"` and `"node_modules.nosync"`.

To regenerate on a fresh checkout (e.g. on the other Mac):

```bash
cd "~/Library/Mobile Documents/com~apple~CloudDocs/website/wordtally"
npm install
mv node_modules node_modules.nosync
ln -s node_modules.nosync node_modules
```

## Dev

```bash
npm run dev      # http://localhost:4358
npm run build    # -> dist/
npm run preview  # serves dist/ on :4358
```

Port **4358** was picked to avoid colliding with sibling standalone tools
being built concurrently at the same time as this one (a live port-collision
was hit and fixed mid-build — see the git/session history). Before reusing
a port for a new sibling tool, check what's actually listening with
`lsof -i :<port>`, not just what other package.json files claim, since
concurrent agents can grab a port before you do.

## Deploy (Netlify)

```bash
source ~/.claude/credentials/netlify.env
export NETLIFY_AUTH_TOKEN="$NETLIFY_API_KEY"
netlify sites:create --name wordtally --disable-linking
# write .netlify/state.json with the returned site ID, then:
netlify deploy --prod --dir=dist
```

Do not point this at any other project's Netlify site. This is its own
site with its own URL.

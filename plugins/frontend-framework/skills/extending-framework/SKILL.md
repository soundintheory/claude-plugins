---
name: extending-framework
description: How to extend the vanilla-CSS + Vite framework itself — add a CSS module, add a JS behaviour module, add design tokens or breakpoints, and the Vite / PostCSS build internals (entry globbing, @custom-media, color-mix, native nesting, cache-busting, static copy). Use when changing the build, adding reusable modules, or adjusting tokens/breakpoints. Read framework-overview first.
version: 1.0.0
---

# Extending the framework

How to add to or modify the framework. Read framework-overview first.

## Add a CSS module (own chunk, lazy-loadable)

1. Create `resources/assets/css/modules/<name>.css`.
2. Start with `@import '../abstract/all.css';` to get tokens + breakpoints
   (deduped at build — emits no duplicate rules).
3. Write styles using native nesting, `var(--token)`, and `@media (--md-up)`.

```css
@import '../abstract/all.css';

.my-block {
    padding: 1rem;
    background: var(--color-bg);

    @media (--md-up) { padding: 2rem; }

    .my-block__title { color: var(--color-primary); }
}
```

It builds automatically (Vite globs `css/modules/*.css`) to
`/assets/css/modules/<name>.css`. To use it, add that URL to `window.lazyStyles`
on pages that need it (see building-pages).

## Add a JS module (own chunk, lazy behaviour)

1. Create `resources/assets/js/modules/<name>.js`.
2. Export `init(element)` (called by `app.js` with the declaring element).
   Optionally export a default for direct import/testing.

```js
class MyThing {
    constructor(el) { this.el = el; /* … */ }
}
export function init(el) { return new MyThing(el); }
export default MyThing;
```

Use in markup via `data-module="<name>"` (eager) or
`data-module-lazy="<name>"` (on scroll). It builds automatically (Vite globs
`js/modules/*.js`). Third-party libs: `npm install` then `import` them in the
module — Vite bundles them into that module's chunk.

## Add / change design tokens

Edit `resources/assets/css/abstract/tokens.css`. Add to `:root`:

```css
:root { --color-accent: #ff6600; }
```

Then `var(--color-accent)` anywhere. Theming/overrides: redeclare the token on a
narrower scope (a `.theme-dark` class, a component root). Replacements for Sass
colour functions:
- `darken($c, 5%)`  → `color-mix(in srgb, var(--c), black 5%)`
- `lighten($c, 5%)` → `color-mix(in srgb, var(--c), white 5%)`
- `rgba($c, .2)`    → `color-mix(in srgb, var(--c), transparent 80%)`

## Add / change a breakpoint

Edit `resources/assets/css/abstract/breakpoints.css`:

```css
@custom-media --xxl-up (min-width: 120rem);
```

Use as `@media (--xxl-up) { … }`. Values are `rem` (16px base). Keep the
`-up / -only / -down` naming for consistency with the existing set. Remember:
media queries can't read `var()`, so breakpoints MUST be `@custom-media`, not
custom properties.

## Where things import (cascade order)

- `app.css` (eager): `abstract/all.css` → `base/all.css` →
  `components/required.css`. Add an eager component to `components/required.css`.
- `app.lazy.css` (post-paint): re-imports `abstract/all.css` then non-critical
  components. Add post-paint styles here.
- `abstract/all.css` = tokens + breakpoints only (idempotent — safe to import in
  every module). Utility *classes* live in `abstract/utilities.css` and are
  imported once via `base/all.css`, NOT in `all.css` (they emit real rules).

## Build internals

**`vite.config.js`** — `root: resources`, `outDir: ../wwwroot`. Globs
`js/modules/*.js` and `css/modules/*.css` into separate Rollup inputs (so each
is its own chunk), plus `app.js`/`app.css`/`app.lazy.css`. `cssCodeSplit: true`.
`~` alias → `resources/assets`. Asset filenames routed to
`assets/{images,fonts,css,js}/`. `emptyOutDir: false` (won't wipe sibling files
in `wwwroot`).

**`postcss.config.js`** — order matters:
1. `postcss-import` — inline `@import` partials (must run first so imported
   `@custom-media` defs are visible).
2. `postcss-custom-media` — resolve `(--md-up)` → real media queries.
3. `autoprefixer` — vendor prefixes per `.browserslistrc`.
Minification is Vite's job on `build`, not PostCSS.

**`vite-plugin-query-string-hash.js`** — appends `?v=<hash>` to asset URLs in
the manifest for cache-busting, then strips it from the on-disk filename. Leave
as-is.

**`vite-plugin-static-copy`** — copies `fonts/ images/ svg/ favicons/` verbatim
to `wwwroot/assets/`. Reference them with absolute `/assets/...` URLs (e.g. in
`@font-face` in `components/fonts.css`).

**`.browserslistrc`** — baseline for autoprefixer. Native CSS nesting and
`color-mix()` are emitted un-transpiled, so the baseline assumes browsers that
support them. To support older browsers, add `postcss-nesting` (flattens
nesting) to the PostCSS chain and widen the browserslist; `color-mix` would need
precomputed fallbacks.

## Gotchas

- New CSS/JS module files are picked up by the glob only on (re)start of Vite —
  restart `dev`/`watch` after adding a file.
- Don't put real rules in `abstract/all.css` — it's imported many times.
- Keep `app.css` lean; push below-the-fold/interaction styles to `app.lazy.css`
  or a module CSS file.
- Verify a build after changes: `npm run build`, then check the output in
  `wwwroot/assets/css/app.css` resolved `@custom-media` to real `min-width`
  queries and kept `color-mix(...)`.

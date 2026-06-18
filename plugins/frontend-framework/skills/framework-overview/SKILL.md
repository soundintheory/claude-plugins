---
name: framework-overview
description: Architecture and conventions of the Sound in Theory vanilla-CSS + Vite frontend framework. Use this FIRST whenever building a site on this framework — it explains the folder layout, the build pipeline, the design-token / @custom-media system, and how the JS module loader works. Then use building-pages to author HTML or extending-framework to add modules; for house code style use html-conventions, css-conventions, js-conventions.
version: 1.0.0
---

# Frontend framework — overview

A vanilla-CSS (no Sass) + Vite + PostCSS frontend framework. Source lives in
`resources/`, builds to `wwwroot/assets/`. CSS uses native nesting, CSS custom
properties for design tokens, and `@custom-media` for breakpoints. JS is a small
lazy-loading module loader (`app.js`) plus on-demand ES modules.

## Where the framework lives (scaffold a project)

This plugin is documentation; the buildable framework source lives in its own
repo. To start a new site, scaffold from:

> **https://github.com/soundintheory/frontend-framework**

Clone it (or copy its `resources/`, `vite.config.js`, `postcss.config.js`,
`package.json`, `.browserslistrc`) into the project, `npm install`, then
`npm run build`. Everything below describes that repo's layout.

## Folder layout

```
resources/assets/
├── css/
│   ├── abstract/
│   │   ├── tokens.css        :root custom properties (colours, fonts, spacing)
│   │   ├── breakpoints.css   @custom-media breakpoint definitions
│   │   ├── utilities.css     utility CLASSES (visually-hidden, fill-absolute…)
│   │   └── all.css           aggregator → tokens + breakpoints (import anywhere)
│   ├── base/
│   │   ├── reset.css body.css text.css grid.css a.css
│   │   ├── button.css forms.css media.css
│   │   └── all.css           aggregator for the base layer
│   ├── components/
│   │   ├── fonts.css header.css footer.css   (skeleton stubs)
│   │   ├── hover-effects.css                 (lazy, non-critical)
│   │   └── required.css      aggregator of eager components
│   ├── modules/              one file per lazy-loadable component → own CSS chunk
│   │   └── example.css
│   ├── app.css               EAGER entry  (abstract + base + required components)
│   └── app.lazy.css          LAZY entry   (injected after first paint)
├── js/
│   ├── build/vite-plugin-query-string-hash.js
│   ├── modules/              one file per behaviour → own JS chunk (lazy)
│   │   └── example.js
│   └── app.js                the module loader (entry)
├── fonts/ images/ svg/ favicons/   copied verbatim to wwwroot/assets/
```

Build config at repo root: `vite.config.js`, `postcss.config.js`,
`.browserslistrc`, `package.json`.

## The three big conventions

1. **Design tokens are CSS custom properties** (`abstract/tokens.css`), not Sass
   variables. Reference them with `var(--color-primary)` etc. Override them by
   redeclaring on a scope (`:root`, a theme class, a component root).

2. **Breakpoints are `@custom-media`** (`abstract/breakpoints.css`). This is the
   replacement for the old Sass media-query mixins. Write real media queries
   that reference a named query:

   ```css
   .thing {
       padding: 1rem;
       @media (--md-up) { padding: 2rem; }   /* was @include md-up */
   }
   ```

   `postcss-custom-media` resolves `(--md-up)` to `(min-width: 48rem)` at build
   time. Media queries cannot read `var()` in browsers — this is why breakpoints
   are `@custom-media` and not custom properties. Names mirror the old mixins:
   `--xs`, `--sm-up/--sm-only/--sm-down`, `--md-*`, `--lg-*`, `--xl-*`, `--xxl-*`,
   `--retina`.

3. **JS behaviour is attribute-driven and lazy.** Put `data-module="name"` on an
   element to init a module on load, or `data-module-lazy="name"` to init it when
   it scrolls into view. See building-pages.

## Build pipeline (what happens on `npm run build`)

- Vite globs every `js/modules/*.js` and every `css/modules/*.css` into separate
  entries (own chunks), plus `app.js`, `app.css`, `app.lazy.css`.
- PostCSS runs (`postcss.config.js`): `postcss-import` inlines `@import`
  partials → `postcss-custom-media` resolves breakpoints → `autoprefixer`.
- Vite minifies, code-splits CSS, writes to `wwwroot/assets/`, and emits
  `assets/manifest.json`.
- The `query-string-hash` plugin appends `?v=<hash>` to asset URLs in the
  manifest for cache-busting.
- `vite-plugin-static-copy` copies `fonts/ images/ svg/ favicons/`.

Native CSS nesting and `color-mix()` are emitted as-is (the `.browserslistrc`
baseline targets browsers that support them). If you must support older
browsers, add a nesting-flattening PostCSS plugin — see extending-framework.

## Commands

| Command | What |
|---|---|
| `npm run dev` | Vite dev server |
| `npm run build` | production build → `wwwroot/assets/` |
| `npm run watch` | rebuild on change (dev mode) |
| `npm run clean` | remove `wwwroot/assets` |

## Related skills

- **building-pages** — authoring HTML pages: grid, classes, tokens, data-module.
- **extending-framework** — adding CSS/JS modules, tokens, breakpoints; build internals.
- **html-conventions / css-conventions / js-conventions** — the house code style
  (how we actually write blocks: one CSS file per block, many small JS modules,
  nesting not BEM, eager/lazy split, the module contract).

## Migration note (came from a Sass codebase)

This framework was migrated from a Sass 7-1 structure. Mapping:
`$variable` → `var(--token)`; `@include md-up` → `@media (--md-up)`;
`darken($c, 5%)` → `color-mix(in srgb, $c, black 5%)`; `rem(16px)` → `1rem`
(literal); SCSS nesting → native CSS nesting; `@import 'partial'` →
`@import 'partial.css'` (inlined by postcss-import).

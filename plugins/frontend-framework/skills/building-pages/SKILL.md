---
name: building-pages
description: How to author HTML pages with this vanilla-CSS + Vite framework — wiring the built CSS/JS, using the grid/container/column classes, the responsive visibility and utility classes, design tokens, and the data-module / data-module-lazy JS loader plus lazy stylesheets. Use when writing or editing page markup. Read framework-overview first for the layout and conventions.
version: 1.0.0
---

# Building pages

How to write an HTML page against the framework. Read framework-overview first.

## 1. Wire up the assets

Load the eager CSS in `<head>` and `app.js` (as a module) before `</body>`.
Queue any non-critical stylesheets in `window.lazyStyles` — `app.js` injects
them as `<link>`s after first paint. Paths are the build output under
`/assets/`; in production append the `?v=<hash>` from `assets/manifest.json`.

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="/assets/css/app.css">
    <script>
        // Stylesheets to load lazily (after first paint), injected by app.js.
        window.lazyStyles = ['/assets/css/app.lazy.css'];
    </script>
</head>
<body>
    <!-- page content -->
    <script type="module" src="/assets/js/app.js"></script>
</body>
</html>
```

If a single component has its own chunked stylesheet
(`/assets/css/modules/<name>.css`), add it to `window.lazyStyles` too.

## 2. Layout: container / row / column

Flexbox grid. Wrap content in `.container` (max-width, centred) or
`.container-fluid` (full width), put `.row`s inside, then column classes.

```html
<div class="container">
    <div class="row">
        <div class="col-full col-md-half col-lg-third">…</div>
        <div class="col-full col-md-half col-lg-twothird">…</div>
    </div>
</div>
```

- **Container:** `.container`, `.container.small`, `.container-fluid`,
  `.container-fluid.lg`.
- **Row modifiers:** `.row-between`, `.vertical-align-middle`, `.no-gutters`,
  `.thin-gutter`.
- **Column widths** (base = all widths): `col-fill` (flex:1), `col-eighth`,
  `col-fifth`, `col-quarter`, `col-third`, `col-twofifth`, `col-half`,
  `col-threefifth`, `col-twothird`, `col-threequarter`, `col-fourfifth`,
  `col-full`.
- **Responsive columns:** prefix with a breakpoint — `col-sm-*`, `col-md-*`,
  `col-lg-*`, `col-xl-*` (mobile-first; applies at that width and up). Order
  helpers: `col-md-order-first/-one/-two`, `col-lg-order-first`.

## 3. Responsive visibility

`hidden-xs`, `hidden-sm`, `hidden-sm-up`, `hidden-sm-down`, `hidden-md-up`,
`hidden-md-down`, `hidden-lg-up`, `hidden-lg-down`, `hidden-xl`.

## 4. Utility & base classes

- `visually-hidden` — hide visually, keep for screen readers.
- `fill-absolute` — absolutely fill the positioned parent.
- `hidden`, `block`, `flex`, `text-center`.
- `btn` + `btn-primary` / `btn-secondary` for buttons.
- `.embed` wrapping an `<iframe>`/`<video>` for a responsive 16:9 embed.

## 5. Design tokens

Use `var(--token)` for colours/fonts in any page-level `<style>` or component:
`--color-primary`, `--color-secondary`, `--color-text`, `--color-bg`,
`--color-border`, `--color-link`, `--color-white`, `--color-black`,
`--color-error`, `--font-family-base`, `--font-family-serif`,
`--font-weight-*`, `--grid-gutter-width`, `--container-max`. Full list in
`resources/assets/css/abstract/tokens.css`.

These ship as **neutral defaults** (system font stack, greyscale). Set real
brand fonts/colours per project by overriding the tokens — e.g. a project
`:root { --color-primary: #b3b372; --font-family-base: "Barlow", sans-serif; }`,
loaded after `app.css`. Don't hardcode brand hex/fonts into framework files.

## 6. JS modules (behaviour)

Attach behaviour declaratively — no manual wiring:

```html
<!-- init immediately on page load -->
<div data-module="example">…</div>

<!-- init lazily when scrolled into view (IntersectionObserver) -->
<div data-module-lazy="example">…</div>
```

`app.js` resolves the name to `/assets/js/modules/<name>.js`, dynamically
imports it (its own chunk), and calls the module's `init(element)`. The short
name works as long as the file is in `js/modules/`.

Other lazy attributes handled by `app.js`:
- `data-src` — set as `src` when in view (lazy images/iframes).
- `data-style` — apply inline `style` when in view.

After injecting markup dynamically (AJAX), dispatch
`window.dispatchEvent(new Event('lazyload:init'))` so the loader re-scans.

## Full page example

```html
<section class="container">
    <div class="row vertical-align-middle">
        <div class="col-full col-md-half">
            <h1>Welcome</h1>
            <p>Intro copy.</p>
            <a class="btn btn-primary" href="/contact">Get in touch</a>
        </div>
        <div class="col-full col-md-half" data-module-lazy="example">
            <div class="embed">
                <iframe data-src="https://player.vimeo.com/video/000" title="Video"></iframe>
            </div>
        </div>
    </div>
</section>
```

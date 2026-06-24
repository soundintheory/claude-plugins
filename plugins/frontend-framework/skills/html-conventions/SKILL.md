---
name: html-conventions
description: How we write HTML/markup in this framework — the house style. Self-contained blocks that link their own CSS, kebab-case nested class naming (NOT BEM), wiring behaviour with data-module / data-module-lazy, lazy assets (data-src, window.lazyStyles), the container/row/col grid, head/body asset wiring, and semantic/accessible markup habits. Use whenever authoring or reviewing page or block markup. Pairs with css-conventions and js-conventions.
version: 1.0.0
---

# HTML / markup conventions (house style)

How we write markup. Pairs with css-conventions and js-conventions. Examples
are framework-agnostic HTML; in a Razor/Piranha project the same patterns apply
inside a block display template (`.cshtml`).

## Blocks are self-contained

A "block" is one content component. Its markup, CSS file, and JS module share
the same kebab-case name (`centered-image-content`). A block:

1. links its own stylesheet(s),
2. has a single root element with the block's class,
3. wires any behaviour via a `data-module` attribute.

```html
<!-- the block links the CSS it needs -->
<link rel="stylesheet" href="/assets/css/modules/centered-image-content.css">

<div class="centered-image-content">
    <div class="container">
        <img class="img-container" src="…" alt="…" loading="lazy">
        <div class="content">
            <h3 class="title">Heading</h3>
            <p>Body copy.</p>
        </div>
    </div>
</div>
```

## Class naming — kebab-case, nested, NOT BEM

- Root class = block name in kebab-case: `.centered-image-content`,
  `.hero-slider`, `.booking-cta`.
- Child elements get **short semantic class names** (`.title`, `.content`,
  `.img-container`, `.inner`) — scoped by nesting in the CSS, not BEM. So it's
  `.hero-slider .title`, never `.hero-slider__title`.
- State/variant classes are plain modifiers toggled by JS or set server-side:
  `is-active`, `closed`, `fixed`, `left`.

See css-conventions for how these nest in the stylesheet.

## Wiring behaviour: data-module

Attach JS by adding `data-module="<module-name>"` to the block root. `app.js`
imports `/assets/js/modules/<module-name>.js` and calls its `init(element)` with
that element. No manual JS wiring per page.

```html
<!-- init on page load -->
<div class="glide" data-module="slider"> … </div>

<!-- init lazily when scrolled into view (IntersectionObserver) -->
<section class="feature-links" data-module-lazy="feature-links"> … </section>
```

Use `data-module` for above-the-fold / immediately-interactive blocks, and
`data-module-lazy` for anything below the fold. After injecting markup
dynamically (AJAX), dispatch `window.dispatchEvent(new Event('lazyload:init'))`
so the loader re-scans.

## Lazy assets

The loader (`app.js`) also handles deferred resources on any element matching
`[data-src],[data-style],[data-module-lazy]`:

- `data-src` — applied as `src` when the element scrolls into view (deferred
  images/iframes):
  ```html
  <iframe data-src="https://player.vimeo.com/video/000" title="Promo video"></iframe>
  ```
- `data-style` — inline `style` applied when in view (e.g. deferred background
  images).

### Lazy stylesheets

Two ways to load a block's CSS lazily:

1. **`window.lazyStyles` (this framework's built-in):** push the stylesheet URL
   into the array before `app.js` runs; it injects a `<link>` after first paint.
   ```html
   <script>
     window.lazyStyles = window.lazyStyles || [];
     window.lazyStyles.push('/assets/css/modules/hero-slider.lazy.css');
   </script>
   ```
2. **`<link … lazy>` / `<link … embedded>` attributes:** in projects with an
   asset-pipeline that understands them (e.g. the Razor/Piranha setup this
   framework came from), a block links eager + lazy CSS declaratively:
   ```html
   <link rel="stylesheet" href="~/assets/css/modules/hero-slider.css" embedded>
   <link rel="stylesheet" href="~/assets/css/modules/hero-slider.lazy.css" lazy>
   ```
   `embedded` = critical (inlined/eager), `lazy` = deferred. This is a
   server-side convention, not something `app.js` does — only use it where the
   pipeline supports it. Otherwise use `window.lazyStyles`.

See css-conventions for what belongs in the eager vs `.lazy.css` file.

## Layout: container / row / col

Wrap content in `.container` (or `.container-fluid`), rows in `.row`, columns
with responsive `col-*` classes. Mobile-first: stack on small, split on larger.

```html
<div class="container">
    <div class="row">
        <div class="col-full col-sm-half col-md-third">…</div>
        <div class="col-full col-sm-half col-md-third">…</div>
        <div class="col-full col-md-third">…</div>
    </div>
</div>
```

Full class list (containers, row modifiers, column widths, `hidden-*` helpers)
is in building-pages.

## Let copy wrap naturally — don't hardcode Figma line breaks

The line breaks you see in a Figma text layer are an artifact of that artboard's
width, **not** meaningful structure. Don't reproduce them by splitting a copy
block into one `<span>` (or `<br>`) per visual line. Write the text as one
continuous run and let it wrap responsively at each breakpoint.

```html
<!-- WRONG — forces three lines at every width -->
<h2 class="tagline-band__headline">
    <span>State of the art games</span>
    <span>Fabulous cocktails</span>
    <span>A grand night out</span>
</h2>

<!-- RIGHT — wraps naturally per breakpoint -->
<h2 class="tagline-band__headline">State of the art games. Fabulous cocktails. A grand night out.</h2>
```

In the CSS this also means **no `span { display: block }`** (or `<br>`) added
just to recreate the comp's wrapping. **Exception:** keep the spans when the
line structure or per-word styling carries real design meaning — e.g. a hero
wordmark where one word is a different colour. That's deliberate structure, not
incidental wrapping.

## Document wiring (layout/head/body)

In the master layout: eager CSS in `<head>`, `app.js` as a module before
`</body>`, lazy styles queued for `app.js` to inject.

```html
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- preconnect / preload critical fonts -->
    <link rel="preload" href="/assets/fonts/yourfont.woff2" as="font" type="font/woff2" crossorigin>

    <link rel="stylesheet" href="/assets/css/app.css">
    <script>window.lazyStyles = ['/assets/css/app.lazy.css'];</script>
</head>
<body>
    <a href="#content" class="visually-hidden">Skip to main content</a>
    <header class="site-header"> … </header>
    <main id="content"> … </main>
    <footer class="site-footer"> … </footer>

    <script type="module" src="/assets/js/app.js"></script>
</body>
```

(In Razor, `app.js` is loaded with `type="module" defer` and version-appended;
blocks contribute their own `<link>`s as shown above.)

## Semantic & accessible markup (habits)

These are expected, not optional:

- **Landmarks**: `<header>`, `<nav>`, `<main id="content">`, `<section>`,
  `<article>`, `<footer>`. One `<main>` per page.
- **Skip link** to `#content` as the first focusable element.
- **Headings** in order — one `<h1>` per page, don't skip levels for styling.
- **Images**: meaningful `alt`; `loading="lazy"` for below-the-fold; set
  `width`/`height` to reserve space; use `<picture>`/`srcset` for art direction.
- **Icon-only controls** get an `aria-label` (`<button aria-label="Open menu">`).
- **Modals**: `role="dialog"`, `aria-modal="true"`, `aria-hidden` toggled,
  labelled via `aria-labelledby`.
- **Tabs**: `role="tablist"` / `role="tab"` / `role="tabpanel"` wired together.

## Checklist for a new block's markup

- [ ] Single root element with the kebab-case block class.
- [ ] Copy wraps naturally — no per-line `<span>`/`<br>` reproducing Figma's
      line breaks (unless line structure carries real design meaning).
- [ ] Child elements use short semantic classes (scoped by nesting, not BEM).
- [ ] Behaviour wired via `data-module` / `data-module-lazy` (lazy below fold).
- [ ] Its CSS is linked (`window.lazyStyles` or `<link embedded/lazy>` per project).
- [ ] Deferred media uses `data-src` / `loading="lazy"`.
- [ ] Semantic landmarks, ordered headings, alt text, ARIA on icons/modals/tabs.

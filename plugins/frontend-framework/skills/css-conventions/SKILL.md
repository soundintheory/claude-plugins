---
name: css-conventions
description: How we write CSS in this framework — the house style. One CSS file per block/module, eager vs .lazy split, native nesting with &-modifiers (NOT BEM), mobile-first @media (--breakpoint) inline, design tokens over hardcoded values, small focused files. Use whenever authoring or reviewing CSS/styles for a block, component, or page. Pairs with html-conventions and js-conventions.
version: 1.0.0
---

# CSS conventions (house style)

How we write CSS. The framework mechanics (tokens, `@custom-media`, build) are
in framework-overview / extending-framework — this is about *style and habits*.

## The core rule: one CSS file per block

Every block/component gets its own stylesheet in
`resources/assets/css/modules/<block-name>.css`. Files are **kebab-case** and
named after the block (`booking-cta.css`, `hero-slider.css`,
`cottage-features.css`). Keep them **small and focused** — most are 40–150
lines; if one grows past ~200, that's a smell that it's really two blocks.

Each module builds to its own chunk (`/assets/css/modules/<name>.css`) so a
page only ships the CSS for blocks it actually uses.

## Anatomy of a module file

```css
/* 1. Always import the abstract layer first → tokens + breakpoints. */
@import '../abstract/all.css';

/* 2. One root class named after the block, everything nested under it. */
.booking-cta {
    display: flex;
    flex-direction: column;
    padding: 2rem 1.5rem;
    border: 3px solid var(--color-border);   /* token, not a hex */

    /* 3. Mobile-first: base styles first, then bump up at breakpoints inline. */
    @media (--md-up) {
        border-left: none;
    }

    /* 4. Element children nested (descendant or > direct child). */
    .title {
        font-size: 1.75rem;
        margin-bottom: 1rem;
    }

    /* 5. State / variant via &-modifier. */
    &.left {
        border-top: 3px solid var(--color-border);
    }

    /* 6. Pseudo-elements nested with &. */
    &::before {
        content: "";
        position: absolute;
        inset: 0;
    }
}
```

## Nesting & selectors — NOT BEM

We use **native CSS nesting**, not BEM class names. Don't write
`.block__element--modifier`. Instead nest:

- **Child elements** by their semantic class, nested under the block:
  `.card { .title { … } }`.
- **Direct children** with `>` when structure matters: `> .inner`, `> li`.
- **Modifiers / states** with `&`: `&.is-active`, `&.left`, `&:hover`,
  `&:first-of-type`.
- **Pseudo-elements** with `&`: `&::before`, `&::after`.

Keep nesting **3–4 levels deep at most**. Deeper than that means the markup is
too coupled — flatten it.

## Mobile-first responsive, inline

Write base (smallest) styles first, then layer breakpoints **inline inside the
selector** — never a separate media-query section at the bottom of the file.

```css
.nav-link {
    padding: 1rem;            /* mobile */
    font-size: 1.25rem;

    @media (--sm-up) { padding: 1.25rem 0.625rem; }
    @media (--md-up) { padding-inline: 1.25rem; }
    @media (--lg-up) { padding-inline: 2.5rem; font-size: 1.375rem; }
}
```

Breakpoint names: `--xs`, `--sm-up/-only/-down`, `--md-*`, `--lg-*`, `--xl-*`,
`--xxl-*`. These are `@custom-media` (see framework-overview) — they replace the
old Sass `@include md-up` mixins one-to-one.

## Eager vs lazy: split decorative styles into `.lazy.css`

A block can split its styles into two files:

- `block-name.css` — **eager / critical**: layout & structure that affects first
  paint. `display`, `flex`, `padding`, `margin`, `width`/`height`, `font-size`,
  `color`, and the responsive breakpoints for those.
- `block-name.lazy.css` — **non-critical / decorative**, loaded after first
  paint: `box-shadow`, decorative `background-image`, `::before`/`::after`
  image overlays, `opacity` effects, transitions/animations.

```css
/* cottage-features.lazy.css — decoration only, no abstract import needed
   unless it uses tokens/breakpoints. */
.cottage-features > li {
    &::after {
        content: "";
        position: absolute;
        inset: 0 0 0 auto;
        width: 2px;
        background: url("/assets/images/border.png");
    }
}
```

Rule of thumb: **if removing it wouldn't shift layout or cause a flash of
unstyled content, it belongs in `.lazy.css`.** See html-conventions for how a
block links its eager + lazy stylesheets.

## Use tokens, not hardcoded values

Reach for `var(--token)` instead of literal hex/fonts:
`var(--color-primary)`, `var(--color-text)`, `var(--color-border)`,
`var(--font-family-base)`, `var(--font-weight-bold)`, `var(--grid-gutter-width)`.
Framework tokens ship neutral; brand values are set per project by overriding
the token (never hardcode a brand hex into a framework/module file). Replace
Sass colour functions with `color-mix()` — e.g. a darker hover is
`color-mix(in srgb, var(--btn-bg), black 5%)`.

## Layer structure (where styles live)

- `abstract/` — tokens, breakpoints, utility classes. No block styles.
- `base/` — element defaults (reset, body, text, grid, links, buttons, forms,
  media). Site-wide, unclassed-ish element styling.
- `components/` — site-wide chrome shared across pages (header, footer, fonts).
  `required.css` aggregates the eager ones into `app.css`.
- `modules/` — one file per content block (the bulk of the work).

## Checklist for a new block stylesheet

- [ ] File is `modules/<kebab-name>.css`, named after the block.
- [ ] `@import '../abstract/all.css';` at the top.
- [ ] Single root class; children nested; modifiers via `&`.
- [ ] Mobile-first; breakpoints inline as `@media (--md-up)`.
- [ ] Tokens, not hardcoded colours/fonts.
- [ ] Decorative/below-the-fold styles moved to `<name>.lazy.css`.
- [ ] Nesting ≤ 3–4 levels; file stays focused (< ~200 lines).

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

## Width comes from `.container` — never a bespoke max-width wrapper

Content width is owned by the grid, not by ad-hoc CSS. Every section wraps its
content in **`.container`** (max-width, centred) or **`.container-fluid`**
(full width) — never a hand-rolled `.section__inner { max-width; margin: 0 auto }`.
That bypasses the grid system and drifts from the rest of the site.

- Layout inside the container uses `.row` + `.col-*`.
- **Narrower-than-container content** (a centred testimonial, intro, CTA): use a
  `.row` with a single `.col-*` of the right width and centre the row — don't
  hardcode a `max-width` on the block.
- **Full-bleed** sections (a hero, a banding with a background image) may be
  full width, but the *content* inside still sits in a `.container`.
- **Leave the container div alone — never style it.** Don't co-locate a styling
  class on `.container`/`.container-fluid`, and never put padding/margin/border/
  background on it. Need inner padding or other styling? Add a child div inside
  and style that, so it can't clash with the grid's container rules. Put
  `.container` on the highest/section-level element (directly inside `<section>`).

```html
<!-- standard section -->
<section class="my-block">
    <div class="container">
        <div class="row"> … cols … </div>
    </div>
</section>

<!-- narrow, centred content (no custom max-width) -->
<section class="testimonial">
    <div class="container">
        <div class="row row-center">
            <div class="col-full col-md-twothird"> … </div>
        </div>
    </div>
</section>
```

## Layout: grid system for columns, flex for the rest, absolute last

Reach for layout techniques in this order — it keeps pages consistent with the
rest of our codebase and avoids cross-browser surprises:

1. **The framework grid system** (`.container` / `.row` / `.col-*`) for
   **horizontal organisation** — content laid out in columns side-by-side across
   the page (two-up sections, card rows, sidebar + main, etc.). This is the
   default for that job; prefer it over CSS `display: grid`. The columns are
   flexbox under the hood and behave predictably everywhere.

   ```html
   <div class="container">
       <div class="row">
           <div class="col-full col-md-half">…</div>
           <div class="col-full col-md-half">…</div>
       </div>
   </div>
   ```

2. **Flexbox** for everything else — vertical stacking and the internal
   arrangement/alignment of a block or component, where `row`/`col` isn't the
   right tool. **Prefer `flex-direction: column`** and place items with flex
   alignment (`justify-content` / `align-items` / `gap`); we find this the most
   consistent across browsers.

3. **`position: absolute` last** — only for genuine overlays (a badge/pill
   sitting on an image, a decorative `::before`, etc.). Don't use absolute
   positioning to build layout that the grid or flex can express.

So: `row`/`col` for laying columns out horizontally, flex (column-first) for
stacking and component internals, absolute only for true overlays. Avoid
`display: grid` for layout these can handle — native CSS `grid` is fine for a
genuinely 2-D component (a true masonry/tile arrangement), but it's the
exception, not the reach-for default.

## Image cards / heroes: fixed-height box + image + overlay

Almost everything ships inside a CMS, where image dimensions and copy length
vary. So **never let an image dictate an element's height**, and never pin
content with absolute coordinates that assume a specific image. Build any
"text over an image" block (card, hero, banner, slide) like this:

1. **Container with an explicit height** (or `aspect-ratio`). CSS owns the
   height — not the asset.
2. **Image inside it**, filling the box as a background:
   `position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover`.
3. **A separate overlay div** layered above the image, covering the element
   (`position: absolute; inset: 0`), holding the content.
4. **Content inside the overlay flows normally** — lay it out with **flex**
   (`flex-direction: column` + `justify-content`/`align-items`/`gap`). Only the
   overlay itself is absolutely positioned; its children are not.

```css
.card {
    position: relative;
    height: 32rem;                 /* or aspect-ratio: 589 / 515; — CSS owns height */
    border-radius: 1.875rem;
    overflow: hidden;
}
.card__image {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;             /* image fills the box like a background */
}
.card__overlay {                   /* the ONLY absolutely-positioned layer */
    position: absolute;
    inset: 0;
    display: flex;                 /* content flows with flex, not absolute */
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    background: linear-gradient(180deg, transparent 40%, rgba(0,0,0,.85));
}
```

A genuine corner badge/pill is the one acceptable exception to absolute-position
a child. Everything else (titles, taglines, buttons) flows inside the overlay.

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

## Font sizes: exact Figma values, clamped mobile→desktop

Font sizes come straight from Figma and are clamped between the **mobile** and
**desktop** sizes — don't guess bounds or pin one fixed size that ignores the
responsive design:

```css
font-size: clamp(<mobile-figma-px>, <fluid-vw>, <desktop-figma-px>);
```

Lower bound = the exact mobile-frame size, upper bound = the exact desktop-frame
size, middle = a vw-based fluid term. Round to whole pixels. If Figma shows the
same size on both frames, a fixed value (no clamp) is fine — but only when it's
genuinely equal.

## Let copy wrap naturally — no `span { display: block }` for Figma line breaks

Don't add `span { display: block }` (or `<br>`) to force a copy block onto the
same lines it has in Figma — those breaks are an artifact of the artboard width,
not structure. Write the text as one run and let it wrap responsively. The
exception is when line structure or per-word styling is deliberate design (e.g.
a two-tone hero wordmark). See html-conventions for the markup side.

## No decimal pixels

Never use fractional pixel values — always round to the nearest whole pixel.
`4.8px` → `5px`, `47.746px` → `48px`. Figma specs are full of sub-pixel values;
round them when porting to CSS. Beware values that *compute* fractional —
`aspect-ratio: 589 / 515` yields a fractional height, so prefer a rounded
explicit `height` (or pick an aspect ratio that lands on whole pixels). Clean
`rem` values (`1rem`, `1.5rem`) are fine; don't invent odd `rem`s that resolve
to half-pixels just to match a Figma decimal. Sub-pixel sizing renders blurry
and inconsistently across browsers — whole pixels are crisp.

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
- [ ] Layout: `row`/`col` for horizontal columns, flex (column-first) for
      stacking/internals, absolute only for overlays — not `display: grid`.
- [ ] Image cards: CSS owns the height; image is `object-fit: cover`; content
      lives in a flex overlay, not absolutely-positioned children.
- [ ] Width comes from `.container`/`.container-fluid` (+ rows/cols), never a
      bespoke `max-width` wrapper; narrow content = centered single-col row.
- [ ] `.container` is clean & section-level — no styling class on it, no
      padding/bg/border; inner styling goes on a nested child div.
- [ ] Font sizes are exact Figma values, clamped `clamp(mobile, fluid, desktop)`
      — not guessed bounds or a single fixed size.
- [ ] No decimal pixels — whole-pixel values only (watch computed fractions).
- [ ] Tokens, not hardcoded colours/fonts.
- [ ] Decorative/below-the-fold styles moved to `<name>.lazy.css`.
- [ ] Nesting ≤ 3–4 levels; file stays focused (< ~200 lines).

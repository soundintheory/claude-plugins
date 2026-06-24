---
name: content-block
description: Every project needs one "content" block that styles raw WYSIWYG / rich-text output (h1–h4, p, ul/ol/li, a, img, blockquote, strong) directly, with no wrapper class — because a CMS editor can't add classes to what it outputs. Covers what to style, why it must target bare tags, the brand type scale, and common companion blocks (an "intro" lead with a rule). Use whenever a project needs editor-managed body copy, an About/article/terms page, or any rich-text region.
version: 1.0.0
---

# The content (rich-text / WYSIWYG) block

Every project gets **one reusable `content` block** whose job is to style the raw
HTML a CMS rich-text field (WYSIWYG editor) produces. This is the block an editor
fills with body copy on an About page, an article, a terms page, or any
free-form text region. Build it early — almost every site needs it.

## The core rule: style bare tags, NOT a wrapper class

A WYSIWYG editor outputs plain elements — `<h2>`, `<p>`, `<ul><li>`, `<a>`,
`<img>`, `<strong>`, `<blockquote>`. **It cannot add a class** to them. So the
content block must style the bare tags as descendants of the block root. Do
**not** require a `.content__body` (or any class) on the copy — the editor will
never produce it, so those styles won't apply to real content.

```css
/* RIGHT — styles whatever the editor outputs, directly. */
.content {
    h1, h2, h3, h4 { /* … */ }
    p  { /* … */ }
    ul, ol { /* … */ }
    li { /* … */ }
    a  { /* … */ }
    img { /* … */ }
}

/* WRONG — real editor output has no .content__body, so nothing applies. */
.content .content__body h2 { /* … */ }
```

This is the one place we deliberately style bare element selectors scoped under a
block, rather than semantic child classes — because the markup is editor-owned,
not author-owned. Everywhere else, keep using nested semantic classes (see
css-conventions).

## What to style (the full WYSIWYG surface)

Cover everything a typical editor can emit, so any content lands styled:

- **Headings** `h1`–`h4` (h5/h6 if the editor exposes them) — on the brand type
  scale, clamped mobile→desktop. Reset their margins and add top margin between
  blocks (`&:not(:first-child) { margin-top: … }`).
- **Paragraphs** `p` — body size/leading; zero the trailing margin on
  `&:last-child`.
- **Lists** `ul`/`ol`/`li` — restore `list-style` and `padding-left` (resets
  usually strip them); style `li::marker` with the brand colour.
- **Links** `a` — brand colour + underline.
- **Images** `img` — `width: 100%; height: auto`, rounded, block, with vertical
  margin (an editor drops images inline between paragraphs).
- **`strong`/`b`/`em`/`blockquote`** — weight/emphasis/quote styling.

Width comes from `.container` + a single readable column (e.g.
`col-md-twothird`), per use-container-for-width — never a bespoke max-width on
the copy. A `--center` modifier (centre the text; keep lists left-aligned via
`display:inline-block`) is a useful variant.

## Sizes: brand type scale, clamped

Headings use the project's real type scale from Figma, clamped between the mobile
and desktop sizes (see font-size-clamp-figma). One scale, defined once in this
block, reused for all editor content. Example shape:

```css
.content {
    h1 { font-size: clamp(<mobile>, <fluid>, <desktop>); }   /* e.g. 72px */
    h2 { font-size: clamp(<mobile>, <fluid>, <desktop>); }   /* e.g. 52px */
    h3 { font-size: clamp(<mobile>, <fluid>, <desktop>); }   /* e.g. 28px */
    h4 { font-size: clamp(<mobile>, <fluid>, <desktop>); }   /* e.g. 22px */
    p, ul, ol { font-size: 18px; line-height: 1.5; }
}
```

## Keep fixed-structure intros as their OWN block

A styled lead/standfirst paragraph (e.g. one with a coloured rule down its side)
is **not** rich-text — it's a fixed, single-purpose block. Give it its own
module (e.g. `intro`) rather than baking it into `content`. The content block is
only for arbitrary editor output; deliberate, designed text treatments are
separate blocks.

## Checklist for the content block

- [ ] One block, reused across the project for all editor-managed copy.
- [ ] Styles **bare tags** (`h1`–`h4`, `p`, `ul`/`ol`/`li`, `a`, `img`,
      `strong`, `blockquote`) — no required wrapper class.
- [ ] Heading sizes on the brand scale, `clamp(mobile, fluid, desktop)`.
- [ ] Lists get `list-style` + `padding-left` back; `li::marker` brand-coloured.
- [ ] `img` is `width:100%; height:auto`, rounded, with vertical margin.
- [ ] Width from `.container` + a readable column, not a bespoke max-width.
- [ ] Optional `--center` variant; lists stay left-aligned when centred.
- [ ] Designed leads (ruled intro, pull-quote) are separate blocks, not in here.

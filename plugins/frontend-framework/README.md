# Frontend Framework Plugin

Conventions and guidance for building HTML/CSS/JS with the Sound in Theory
vanilla-CSS + Vite frontend framework.

The framework itself (the buildable Vite/PostCSS + CSS/JS source you scaffold a
project from) lives in its own repository:

**https://github.com/soundintheory/frontend-framework**

This plugin contains the skills that document how to use and extend it.

## Install

```
/plugin install frontend-framework@soundintheory
```

## Skills

- **framework-overview** — architecture, folder layout, build pipeline, the
  design-token / `@custom-media` system, and the JS module loader. Read first.
- **building-pages** — authoring HTML pages: grid/container/column classes,
  tokens, `data-module` / lazy styles, a page template.
- **extending-framework** — adding CSS/JS modules, tokens, breakpoints, and the
  Vite / PostCSS build internals.
- **html-conventions** — house style for markup: self-contained blocks,
  kebab-case nested classes (not BEM), `data-module` wiring, accessibility.
- **css-conventions** — house style for CSS: one CSS file per block, eager vs
  `.lazy.css` split, native nesting, mobile-first `@media (--breakpoint)`.
- **js-conventions** — house style for JS: many small modules, the `init(el)`
  contract, class-based modules, wrapping third-party libraries.

---
name: js-conventions
description: How we write JavaScript in this framework — the house style. Many small single-purpose modules, the init(el) + default-class export contract, class-based modules with instance state / bound events / custom events, wrapping third-party libraries, helpers vs utils, config via constructor + static defaults, kebab-case files. Use whenever authoring or reviewing JS behaviour modules. Pairs with html-conventions and css-conventions.
version: 1.0.0
---

# JavaScript conventions (house style)

How we write JS. The loader mechanics live in framework-overview /
extending-framework — this is about *style and habits*.

## Philosophy: many small modules

**One module = one behaviour/component.** Prefer lots of small, composable,
single-purpose modules over a few big ones. Real modules range from ~10 lines
(a thin wrapper around a library) to ~130 lines (complex stateful UI); anything
much bigger is usually two behaviours that should be split. `app.js` orchestrates
them via HTML `data-module` attributes — see html-conventions.

Files live in `resources/assets/js/modules/` and are **kebab-case**:
`tabbed-navigation.js`, `site-header-affix.js`, `video-popup.js`.

## The module contract

Every module exports an `init(el)` function (the loader calls it with the
element that declared the module) and a default export (usually the class, for
direct import/testing):

```js
class GenericSlider {
    constructor(el, options = {}) {
        this.el = el;
        this.glide = new Glide(el, { ...this.defaults, ...options }).mount();
    }

    get defaults() {
        return { perView: 1, autoplay: 5000, rewind: true };
    }
}

export function init(el) {
    return new GenericSlider(el);
}

export default GenericSlider;
```

The loader does: `module.init(element)` if `init` exists, else `module.default`.
Keep `init(el)` as the single entry point — don't do work at import time unless
the module is intentionally a "run once on load" module (see below).

## Class-based, with instance state

Favour a class. Store state on the instance, query child elements in the
constructor, bind events there.

```js
class Expander {
    static defaults = { duration: 150, easing: 'linear' };

    constructor(el, config = {}) {
        if (!el) throw new Error('element null or undefined');

        this.config = Object.assign({}, Expander.defaults, config);
        this.el = el;
        this.expanded = !el.classList.contains('closed');
        this.btn = el.querySelector('[data-expander=true]');

        this.btn?.addEventListener('click', (e) => this.toggle(e));

        // Allow external control via custom events.
        this.el.addEventListener('expander:expand', this.expand.bind(this));
        this.el.addEventListener('expander:collapse', this.collapse.bind(this));
    }

    toggle(e) { /* … */ }
    expand(e) { /* … */ }
    collapse(e) { /* … */ }
}
```

Conventions seen throughout:
- **Bound handlers**: arrow callbacks or `.bind(this)` so `this` is the instance.
- **Custom events** for cross-component communication, namespaced with a colon:
  `expander:expanded`, `location-selected`. Dispatch with
  `this.el.dispatchEvent(new Event('expander:expanded', { bubbles: true }))`.
- **Guard clauses** for missing elements (`if (!el) throw …`, `this.btn?.`).

## Config patterns

- **Constructor params + static defaults**: `Object.assign({}, Class.defaults,
  config)` or `{ ...this.defaults, ...options }`. This is the primary pattern.
- **Data attributes** for per-element wiring:
  `el.querySelector(btn.dataset.target)`, `el.dataset.module`.
- **Class selectors** when a module initialises all matching elements itself.
- **No JSON config files / no globals for config.** Defaults are hardcoded in
  the class; overrides come through the constructor.

## Wrapping third-party libraries

Wrap a library inside the module — instantiate it in the constructor and store
the instance. Don't expose the raw library to the rest of the app.

```js
import MicroModal from 'micromodal';
import Glide, { Controls, Images } from '@glidejs/glide/dist/glide.modular.esm';

class ImageGallery {
    constructor(root) {
        this.root = root;
        this.slider = null;
        MicroModal.init({
            onShow: () => {
                this.slider = new Glide(this.root, { perView: 1, autoplay: 4000 })
                    .mount({ Controls, Images });
            }
        });
    }
}

export function init(el) { return new ImageGallery(el); }
export default ImageGallery;
```

`npm install` the lib and `import` it in the module — Vite bundles it into that
module's lazy chunk, so the cost is only paid on pages that use the block.

For heavy external SDKs (YouTube/Vimeo APIs, Google Maps), **lazy-load the
script** on demand rather than importing eagerly — use a shared
`utils/load-script.js`-style helper that injects a `<script>` and dedupes.

## "Run once on load" modules

Some tiny modules just act on all matching elements at import time instead of
per-element via `init`:

```js
import Player from '@vimeo/player';

document.querySelectorAll('.video-banner-iframe').forEach((iframe) => {
    const player = new Player(iframe);
    player.setVolume(0);
    player.setLoop(true);
    player.play();
});
```

Use this only for trivial, idempotent wrappers. Anything stateful should be a
class wired via `data-module`.

## Helpers vs utils vs components

Keep cross-module code small and focused, split by intent:

- `js/helpers/` — small task helpers (`debounce-function.js`, `recaptcha.js`).
- `js/utils/` — DOM / loading primitives (`get-ancestor.js`, `load-script.js`,
  `on-google-ready.js`). Example contract — a promise-returning, deduped script
  loader:

  ```js
  let loadedScripts = {};
  export default function (src) {
      return new Promise((resolve, reject) => {
          if (loadedScripts[src]) return resolve();
          const tag = document.createElement('script');
          tag.async = true;
          tag.src = src;
          tag.addEventListener('load', () => { loadedScripts[src] = true; resolve(); });
          tag.addEventListener('error', reject);
          document.getElementsByTagName('script')[0].parentNode.insertBefore(tag, null);
      });
  }
  ```

- `js/components/` — reusable UI behaviours imported by multiple modules.

Import helpers/utils into modules; don't duplicate the logic.

## Naming

- **Files**: kebab-case (`site-header-dropdown.js`).
- **Classes**: PascalCase (`SiteHeaderDropdown`).
- **Functions/methods**: camelCase (`init`, `toggle`, `loadApi`).
- **Custom events**: kebab/namespaced (`expander:expand`, `location-selected`).

## Checklist for a new JS module

- [ ] File is `modules/<kebab-name>.js`, one behaviour only, small.
- [ ] Exports `init(el)` + a default class.
- [ ] State on the instance; events bound in the constructor; guard clauses.
- [ ] Config via constructor params + static defaults.
- [ ] Any third-party lib is wrapped (and heavy SDKs lazy-loaded).
- [ ] Shared logic pulled from helpers/utils, not duplicated.
- [ ] Wired in markup via `data-module` / `data-module-lazy` (html-conventions).

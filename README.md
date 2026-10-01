# Manelis Studio

Manelis Studio is a jewelry atelier going online — necklaces, charms, chains, and
bundles designed to become heirlooms. This repo will grow into the storefront;
right now it's very early.

## Current state

- **`index.html`** — a static, framework-free "coming soon" landing page. No
  build step, just HTML with inline CSS, Google Fonts, and a small vanilla-JS
  localization layer (see below). Open it directly in a browser — including
  as a local `file://` path — to view it in any of the 8 languages.
- **`locales/data.js`** — the localization manifest and every language's
  translated strings, as plain JS data (not JSON — see below for why).
- **`assets/i18n.js`** — the runtime engine that resolves the visitor's
  language and renders the translated text.
- **`assets/flags.js`** — hand-drawn SVG flag icons shown in the language
  switcher, keyed by locale code.
- **`assets/icons/`, `favicon.ico`, `site.webmanifest`** — the browser-tab /
  home-screen icon — an exact copy of the page's logo (SVG favicon, ICO
  fallback, Apple touch icon, 192/512 and maskable PNGs). `assets/icons/icon-source.svg` is the
  source; the PNGs and ICO are exports, re-rendered from it with any SVG
  rasteriser (the root `favicon.ico` is a copy of the one in `assets/icons/`).
  The icon is static and not localized.
- **`scrapped/charmlanecollection.md`** — a scraped copy of
  charmlanecollection.com, kept only as design/content reference for the real
  storefront. Not code, and not meant to be copied verbatim.

No application framework, package manager, or build tooling has been chosen
yet — the real catalog, cart, and checkout experience haven't been built.

## Localization

The placeholder page is available in 8 languages: English (default), Spanish,
French, Italian, Portuguese, Russian, Ukrainian, and Hebrew (right-to-left).

Language is chosen, in order: a `?lang=xx` URL parameter, a previously saved
choice, the browser's language, then English. Visitors can also pick a
language from the switcher at the bottom of the card; the choice is saved and
reflected in the URL.

Locale data lives in `locales/data.js` as plain JavaScript (`window.MANELIS_I18N = {...}`), loaded
with a normal `<script src>` tag rather than fetched as JSON. Browsers block `fetch()` on `file://`
pages for security, but a `<script src="...">` tag loads a local file the same way regardless of
whether the page is opened directly or served over HTTP — that's what lets `index.html` localize
correctly straight from disk, with no local server needed.

**Adding a language** requires no changes to `index.html` or `assets/i18n.js` — edit only
`locales/data.js`:

1. Add one property to `BUNDLES` with the same keys as `BUNDLES.en`, translated.
2. Add one entry to `MANIFEST.locales` — `code`, the language's native `name`, and `dir`
   (`"ltr"` or `"rtl"`).
3. If the language needs a script the existing Google Fonts (Cormorant Garamond, Jost) don't
   cover, also set `fontsHref`, `fontSerif`, and `fontSans` on that manifest entry (see the `he`
   entry for an example) — they're loaded only while that language is active.
4. Optional: add a matching SVG flag to `assets/flags.js`, keyed by the same `code`, to show an
   icon in the switcher. A locale with no entry there just renders without one — this step doesn't
   block the language from working.

A missing translation key falls back to English for that key only; a missing bundle falls back to
English entirely — the page never breaks or goes blank.

Flags are drawn as small inline SVGs rather than emoji (Windows renders flag emoji as plain
two-letter codes, not pictures) or an external icon library (which would need a network request
and break offline/`file://` use). They're hardcoded in `assets/flags.js`, not in the translation
data in `locales/data.js` — that keeps the rule that nothing rendered from translator-editable
content ever uses `innerHTML`.

**Testing locally:** just open `index.html` in a browser, or visit
`index.html?lang=he` (or any other locale code) directly from disk. A local HTTP server (e.g.
`python -m http.server 8000`) works identically if you prefer one.

## Contributing

See [CLAUDE.md](CLAUDE.md) for guidance on working in this repo, including
notes for AI coding agents.

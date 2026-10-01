# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

See [README.md](README.md) for a repo overview. Manelis Studio is intended to become an
e-commerce jewelry site, but that build hasn't started:

- [index.html](index.html) — a static, framework-free "coming soon" placeholder page (inline
  CSS, Google Fonts, no build step). It now carries a small vanilla-JS localization layer — see
  "Localization" below. This is what's currently live at the domain root, not the real storefront.
- [scrapped/charmlanecollection.md](scrapped/charmlanecollection.md) — a scraped copy of
  charmlanecollection.com (a jewelry e-commerce storefront: necklaces, charms, chains, bundles),
  kept as design/content reference only — not code, and not to be copied verbatim.

No framework, package manager, or build tooling has been chosen yet for the real storefront
(catalog, cart, checkout), so there are still no build/lint/test commands and no architecture
to document.

**When scaffolding begins:** update this file with the actual stack, the real commands (build,
dev server, lint, test — including how to run a single test), and the high-level architecture
(routing, data/catalog model, cart/checkout flow, styling approach) once those exist in the repo.
Do not carry over assumptions from the reference scrape — verify against the code that's actually
committed. If the storefront gets server rendering or a build step, revisit the localization
approach below — a build-time approach can emit static per-locale HTML and true server-rendered
`hreflang`/`<title>`, which this runtime approach cannot.

## Conventions

- **Mobile accessibility:** every edit (markup, CSS, JS, assets) must be made with mobile
  accessibility in mind — check phone-width layout, touch targets (≥44px), zoom/reflow, contrast,
  RTL, and avoid hover-only interactions.

## Localization

Runtime, manifest-driven i18n — chosen because GitHub Pages has no server-side templating and the
issue required that adding a language never touch existing pages. Locale codes, metadata (name,
text direction, optional per-locale fonts), and all translated strings live only in
[locales/data.js](locales/data.js); neither `index.html` nor `assets/i18n.js` ever hardcodes a
locale.

Locale data is loaded via a plain `<script src="locales/data.js">` tag, **not** `fetch()`/JSON.
Browsers block `fetch()` on `file://` for security, but a normal `<script src>` loads a local file
fine either way — this is what makes the page localize correctly whether it's opened directly as
`file:///.../index.html` or served over http(s). Do not reintroduce a `fetch()`-based loader for
locale data without re-verifying `file://` still works.

- **How it works:** `index.html` keeps the English copy inline (the no-JS / no-data fallback) and
  tags translatable elements with `data-i18n` / `data-i18n-attr` / `data-i18n-slot`.
  `locales/data.js` sets `window.MANELIS_I18N = { manifest, bundles }` synchronously.
  [assets/i18n.js](assets/i18n.js) resolves the visitor's locale (`?lang=` → `localStorage` →
  `navigator.languages` → the manifest's `default`) and renders text via `textContent`/DOM nodes
  only — never `innerHTML` — so a bundle can never inject markup. A missing key falls back to the
  default-locale bundle per-key; a missing bundle falls back to the default locale entirely, with
  a `console.warn`, never a blank page. A synchronous boot `<script>` in `<head>` (after
  `locales/data.js`) resolves the same way to set `dir`/`lang` before first paint, avoiding a flash
  of the wrong direction or a stale language.
- **Adding a language:** in `locales/data.js`, add one entry to `MANIFEST.locales` (`code`, native
  `name`, `dir`) and one property to `BUNDLES` with the same keys as `BUNDLES.en`. For a
  non-Latin/non-Cyrillic script, also set `fontsHref` (a Google Fonts CSS2 URL), `fontSerif`, and
  `fontSans` on that manifest entry — they're injected only while that locale is active. No changes
  to `index.html` or `assets/i18n.js` needed. Optionally add a matching flag to
  [assets/flags.js](assets/flags.js) keyed by the same `code` (see "Flag icons" below) — a locale
  without one just renders without an icon.
- **RTL:** `he` (Hebrew) is the reference RTL locale — `dir: "rtl"` in the manifest flips
  `<html dir>` and the page's centered layout handles it without extra CSS. The switcher's flag +
  label layout uses `display: inline-flex` with no manual RTL override, so it mirrors correctly for
  free under `dir="rtl"`.
- **Flag icons:** [assets/flags.js](assets/flags.js) sets `window.MANELIS_FLAGS`, a locale-code →
  SVG-string map, keyed to match `MANIFEST.locales[].code`. These are hand-authored, hardcoded SVGs
  — deliberately kept out of `locales/data.js` and rendered via a scoped `innerHTML` in
  `buildSwitcher()` (commented at the call site) — because they are trusted application code, never
  translator-editable content, unlike everything rendered through `data-i18n*`, which must keep
  using `textContent`/DOM nodes only. Do not move flag markup into `locales/data.js` or render
  bundle values via `innerHTML` — that would reopen the injection risk this split exists to avoid.
  Emoji flags were rejected: Windows renders them as plain two-letter codes, not pictures.
- **Local testing:** works by opening `index.html` directly (`file://`) — no server required. A
  local HTTP server (e.g. `python -m http.server 8000`) works identically and is closer to
  production, but is optional.

## Favicon / app icons

Static and not localized — nothing in `locales/data.js` touches it. The icon is an **exact copy of
the page's `.ornament` logo** (same paths, tendrils, 1.4px stroke and 0.18 fill-opacity), with
`currentColor` replaced by `--gold-soft` `#cda863` and the viewBox cropped to the mark. Do not
simplify or restyle it. [assets/icons/](assets/icons/) holds `icon-source.svg` (the source) and an
identical `favicon.svg`, plus exports: `favicon.ico` (16 + 32), `icon-192.png`, `icon-512.png`
(transparent), and `apple-touch-icon.png` (180) and `icon-maskable-512.png`, both the logo centred
on an opaque cream `#fbf6ef` square — iOS fills transparency with black, and the maskable one also
needs a full-bleed background with the mark inside the ~80% safe zone. A byte-identical `favicon.ico` also
sits at the repo root so the default `/favicon.ico` request never 404s, and
[site.webmanifest](site.webmanifest) lists the PNGs. They're wired by plain `<link>` /
`<meta name="theme-color">` tags in `index.html`'s `<head>` — no JS, so it works from `file://`. The
PNGs/ICO are exports: if the `.ornament` in `index.html` changes, update `icon-source.svg` and
`favicon.svg` to match, then re-render with any SVG rasteriser; no build tooling is committed.

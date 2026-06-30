# Handoff — González Fischer website

## Overview
A single-page marketing/brand site for **González Fischer**, a small family winery in Sacramenia (Segovia, Spain). It tells the estate's story (El proyecto / El terruño / El oficio), presents three wines (Gatico, Lechuzo, Paramera) with per-wine detail views, and links out to an external shop for purchase. Bilingual (Spanish default / English), with an 18+ age gate on first visit.

## About these files — read first
Unlike a typical design handoff, **these files are not throwaway mockups to rebuild in a framework — they ARE the production code.** The site is a hand-authored, framework-free static site: one HTML file, one shared CSS token file, vanilla JS, and an `assets/` folder. It is already deployable as-is.

The recommended path (agreed with the client) is to **keep it framework-free and static** — no build step, no JSON/Markdown extraction, no SSG. Edit the HTML/JS directly. Reasons: the content is small (one page, three wines), a static site is the most secure and lowest-maintenance option, and GitHub Pages serves it with zero configuration.

## Fidelity
**High-fidelity / final.** Colors, typography, spacing, copy (ES + EN), interactions and responsive behavior are all final and production-ready. Don't re-interpret — preserve.

## Run & deploy
- **Local:** open `docs/index.html` in a browser (it works from the filesystem; all paths are relative).
- **GitHub Pages:** the site lives in `docs/`; Pages is configured to serve from the **main** branch, **/docs** folder. No build step. Custom domain via a `CNAME` file in `docs/` when ready.

## File structure
```
docs/
  index.html              ← the entire site (markup + inline CSS + JS)
  colors_and_type.css     ← shared design tokens + base type (the "design system" layer)
  assets/
    logo.png              ← wordmark (used in header, footer, age gate)
    photo-*.jpg/.jpeg     ← editorial photography (hero + full-bleed bands)
    bottle-*.webp         ← the three bottle shots
    seal-*.svg            ← certification marks (white, for the dark footer)
```

## Architecture
- **One HTML file.** Page-specific CSS lives in a `<style>` block in `<head>`; shared tokens/type live in `colors_and_type.css` (linked first, so the page can override). Vanilla JS at the end of `<body>` — no libraries, no framework.
- **Two "views"** (`[data-view="home"]` and `[data-view="detail"]`) toggled by a tiny client-side router (`route()`); the wine **detail** view is rendered from data by `renderDetail()`. Single-page interaction; current route is remembered in `localStorage` (`gf.route`).
- **Shareable detail links:** opening a wine sets `?wine=<id>` on the URL via `history.pushState` (alongside any `?lang=`), so detail pages can be shared/bookmarked; loading such a URL opens that wine (`openDetail(id, {push:false})` in init — read-only on load, per the `?lang` caveat). Browser back/forward is wired through a `popstate` handler, and returning home (`dropWineParam()`) strips the param. Works on GitHub Pages with no server (query string, not a real path).
- **Content as data:** the three wines live in the `WINES` array in the `<script>` (id, name, price, image, plus `idea` / `making` / `tasting` / `pairing` / `tech`, and an `en:{…}` block mirroring all text). Narrative prose (the chapters) lives directly in the HTML.

## Internationalisation (ES / EN)
No JSON — translations live **inline**, three mechanisms:
1. **Static text:** any element with a `data-en="…"` attribute. Its Spanish is the element's normal content; the English is the attribute. On load the script snapshots the Spanish into `data-es` and swaps `innerHTML` between the two. To translate a new element, add a `data-en`.
2. **`alt` attributes (editorial photos):** an element with `data-en-alt="…"` swaps its **`alt`** attribute (not innerHTML) — the Spanish lives in `alt="…"`, the English in `data-en-alt="…"`. The script snapshots the Spanish into `data-es-alt`. Use this for images; a plain `data-en` on an `<img>` would not work (the swap targets innerHTML). The bottle photos are JS-rendered, so they localise via mechanism 3 (`tr('bottleOf')`) instead. Logos and certification seals keep one proper-noun `alt` in both languages.
3. **JS-rendered text (wines + UI labels):** the `UI` dictionary (`{es, en}` per key) and each wine's `en:{…}` block. `wineView(w)` returns the language-correct merge.
- Language is chosen by `?lang=` → `localStorage('gf.lang')` → default `es`. The header **ES / EN** toggle calls `applyLang(l, true)`; it writes `?lang=` to the URL **only on click** (writing it on load caused a preview reload loop — do not change this).
- Price/number formatting also localises (e.g. `€38,00` ES vs `€38.00` EN) — currently prices are hidden, but the logic remains.

## Design tokens (in `colors_and_type.css`)
**Colour** (foreground on white / on dark):
- `--fg-1 #16181b` (17.8:1) — primary text & near-black
- `--fg-2 #2a2d31` (13.8:1) — subheads/leads
- `--fg-3 #4a4e54` (8.4:1) — body
- `--fg-4 #656a70` (5.5:1 white / 4.9:1 paper) — small mono labels. **This value was darkened from #7c8087 to meet WCAG AA — keep it ≥4.5:1.**
- `--fg-5 #b2b5ba` — only on the dark footer (8.6:1 there)
- `--bg-white #fff`, `--bg-1 #f2f3f2` (paper), `--bg-dark #16181b`
- `--accent #16181b` (near-black) — buy buttons & focus rings. (An ink-blue `#34465d` is the alternate; see Tweaks lock.)
**Type:** one sans family for everything, driven by `--font-sans`. Locked to **Instrument Sans**. Fluid type scale (`--step-*`, all `clamp()`), so it adapts to any family with no layout changes.
**Spacing:** `--space-1…10` (rem, 4px-based). **Gutter:** `--gutter` is fluid `clamp(1.25rem,3vw,2.5rem)`. **Container:** `--container 1240px`.

## Sections (home, top → bottom)
Hero (full-bleed parallax photo + motto) → **I El proyecto** (founders' story + pull-quote) → **II El terruño** (place; 4-cell data band: Altitud/Suelo/Clima/Amplitud térmica) → **III El oficio** (craft; two data bands: vineyard + production) → **IV Los vinos** (three wine cards + "Una bodega familiar" invitation block) → footer. Full-bleed photo bands separate the chapters. Wine **detail** view: sticky bottle photo + name, hook line, Elaboración / En boca / A la mesa, a spec table, buy + "otros vinos" cross-links.

## Interactions
- **Router / views:** `route('home'|'detail')`; brand logo → home; in-page anchors via `goToAnchor()` with scroll-spy highlighting the active nav item.
- **Wine detail:** click any wine (cards or footer) → `openDetail(id)`; "Otros vinos" switches wine in place.
- **Age gate (18+):** opaque overlay on first visit. An inline `<head>` script adds `age-gated`/`age-ok` to `<html>` before paint (no flash). **Yes** → store `gf.age.ok`, reveal site, move focus to the visually-hidden page `<h1>` (a `tabindex="-1"` script-managed landing target — screen readers announce the title, and `.vh` + `outline:none` means no visible focus ring; previously focused the logo button, which showed a jarring `:focus-visible` ring on programmatic focus). **No** → show denial message, stay blocked. Focus is trapped in the dialog while open. *(Self-declaration is the correct pattern for a brand site; the legally-robust age verification belongs at the shop's checkout, not here.)*
- **Mobile nav (≤910px):** hamburger → full-screen overlay menu (links + shop + ES/EN); closes on navigation; locks body scroll. (The JS resize handler that auto-closes the menu uses the same 910px threshold — keep them in sync.)
- **Parallax** on full-bleed photos; **disabled** under `prefers-reduced-motion` (handled in JS and CSS).

## Responsive
Breakpoints: **910px** (nav → hamburger), **820px** (footer → 2-col), **800/880px** (existing grid collapses), **760px** (data bands → 2-up; values may wrap), **600px** (tighter section padding), **520px** (footer → 1-col; hero text wraps). Data bands on desktop size each column to its content (`max-content` + `space-between`) so values never wrap.

## Accessibility (already done)
Single landmarks; `lang` set & updates; all images have `alt`; icon links have `aria-label`; visually-hidden `H1` on home; heading levels ordered; real anchor hrefs; `:focus-visible` outline; reduced-motion support; AA contrast (see `--fg-4` note). Keep these when editing.

## The Tweaks panel & the lock
During design there was an in-page font/accent picker ("Tweaks"). It is **locked off**: `const TWEAKS_ENABLED = false;` in the script. The chosen values are baked into `TW_DEFAULTS` (`face: "Instrument Sans"`, `accent: "#16181b"`). To bring the live picker back, flip the flag to `true`. To change the locked font/accent without the picker, edit `TW_DEFAULTS`. (You may also delete the whole Tweaks block in production — values are applied from `TW_DEFAULTS` regardless.)

## Deploy checklist (the few open items)
1. ~~**Self-host the font (recommended for EU/GDPR + performance).**~~ **Done.** Instrument Sans (OFL) is now self-hosted: the variable upright (weights 400–700, latin + latin-ext subsets) lives in `assets/fonts/instrument-sans-*.woff2`, declared via `@font-face` at the top of the `<style>` block. The runtime Google-font loader (`loadGoogleFont`) and the two `<link rel="preconnect">` font lines have been removed — no visitor IPs go to Google. (Italics are not used anywhere, so only the upright file is shipped; if you ever add `font-style: italic`, fetch the italic subset too.)
2. ~~**Absolute `og:image`.**~~ **Done.** `og:url` and `og:image` are set to absolute `https://gonzalezfischer.com/…` URLs (the production domain). They only resolve in link previews once DNS points the domain at GitHub Pages — see "Custom domain cutover" below.
3. ~~**Favicon.**~~ **Done.** Replaced the wordmark `logo.png` favicon with the brand heart (`#16181b`) centered on a solid paper tile (`#f2f3f2`): `docs/favicon.png` and `docs/apple-touch-icon.png` (both 512×512). Linked via `<link rel="icon">` and `<link rel="apple-touch-icon">`. Solid tile means it stays legible on dark-mode tab strips and on iOS home screens (which composite transparency onto black).
4. ~~**Optimise photos.**~~ **Done.** The six editorial photos (`photo-*.jpg`) were converted to grayscale colorspace, the 30 MP `photo-vacio-aire.jpg` was downscaled to 2560 px on the long edge, and all were recompressed at q80 (`cjpeg -optimize`, EXIF stripped). Total photo payload dropped ~10.9 MB → ~3.7 MB. **Note:** the grayscale look on the hero and full-bleed bands is now baked into the source files — the `filter: grayscale(...)` CSS was removed from `.hero img` / `.full-bleed img`. So any **replacement** photo for those slots must be pre-converted to B&W (e.g. `jpegtran -grayscale in.jpg > out.jpg`); a color JPEG would now render in color. (The unused design-system selectors `.feature`/`.split`/`.cart-item` etc. still carry `filter: grayscale(1)` for any future color-sourced component.)

## Custom domain cutover (gonzalezfischer.com)
The production domain is **gonzalezfischer.com**, currently still served by a WordPress site. Do these steps **only when ready to switch**, because configuring the custom domain in GitHub Pages makes the `…github.io/gf_site/` URL redirect to the custom domain — so until DNS points here, the preview URL would redirect to whatever the domain still serves (WordPress).

Preview safely in the meantime at **https://d4weed.github.io/gf_site/** (all asset paths are relative, so the subpath works; only the absolute OG meta URLs differ, and those only matter to social crawlers).

Cutover steps, in order:
1. **DNS** at the registrar — point the apex `gonzalezfischer.com` at GitHub Pages' four A records (`185.199.108.153`, `.109.153`, `.110.153`, `.111.153`) and AAAA records, and add a `www` CNAME → `d4weed.github.io`. (Removes/replaces the WordPress records — this is the irreversible-feeling step; lower the TTL a day before to speed rollback.)
2. **CNAME file** — add a `docs/CNAME` containing `gonzalezfischer.com` (one line), commit & push.
3. **GitHub Pages settings** → set **Custom domain** to `gonzalezfischer.com`, wait for the DNS check, then enable **Enforce HTTPS** once the cert is issued.

## How to evolve it
- **Edit copy:** find the text in `index.html`. For bilingual strings, update **both** the visible Spanish **and** the `data-en` attribute next to it (the one easy thing to forget).
- **Add / change a wine:** edit the `WINES` array (add `id`, `name`, `img`, `kind`, `idea`, `making`, `tasting`, `pairing`, `tech`, and the parallel `en:{…}`). Add its bottle image to `assets/` and a footer link.
- **Add a language:** generalise the two-language swap (today it's ES base + `data-en`); for a third language you'd move to a small key→{lang} map. Ask before doing this — it's the one change that benefits from restructuring.
- **Add a page:** the only real maintainability cost of staying single-file is that a second page duplicates the header/footer/age-gate. If you add several pages, that's the moment to introduce a 20-line include/build step (or a light SSG) — not a framework.

## Design round-trip (Claude Code ⇄ Claude design tool)
- **Content / copy / translations / a new wine** → do directly here in Claude Code (edit HTML/JS).
- **Visual changes (layout, type, colour, a new treatment) or a brand-new page** → take `index.html` into the Claude **design** environment, make the change visually, bring the updated HTML/CSS back. Keep the markup un-abstracted so the same file opens cleanly in both places — treat `index.html` as the design-of-record.
- Tokens live in `colors_and_type.css`; prefer changing a token over hard-coding a value, so both tools and any future component stay consistent.

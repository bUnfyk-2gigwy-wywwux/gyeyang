# CLAUDE.md

Guidance for AI assistants (Claude Code and others) working in this repository.

## What this is

`gyeyang` is the website and supporting tools for **계양부동산** (Gyeyang Real
Estate), a licensed real-estate brokerage (공인중개사사무소) in Gyeyang-gu,
Incheon, South Korea. It is a **static, hand-authored site with no build
step** — just HTML files committed directly to the repo and served as-is
(e.g. GitHub Pages; the production domain is `계양부동산.com`).

There is **no package.json, no bundler, no framework, no CI, and no test
suite**. Everything (HTML, CSS, JS, and even images) lives inline in a handful
of `.html` files. Treat each file as a self-contained deliverable.

The project, its content, comments, and commit messages are primarily in
**Korean**. Keep user-facing copy in Korean and match the existing tone
(polite, professional 존댓말). Code identifiers are a mix of English and Korean.

## Files

| File | Purpose | Self-contained? | Persistence | Notes |
|------|---------|-----------------|-------------|-------|
| `index.html` | Main marketing landing page | Full HTML document (`<!DOCTYPE html>` … `</html>`) | — | The primary site. ~1,260 lines. |
| `sangga.html` | "가로변 상가 현황" — commercial-storefront (상가) street-map tool | **Fragment** (no `<html>/<head>/<body>`) | `window.storage` | Built for the Claude Artifacts runtime; imports Excel via SheetJS. Generic (road name blank by default). |
| `sangga_계산로.html` | Standalone storefront map specialized for **계산로 (Gyesan-ro)** street | Full HTML document | `localStorage` | Deployable version with 계산로 data baked in. No Excel import. |

> The filename `sangga_계산로.html` contains Korean characters. In the shell,
> quote it or use a glob (`sangga_*.html`); bare `cd`/`cat` with the literal
> name often fails on byte expansion.

### `index.html` — the landing page

- Single-page layout. Sections (each an `id` used by the sticky nav):
  `#hero`, `#profile`, `#about`, `#services`, `#area`, `#location`, `#blog`,
  `#contact`.
- **Styling:** all CSS is in one `<style>` block in `<head>`. Theme is driven
  by CSS custom properties under `:root` — a navy + gold palette
  (`--navy`, `--gold`, `--cream`, etc.). Fonts are Google Fonts
  *Noto Serif KR* (headings) and *Noto Sans KR* (body). Reuse these variables
  rather than hard-coding colors.
- **Images** are embedded as base64 `data:` URIs (this is why the file is
  ~140 KB). This keeps the site dependency-free but makes the file large and
  noisy in diffs — avoid reformatting lines containing base64 blobs.
- **JavaScript** lives in one `<script>` at the bottom of `<body>`:
  - `formatPhone()` — auto-hyphenates the phone input.
  - `submitForm()` — validates the contact form (name, phone, privacy
    consent required) and `POST`s JSON to **Formspree**
    (`https://formspree.io/f/mgorgndg`). On success it swaps the form for a
    success message; on failure it tells the user to use the KakaoTalk channel.
  - Two `IntersectionObserver`s: one highlights the active nav link on scroll,
    one adds a `.visible` class to `.fade-up` elements for entrance animations.
- **External contact integrations** (keep these in sync if they change):
  - Formspree form endpoint: `https://formspree.io/f/mgorgndg`
  - KakaoTalk channel: `http://pf.kakao.com/_NmKTX`
  - Phone: `tel:010-5274-2399`
- SEO matters here: there are deliberate `<meta>` description/keywords,
  Open Graph, and Twitter Card tags targeting Gyeyang-gu real-estate search
  terms. Preserve them when editing the `<head>`.

> Note on form history: earlier commits integrated EmailJS, then Formspree.
> **Formspree is the current implementation** — do not reintroduce EmailJS.

### `sangga.html` / `sangga_계산로.html` — storefront map tools

These render a street with shops as tiles on the left/right side of a road,
showing area in both ㎡ and 평 (pyeong), and let the user add/edit shops and
mark them open (`open`) or vacant (`vacant`).

**Shop data model** (array of objects, JSON-serialized into storage):

```
{ id, name, addr, bld, area, status, side, order, prev }
```
- `status`: `'open'` | `'vacant'` (legacy `'preparing'` is migrated to `'open'`).
- `side`: `'left'` | `'right'`; `order` controls position along the road.
- `area` is ㎡; `pyung()` converts via `sqm / 3.3058`.
- `prev` (previous tenant) is shown only for vacant units.

**The two variants differ mainly in persistence**, and you must respect this:
- `sangga.html` uses the Artifacts **`window.storage`** async API
  (`window.storage.get/set`). It reads from a versioned key list
  (`gyeyang-shops-v6` … `v2`) and migrates older data forward into the current
  key (`gyeyang-shops-v6`); road name is `gyeyang-road-v1`. It also loads
  **SheetJS** from `cdnjs` to import shops from an Excel file. **When bumping
  the schema, add a new `gyeyang-shops-vN` key to the front of the `KEYS`
  list and add migration logic in `load()` — do not silently change an
  existing version's shape.**
- `sangga_계산로.html` uses plain **`localStorage`** (`LS_SHOPS`, `LS_ROAD`),
  has no Excel import, and ships with the 계산로 dataset inline. It is the
  version meant to run as an ordinary web page.

If you change shared logic, decide consciously whether the change applies to
one variant or both — they are near-duplicates that have intentionally diverged
(runtime, storage, dataset).

## Conventions

- **No build / no tooling.** Edit the `.html` files directly. There is nothing
  to install, compile, lint, or transpile. Don't introduce a build system,
  package manager, or framework unless explicitly asked.
- **Keep files self-contained.** Inline CSS/JS/images is the established
  pattern. Don't split into separate `.css`/`.js` assets without being asked.
- **Korean-first content.** Write copy, labels, and alerts in Korean. Commit
  messages in this repo are written in both Korean and English — either is
  fine; be descriptive.
- **Preserve SEO/meta and the navy-gold theme** in `index.html`.
- **Don't churn base64 blobs.** When editing `index.html`, target the specific
  markup/logic you're changing and leave long `data:image/...` lines untouched.
- Indentation is 2 spaces in `index.html`; the `sangga*` files are written
  densely (minified-style, little whitespace). Match the surrounding file's
  style.

## How to verify changes

There are no automated tests. Verify visually:

- Open the file in a browser (`index.html` and `sangga_계산로.html` work as
  plain `file://` pages).
- `sangga.html` depends on the `window.storage` runtime and won't fully work as
  a bare `file://` page — that's expected; it's designed for the Artifacts
  environment.
- After touching the contact form, sanity-check the validation paths
  (missing name / phone / consent) and that the Formspree fetch still targets
  the correct endpoint.

## Git workflow

- Default branch: `main`. Feature work happens on dedicated branches
  (current: `claude/claude-md-docs-ZmYCy`).
- Commit with clear messages and push with `git push -u origin <branch>`.
- **Do not open a pull request unless explicitly asked.**

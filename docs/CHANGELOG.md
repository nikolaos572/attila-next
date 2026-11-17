# Changelog

This document records all changes made in `attila-next` since the initial upstream import from [zutrinken/attila](https://github.com/zutrinken/attila) (`ea30f558530e08ca5f59813c90581d5ab6de270c`), listed in chronological order.

To see the full diff of all changes since the original upstream code:

```bash
git diff 3cbf25b HEAD
```

---

## 1. Initial Upstream Import
- **Commit:** `3cbf25b` (`3cbf25bfce1bb8e4ea10221dc035eb608a9bae37`)
- **Date:** 2026-11-16 06:49:37 UTC
- **Title:** *Initial import of upstream attila*
- **What changed:** Imported all 78 tracked files from upstream `zutrinken/attila` (`master` branch at commit `ea30f558530e08ca5f59813c90581d5ab6de270c`) unchanged, along with the initial `attila-next` `README.md`.
- **Follow-up commit:** `b9a627a` (`b9a627ad0357bb56009ffff9e9b561404e152d6a`) — *Document initial import commit hash in README.md* (recorded `3cbf25b` in `README.md`).

---

## 2. License, Documentation Split (`docs/`), and Build Exclusions
- **Commit:** `157f7af` (`157f7afbb8619990cb4ff0bb8da93cb2674242c7`)
- **Date:** 2026-11-16 07:01:37 UTC
- **Title:** *Update LICENSE, README.md, and add docs for localization and development*
- **Files modified:** `LICENSE`, `README.md`, `Gruntfile.js`, `docs/DEVELOPMENT.md`, `docs/LOCALIZATION.md`
- **What changed & why:**
  - Added `Copyright (c) 2026 Nikolaos Baras (@nikolaos572)` to `LICENSE` and `README.md` alongside the original upstream MIT copyright notice.
  - Restored the theme screenshot (`src/screenshot.webp`) and Features list to `README.md`.
  - Moved the Localization status table into `docs/LOCALIZATION.md` and the build instructions into `docs/DEVELOPMENT.md` to keep `README.md` concise.
  - Added `'!docs'` and `'!docs/**'` to the `compress` task in `Gruntfile.js` so documentation files are excluded when packaging `dist/attila.zip`.

---

## 3. Centered Header Navigation, Sticky Table of Contents, Reading Progress Bar, and Bug Fixes
- **Commit:** `8929cf1` (`8929cf15b496accce8f3ca7564be999611701e7f`)
- **Date:** 2026-11-16 09:06:07 UTC
- **Title:** *Center header nav, add sticky Table of Contents, and fix reading progress bar*
- **Files modified:** `src/sass/style.scss`, `assets/css/style.css`, `post.hbs`, `page.hbs`, `locales/en.json`
- **What changed & why:**
  1. **Centered Top Header Navigation (`src/sass/style.scss`):**
     - Added `max-width: calc(84rem + 8vw); margin: 0 auto; box-sizing: border-box;` to `.nav-header .nav-wrapper` so the top navigation links and action buttons gravitate toward the horizontal center column instead of being pushed to the far edges of wide monitors.
  2. **Sticky Table of Contents on Posts (`post.hbs`, `src/sass/style.scss`, `locales/en.json`):**
     - Added `<aside class="toc-container">` containing a collapsible `<details class="toc" open>` card above `.post-content` in `post.hbs`.
     - Added JavaScript in `post.hbs` that scans `.post-content` for `h1, h2, h3, h4` headings, assigns unique `id` slugs if missing, builds a nested bullet list (`•`, `◦`, `▪`), and reveals `.toc-container` via `.has-toc`. If a post has no headings, the Table of Contents remains hidden (`display: none`).
     - Styled `.toc-container` to float as a sticky right-hand sidebar (`position: sticky; top: 4rem`) on wide viewports (`>= 1300px`) and sit inline above the post content on narrower viewports (`< 1300px`).
  3. **Top Blue Reading Progress Bar (`post.hbs`, `src/sass/style.scss`):**
     - Changed `.progress-bar` color from `var(--ghost-accent-color)` to `#2997ff` (blue).
     - Removed `.progress-container.complete { transform: translateY(-3px); }` from `src/sass/style.scss`, which previously hid the progress bar off-screen when reaching the bottom of a post.
     - Replaced the 50ms `throttle` scroll listener (which lacked a trailing call and could drop the final scroll frame) with `requestAnimationFrame` calculating `(scrollTop / (scrollHeight - innerHeight)) * 100`, so the bar smoothly fills from `0%` at the top to `100%` at the bottom.
  4. **Bug Fix — CSS `position: sticky` compatibility (`src/sass/style.scss`):**
     - Changed `overflow-x: hidden` to `overflow-x: clip` on `html` and `body`. In CSS, setting `overflow-x: hidden` on both `html` and `body` broke `position: sticky` inside `body`.
  5. **Bug Fix — Single-line code block line numbers (`post.hbs`, `page.hbs`):**
     - Fixed an off-by-one calculation (`if (lines > 1) lines++;` followed by `length: lines - 1`) that caused 1-line `<pre><code>` blocks (`lines === 1`) to render an empty `.lines` gutter with `0` line numbers instead of `1`.
- **How to revert:** `git revert 8929cf1` (note: revert `df6f40b` first as it builds on the `.toc-container` styles).

---

## 4. Constrain Wide, Full-Width, and Gallery Images When Table of Contents is Present
- **Commit:** `df6f40b` (`df6f40b21df7dd4b94c3ef0d9de3e6a9f3dfc705`)
- **Date:** 2026-11-16 09:18:39 UTC
- **Title:** *Constrain wide, full, and gallery cards when Table of Contents is present*
- **Files modified:** `src/sass/style.scss`, `assets/css/style.css`
- **What changed & why:**
  - Added `.toc-container.has-toc + .post-content` rules to cap `.kg-image-card.kg-width-wide`, `.kg-image-card.kg-width-full`, and `.kg-gallery-card` to `max-width: 100% !important; width: 100%;` and reset their `figcaption` horizontal padding/underline offset to `0`.
  - Prevents wide/full-width images from expanding into the right margin and colliding with the sticky Table of Contents on posts that have headings. Posts without headings and static pages keep the original wide/full breakout behavior.
- **How to revert:** `git revert df6f40b` and run `npx grunt build`.

---

## 5. On-Demand GitHub Action Workflow for Packaging Theme
- **Commit:** `42fdd27` (`42fdd27450904c67a0a1e572df5d645487326f72`) on `main`, merged into branch via `3915fc5` (`3915fc58ad068535d7f64a13377e8051f7ed3110`)
- **Date:** 2026-11-16 12:54:17 UTC
- **Title:** *Add on-demand GitHub Action workflow to build and package theme*
- **Files modified:** `.github/workflows/package-theme.yml`, `Gruntfile.js`
- **What changed & why:**
  - Added `.github/workflows/package-theme.yml` triggered manually via `workflow_dispatch` to run `npm ci`, `npx grunt build`, and `npx grunt compress`, and upload the ready-to-install Ghost theme artifact with `retention-days: 1` (24 hours).
  - Added `'!.github'` and `'!.github/**'` to `Gruntfile.js` so workflow files are excluded from the packaged theme zip.
- **How to revert:** `git revert 42fdd27`

---

## 6. Centered Footer Navigation Wrapper
- **Commit:** `1f11a75` (`1f11a7569f65b17f386d9f94beaeca9ea091c3b4`)
- **Date:** 2026-11-16 13:13:45 UTC
- **Title:** *Center footer navigation wrapper horizontally to match header*
- **Files modified:** `src/sass/style.scss`, `assets/css/style.css`
- **What changed & why:**
  - Added `max-width: calc(84rem + 8vw); margin: 0 auto; box-sizing: border-box;` to `.nav-footer .nav-wrapper` in `src/sass/style.scss` so the footer items (copyright, theme toggle, and "Published with Ghost") align horizontally with the centered top header instead of sitting at the far left and right edges of wide monitors.
- **How to revert:** `git revert 1f11a75` and run `npx grunt build`.

---

## 7. Hide `"Article — <Category>"` Label Above Post Title
- **Commit:** `a3fbafe` (`a3fbafe46d35a0313984bde54e0b9ea8daa30b44`)
- **Date:** 2026-11-16 13:31:15 UTC
- **Title:** *Comment out 'Article - Category' label above post title in post.hbs*
- **Files modified:** `post.hbs`
- **What changed & why:**
  - Wrapped `<span class="post-info">...</span>` in `post.hbs` (lines 12–19) inside a Handlebars block comment (`{{!-- ... --}}`) for a cleaner, more minimalist post header above the post title (`<h1 class="post-title">`).
  - The `.post-info` CSS styles in `src/sass/style.scss` were intentionally kept intact so the label can be restored at any time without rebuilding CSS.
- **How to restore / revert:**
  - Either remove the `{{!--` and `--}}` comment markers around `<span class="post-info">...</span>` in `post.hbs` (lines 12–19), or run:
    ```bash
    git revert a3fbafe
    ```

---

## 8. Reduce Post Header Title Font Size on Desktop and Mobile
- **Title:** *Reduce post header title font size on desktop and mobile*
- **Files modified:** `src/sass/style.scss`, `assets/css/style.css`
- **What changed & why:**
  - Reduced `.post-header .post-title` font size from `6em` (`60px`) to `4em` (`40px`) on desktop, and added a responsive rule setting `font-size: 3em` (`30px`) on mobile viewports (`@media only screen and (max-width: $breakpoint-medium)`, i.e. `<= 640px`).
  - Previously, `.post-header .post-title` was hardcoded to `6em` (`60px`) with no mobile media query, causing long post titles to look disproportionately huge on desktop and wrap across many hyphenated lines on narrow mobile screens.
- **How to revert:** Restore `font-size: 6em;` on `.post-header .post-title` in `src/sass/style.scss` and run `npx grunt build`.

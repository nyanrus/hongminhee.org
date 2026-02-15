Repository guidelines
=====================

Overview
--------

Personal homepage for Hong Minhee (hongminhee.org).  Flat-structured,
framework-free site using vanilla PHP, HTML, CSS, and JavaScript.  No build
step, no package managers, no preprocessors.  PHP handles server-side
language negotiation; everything else is static.


Project structure
-----------------

~~~~
index.php                  Main entry — language negotiation, includes
                           the best-matching index.*.html
index.en.html              English page
index.ja.html              Japanese page
index.ko-Hang-KR.html      Korean (Hangul-only) page
index.ko-Kore.html         Korean (mixed Hanja/Hangul) page
_head.html                 Shared <head> fragment (meta, feeds, SRI, analytics)
_languages.html            Language-switcher nav
_links.html                Social/service links (locale-aware PHP conditionals)
_pgp.html                  PGP public-key block
404.html                   Static 404 error page
style.css                  Global stylesheet (raw CSS)
projects.js                Client-side show/hide toggler for project list
seonbi/index.php           301 redirect to dahlia.github.io/seonbi/
pic.jpg                    Portrait photo
.github/workflows/
  publish.yaml             SFTP deploy + CloudFront invalidation
~~~~

Fragment files (`_*.html`) are included from `index.*.html` via PHP `include`.
They have access to `$LANG` set by `index.php`.  Despite `.html` extensions,
these files contain inline PHP and are executed by the PHP engine.


Build, lint, and test commands
------------------------------

There is no build step; files are served as-is.

### Local development server

~~~~ bash
php -S 127.0.0.1:8000
~~~~

Test language negotiation via `/?lang=en`, `/?lang=ko`, `/?lang=ja`, etc.

### Lint PHP (run before every commit)

~~~~ bash
php -l index.php && php -l seonbi/index.php
~~~~

No other linters are configured.

### No automated test suite

Manual browser verification only: default language selection, `?lang=`
overrides, mobile layout (breakpoint 990px), dark mode, project toggler.


Coding style
------------

### General

 -  **Indentation**: 2 spaces everywhere (HTML, CSS, PHP, JS, YAML).  No tabs.
 -  **Filenames**: lowercase.  Localized pages use `index.<BCP-47-tag>.html`.
    Shared fragments use underscore prefix (`_head.html`, `_links.html`).
 -  **Line endings**: LF (Unix).
 -  **No frameworks or build tools** unless discussed first.

### HTML

 -  Doctype: `<!DOCTYPE html>` (HTML5).
 -  Charset: `<meta charset="utf-8">` in `_head.html`.
 -  Inline PHP for dynamic content: `<?php echo $LANG ?>`.
 -  HTML comment pairs (`<!-- -->`) suppress whitespace between inline
    elements, especially in `_links.html`.
 -  Use `rel="me"` on social profile links (IndieWeb/fediverse verification).
 -  Set `lang` on `<html>` and on elements when the language differs from
    the page language (e.g., `lang="lzh"`).

### CSS

 -  Raw CSS only — no Sass, PostCSS, or CSS-in-JS.
 -  No CSS custom properties (variables); use literal values.
 -  Simple descriptive class names (`.toggler`, `.handle`, `.footnote`).
 -  Use `:lang()` pseudo-class for internationalized typography.
 -  Dark mode via `@media (prefers-color-scheme: dark)` blocks placed near
    the elements they affect (not consolidated into one block).
 -  Single responsive breakpoint at `max-width: 990px`.
 -  SRI hash for `style.css` is computed at request time by `sri_hash()` in
    `_head.html`; update `style.css` freely — the hash auto-recomputes.

### JavaScript

 -  Vanilla JS only.  No libraries, no module bundlers.
 -  Use `const` (no `let` or `var`).  Comma-separated `const` declarations
    are acceptable for related variables.
 -  Named `function` declarations for callbacks.
 -  DOM access via `getElementsByClassName`, `createElement`,
    `createTextNode`, etc.  No `querySelector` in current code.
 -  Event binding via `.onclick` assignment.
 -  Semicolons at end of statements.
 -  `<script>` tags are placed inline in the HTML where the relevant DOM
    subtree is already parsed (no `DOMContentLoaded` needed).

### PHP

 -  Procedural style only — no classes or OOP.
 -  Function names use `snake_case`.
 -  Type hints on parameters (`string`, `array`).  Return types on
    signatures (`: array`, `: int`).
 -  Use `<?php if (...): ?>...<?php endif ?>` alternative syntax for
    conditional output in templates.
 -  Prefer `===` for comparisons (strict equality).
 -  No external dependencies (no Composer).  No error handling needed.


Localization
------------

When adding or editing content, update **all four** `index.*.html` files
plus any affected `_*.html` fragments.

| Tag          | File                    | Notes                         |
| ------------ | ----------------------- | ----------------------------- |
| `en`         | `index.en.html`         | English                       |
| `ja`         | `index.ja.html`         | Japanese (uses ruby/furigana) |
| `ko-Hang-KR` | `index.ko-Hang-KR.html` | Korean, Hangul only           |
| `ko-Kore`    | `index.ko-Kore.html`    | Korean, mixed Hanja/Hangul    |

`ko-Kore` and `ko-Hang-KR` must be kept in exact 1:1 correspondence: the
two pages are linguistically identical and differ **only** in script (Hanja
mixed vs. Hangul only).  Any content change to one must be mirrored in the
other.

Locale-specific labels inside shared fragments use inline PHP conditionals
checking `$LANG`.  Toggler labels in `projects.js` come from `data-show` and
`data-hide` HTML attributes set per locale.


Commit & pull request guidelines
--------------------------------

 -  **Commit messages**: short imperative fragments, 2–5 words typical.
    Examples from history: `Codeberg link`, `Reorganize projects`,
    `Add a link to Hackers' Pub`.
 -  Group related content, style, and script changes in a single commit.
 -  PRs should note user-visible changes, affected `index.*.html` files,
    screenshots if layout changed, and deployment or secret implications.


Deployment
----------

Every push to any branch triggers `.github/workflows/publish.yaml`:
checkout, SFTP upload, CloudFront cache invalidation (`/*`).  Relies on
8 GitHub Actions secrets.  Review workflow and secret changes carefully —
every push deploys.

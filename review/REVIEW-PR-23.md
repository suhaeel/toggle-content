# Review: PR #23 — Keep the block icon visible when the editor highlights it

- PR: https://github.com/EssentialBlocks/toggle-content/pull/23
- Branch reviewed: `fix/block-icon-selected` (copy: `review/toggle-content-pr-23`), base `dev`
- Reviewer: Abdullah Aas Suhaeel
- Verdict: **Approve** (2 optional nits below)

## What the PR does
The Toggle Content icon is filled `#6C3BFF`. On the blue highlighted row of the `/` inserter, a selected List View row and the drag chip, that purple almost disappears. The PR adds `assets/css/block-icon-selected.css` (recolours those fills to `currentColor`) and enqueues it on `enqueue_block_editor_assets`.

## Scope check
3-way merge into `origin/dev` (`git merge-tree`): conflict-free, 2 files, +56 lines (the CSS file and `toggle-content.php`). No change to `dist/`, `controls`, `lib/` or saved block markup, so existing posts cannot be affected.

## How it was tested
Local site, WordPress 7.1.3, PHP 8.2.29, headless Chrome 153, plugin linked in and activated. The PR author states this plugin was not tested in a browser (only accordion-toggle and infobox), so it was tested here. "Before" = same session with the stylesheet request blocked.

| Where | Before | After |
|---|---|---|
| `/` inserter highlighted row (bg `rgb(56,88,233)`) | icon fill `rgb(108,59,255)` | `rgb(239,240,242)` |
| Selected List View row | `rgb(108,59,255)` | `rgb(255,255,255)` |

Screenshots: `review/screenshots/pr23-*.png`. Before, the icon is lost on the blue row; after, it is clearly visible.

Other checks
- `php -l toggle-content.php`: no syntax errors (PHP 8.2.29).
- This icon differs from Testimonial's: `src/icon.js` has one `<g fill="#6C3BFF">` holding two paths, plus a `fill="#fff"` path inside a `<clipPath>` in `<defs>`. The descendant selector matches the `<g>` and the paths inherit it, confirmed by the computed fill above. The `#fff` rule only targets direct children of `<svg>`, so the clip path is correctly left alone.
- Stylesheet loads once, editor only. No console errors or PHP notices.
- Not tested: the drag chip (same rule set, only the colour differs) and browsers other than Chrome.

## Findings
No blocking bugs.
1. (nit) `'1.0.0'` is hard-coded as the style version. The handle is shared across the standalone plugins, so the first plugin to enqueue wins; a later CSS change can be served from cache.
2. (nit) The `#C5B1FF` and white-knockout rules are unused by this icon. Harmless dead rules in a shared file.
3. (info) The PR says the rules match Essential Blocks' `block-icon-selected.css`; that file is not present in the local `essential-blocks-for-gutenberg` `main`, so the claim could not be verified.

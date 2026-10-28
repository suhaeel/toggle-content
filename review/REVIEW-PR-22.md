# Review: PR #22 — Replace deprecated `DOMNodeInserted` listener in `eb-animation-load.js`

- PR: https://github.com/EssentialBlocks/toggle-content/pull/22
- Branch reviewed: `fix/animation-mutation-event` (copy: `review/toggle-content-pr-22`), base `dev`
- Reviewer: Abdullah Aas Suhaeel
- Verdict: **Approve**, with a suggestion to rebase so the PR diff shows only the real change

## What the PR does
`assets/js/eb-animation-load.js` attached a `DOMNodeInserted` listener for the admin panel. The PR replaces it with one capture-phase `change` listener on `document` that checks `event.target.id === 'eb-animation-style'`.

## "19 files" is a display artifact
The branch was cut from `fix/audit-followups-1.3.1`, squash-merged into `dev` as #21, so GitHub still lists that PR's commits. A 3-way merge into `dev` (`git merge-tree origin/dev origin/fix/animation-mutation-event`) is **conflict-free** and changes **1 file**: `assets/js/eb-animation-load.js`, +7 / -11.

Suggestion: `git rebase --onto origin/dev 9295bc5` so the PR shows just that one commit.

## Does it work? (tested in a real browser)
Headless Chrome 153, WordPress 7.1.3, Local site, plugin active.

1. `DOMNodeInserted` listeners registered while the editor loads: **1 before, 0 after**.
2. Functional test of the admin handler: add `<div class="eb___animated eb___fadeIn">` in view, add `<select id="eb-animation-style">` after page load (like the inspector), fire `change` on the select, wait 900 ms.

| JS | Div classes after `change` |
|---|---|
| old (`dev`) | `eb___animated eb___fadeIn` (nothing happened) |
| PR #22 | `eb__animated eb__fadeIn` (handler ran) |

So this is not only a console-warning cleanup: Chrome removed mutation events in Chrome 127, so on current Chrome the old admin handler **never ran**. The PR fixes a real functional bug. Caveat: the select was injected by script, not driven through the real inspector UI.

## Findings
No blocking bugs.
1. (suggestion) Rebase onto `dev` as above.
2. (info) The script is registered under the shared handle `essential-blocks-eb-animation`; the first standalone EB plugin to register it wins, so every standalone plugin needs this change before the problem is gone.
3. (info, pre-existing) `replaceAnimationClasses` queries only the main document, not the block-editor iframe canvas. Unchanged behaviour, out of scope.
4. (info) `setTimeout(..., 500)` is kept as before.

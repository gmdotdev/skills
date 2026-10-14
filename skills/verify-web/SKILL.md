---
name: verify-web
description: Verify a user-visible web change by running it like a visitor - build, serve or open the preview, drive it in a real browser, screenshot, and check it against the repo's feature map. Use before handing off any change to pages, styles, or the document head.
---

# Verify a web change

Tests prove logic. This skill proves the page: run the real build in a real browser and leave an artifact a reviewer can check faster than reading the diff. Do it before opening a PR for anything user-visible, and again on the PR preview.

## Run it

Build and serve the production output locally (the repo's `AGENTS.md` names the commands; usually `build`, then `preview`). For a PR or `main`, use the preview URL instead of a local server. For production, the live domain.

Drive it with a browser MCP when one is connected (Chrome DevTools: `new_page`, `take_snapshot`, `take_screenshot`, `list_console_messages`), otherwise headless Chrome:

```sh
chrome --headless=new --hide-scrollbars --window-size=1280,1000 \
  --virtual-time-budget=10000 --screenshot=/tmp/out.png <url>
```

Write screenshots to a scratch directory, not the repo. Shoot desktop (1280 wide) and mobile (about 390 wide).

## Feature map

The repo keeps a feature map: every feature, how to reach it, and what correct looks like. Keep it with the repo's docs (for example `docs/features.md`) and create it on first use:

```md
| Feature | Reach it | Correct looks like |
| --- | --- | --- |
| Home | `/` | ... |
| Not found | `/nope` | 404 page, and HTTP 404 (`curl -s -o /dev/null -w '%{http_code}' <url>/nope`) |
| Head | view-source | title, description, canonical, og tags; no console errors |
```

A PR that adds or changes a feature updates the map in the same PR. A stale map makes later verification confidently wrong.

## Checks that catch real bugs

- **Hydration.** For server-rendered or prerendered pages, click an internal link and confirm it navigates without a full reload. A page that does not hydrate looks correct but is dead.
- **Status codes.** Unknown paths answer 404, not a soft 200.
- **Head.** Title, description, canonical URL, and structured data are right per page (view-source, not the DOM).
- **Console.** No errors or failed requests.
- **Appearance.** Compare with production or the previous preview at both widths. Call out every difference; do not wave away "minor" ones.

## Report

Leave the proof in the PR: what you drove (rows from the map), the URL and commit it ran against, the screenshots, and anything you could not verify. Say so plainly when something failed or was skipped.

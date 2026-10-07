---
name: shopify-theme-check-fix
description: Run Shopify Theme Check on a theme, triage the results, and fix the safe issues automatically while flagging risky ones. Use when asked to lint a Shopify theme, clean up Theme Check warnings, or get a theme ready for review or handoff.
---

# Shopify Theme Check Fix

## Run

1. Locate the theme root (has `sections/`, `snippets/`, `layout/`).
2. Run `shopify theme check --path PATH_TO_THEME --output json > theme-check.json`. If the CLI is missing, tell the user to install it (`npm install -g @shopify/cli`) and stop; do not guess at results.
3. The JSON is a list of files, each with an `offenses` array (`check`, `severity`, `message`, `start`). Summarize total offenses by severity and by check name.

## Triage

Sort offenses into:

- **Safe to auto-fix**:
  - `DeprecatedFilter`: swap to the suggested filter. For `img_url` on an img element, replace the whole tag with `{{ image | image_url: width: N | image_tag: alt: ... }}`, which also clears `ImgWidthAndHeight`.
  - `ImgWidthAndHeight`: add `width`/`height` from the image object (`image.width`, `image.height`), or switch to `image_tag`.
  - `UnusedAssign`: delete the assign if nothing else (including `{% render %}` arguments) uses it.
  - `TranslationKeyExists`: add the key to `locales/en.default.json`. Edit the file surgically (insert one line); never re-serialize the whole JSON, which reorders and reformats it.
  - `VariableName`: rename to snake_case within the file.
- **Needs judgement** (list and ask):
  - `ParserBlockingScript`: adding `defer` breaks inline scripts that run before the file loads; check for inline usage first.
  - `RemoteAsset`: third-party CDNs; often an app or a client requirement.
  - `OrphanedSnippet`: may be rendered by apps or dynamic names.
  - `LiquidComplexity`, `MaxFileSize`, `AssetSize*`: refactors, not fixes.
  - Anything in `layout/theme.liquid`, since it affects every page.
- **Ignore with reason**: offenses in files the theme vendor owns that the user doesn't want to diverge from upstream — only with the user's agreement.

## Fix

- Fix the safe group file by file. Keep each change minimal; do not reformat untouched lines.
- Re-run Theme Check after fixing and confirm the counts dropped and no new offenses appeared.
- For the judgement group, list each with a one-line recommendation and ask before changing.
- If the user wants some checks silenced, add them to `.theme-check.yml` with a comment saying why, rather than deleting code.

## Deliver

Report before/after counts, the files changed, and the open judgement items.

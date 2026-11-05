---
name: shopify-theme-check-fix
description: Run Shopify Theme Check (shopify theme check) on a Liquid theme, triage the offenses, fix the safe ones and flag the risky ones for a decision. Use when asked to lint a Shopify theme, clean up Theme Check warnings or errors, or get a theme ready for review or handoff.
license: MIT
metadata:
  version: "1.1.0"
---

# Shopify Theme Check Fix

## Before you start

- This skill edits theme files in place. Check that the theme is in git with a clean working tree (`git status`), or that the user has a copy. If neither is true, ask the user to commit or copy the theme first and wait, so every change can be reviewed and reverted.
- `shopify theme check --auto-correct` is Shopify's own fixer and covers only a few checks, such as `DeprecatedFilter`. Do not run it unless the user asks. This skill makes small edits by hand and leaves the judgement calls to the user.

## Run

1. Locate the theme root (has `sections/`, `snippets/`, `layout/`).
2. Run `shopify theme check --path PATH_TO_THEME --output json > theme-check.json`. A non-zero exit code usually means offenses at or above the fail level were found, but a bad path or a crash also gives one, so read the file. If it is empty or not valid JSON, report the error output and stop. If the CLI is missing, tell the user to install it (`npm install -g @shopify/cli@latest`, which needs Node.js 22.12 or later and Git 2.28 or later) and stop; do not guess at results.
3. The JSON is a list with one entry per file that has offenses: `path`, an `offenses` array and per-severity counts. Each offense has `check`, `severity`, `message`, `start_row`, `start_column`, `end_row` and `end_column`. Row and column numbers may start at 0 (the CLI source treats rows as zero-based) rather than 1. Read the file first to confirm the keys in the installed CLI version, and check the first offense against the file before relying on a row number. Summarize total offenses by severity and by check name.

## Triage

Sort offenses into:

- **Safe to auto-fix**:
  - `DeprecatedFilter`: swap to the suggested filter. For `img_url` on an img element, replace the whole tag with `{{ image | image_url: width: N | image_tag: alt: ... }}`, which also clears `ImgWidthAndHeight`. `image_tag` writes its own markup (`width`, `height` and `srcset`), so pass the old tag's `class`, `loading` and other plain attributes to it as arguments. If the tag has `data-*` attributes, an inline `style` or event handlers, or is a hook for a script (a lazy-loading library, for example), move it to the judgement list instead.
  - `ImgWidthAndHeight`: add `width`/`height` from the image object (`image.width`, `image.height`), or switch to `image_tag` (same attribute rule as above).
  - `UnusedAssign`: delete the assign if nothing else (including `{% render %}` arguments) uses it. Theme Check looks at one file at a time, so search the theme for the name first. If another file uses it (snippets pulled in with the deprecated `{% include %}` share variables with the file that includes them), move it to the judgement list instead.
  - `TranslationKeyExists`: add the key to `locales/en.default.json`. Edit the file surgically (insert one line); never re-serialize the whole JSON, which reorders and reformats it.
  - `VariableName`: rename to snake_case within the file. Search the theme for the old name first. If another file uses it (snippets pulled in with the deprecated `{% include %}` share variables with the file that includes them), move it to the judgement list instead.
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

Report before/after counts, the files changed, and the open judgement items. If the theme is in git, point the user to `git diff` to review the edits. Delete `theme-check.json` unless the user wants to keep it, so it does not end up in a commit.

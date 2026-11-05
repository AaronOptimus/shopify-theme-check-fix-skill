# Changelog

Newest version first.

## 1.1.0

Packaging release. The triage categories are the same (fixed directly, listed for your decision, ignored with a reason), with three exceptions that are now held back for your decision: a `VariableName` rename or an `UnusedAssign` deletion when another file uses the variable, and replacing an `<img>` tag with `image_tag` when the tag has custom attributes or is a hook for a script.

### Changed

- Moved the skill from `shopify-theme-check-fix/` to `skills/shopify-theme-check-fix/`. `skills/<name>/SKILL.md` is the layout documented for Claude Code plugins and the skills CLI, and the one the plugin directories look for. A copy you made from the old folder keeps working. To update, copy from the new path.
- Rewrote the skill description so it names Theme Check, Liquid themes and the usual ways of asking for this job.
- Added `license` and `metadata.version` to the SKILL.md frontmatter.
- Rewrote the README: what the skill does, who it is for, how it relates to Shopify's own tools, install steps for Claude Code, Cursor and the skills CLI, prerequisites, example prompts, troubleshooting, a note to run it only on themes you trust, and a disclosure of the paid pack.

### Added

- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`, so Claude Code can install the skill as a plugin from this repository.
- SKILL.md: a first step that asks the user to confirm the theme is committed or copied before anything is edited.
- SKILL.md: a note that `shopify theme check --auto-correct` is Shopify's own fixer, covers only a few checks, and is not run unless the user asks.
- SKILL.md: before a `VariableName` rename or an `UnusedAssign` deletion, search the theme for the name. If another file uses it, list it for a decision instead. `UnusedAssign` needs this because Theme Check looks at one file at a time and does not follow `{% include %}`, so a variable that only an included snippet reads is reported as unused.
- SKILL.md: when an `<img>` tag is replaced with `image_tag`, pass the old tag's `class`, `loading` and other plain attributes as arguments. If the tag has `data-*` attributes, an inline `style` or event handlers, or is a hook for a script, list it for a decision instead.
- SKILL.md: at the end, point to `git diff` and delete `theme-check.json` unless the user wants to keep it.
- This changelog.

### Fixed

- SKILL.md named a `start` key in the Theme Check JSON. The CLI writes `start_row`, `start_column`, `end_row` and `end_column`. Shopify's own source describes the row base (0 or 1) in two different ways, so the text now tells the agent to confirm the keys in the installed CLI version and to check the first offense against the file before relying on a row number.
- SKILL.md now says that a non-zero exit code from `shopify theme check` usually means offenses were found, not that the command failed. A bad path or a crash also gives one, so the skill reads the output file and stops with the error output if the file is empty or not valid JSON.
- Install steps now give exact commands for macOS, Linux and Windows PowerShell, and create the target folder first.

## 1.0.0

First public version (October 2026). One skill, `shopify-theme-check-fix/SKILL.md`, MIT licensed. It runs Shopify Theme Check, sorts the offenses into safe to auto-fix, needs judgement and ignore with reason, fixes the safe group file by file, and reports before and after counts.

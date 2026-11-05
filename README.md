# Shopify Theme Check Fix

A free agent skill for Claude Code and Cursor that runs Shopify Theme Check on a Liquid theme, fixes the offenses that are safe to fix, and lists the risky ones with a recommendation so you can decide.

It is one `SKILL.md` file in the open [Agent Skills](https://agentskills.io) format, released under the MIT license. It contains no scripts.

## Who it is for

Developers and freelancers who work on Shopify Liquid themes (Online Store 2.0 themes such as Dawn or Horizon) and already use Claude Code or Cursor. It is meant for themes where Theme Check reports a long list and you want the mechanical items cleared while the riskier ones are left for you to decide.

## What it does

1. Runs `shopify theme check --output json` on the theme and summarizes the offenses by severity and by check.
2. Fixes these directly, one file at a time, with minimal diffs: `DeprecatedFilter`, `ImgWidthAndHeight`, `UnusedAssign`, `TranslationKeyExists`, `VariableName`. Three cases are held back for your decision instead:
   - a `VariableName` rename, when another file uses the variable;
   - an `UnusedAssign` deletion, when another file uses the variable (Theme Check looks at one file at a time and does not follow `{% include %}`, which shares variables between files);
   - replacing an `<img>` tag with `image_tag`, when the tag has custom attributes or is a hook for a script.
3. Lists these with a one-line recommendation and asks before changing anything: `ParserBlockingScript` (adding `defer` can break inline scripts), `RemoteAsset`, `OrphanedSnippet` (an app may render the snippet), `LiquidComplexity`, `MaxFileSize`, `AssetSize*`, and anything in `layout/theme.liquid`.
4. Runs Theme Check again and reports the before and after counts, the files it changed and the decisions still open.

If you ask it to silence a check, it adds the check to `.theme-check.yml` with a comment saying why, instead of deleting code.

## How it relates to Shopify's own tools

This skill sits on top of Shopify's tools. It does not replace them.

- `shopify theme check` finds the offenses, and the skill needs it.
- `shopify theme check --auto-correct` fixes offenses automatically. As of November 2026, Shopify's [checks reference](https://shopify.dev/docs/storefronts/themes/tools/theme-check/checks) marks only four checks as auto-correctable (`DeprecatedFilter`, `MissingTemplate`, `MatchingTranslations`, `RequiredLayoutThemeObject`). If those are the offenses you care about, run the command yourself and skip this skill.
- Shopify's [AI Toolkit](https://shopify.dev/docs/apps/build/ai-toolkit) connects coding agents to Shopify's developer docs and API schemas and validates Liquid against Shopify's schemas. Use it when you want current docs or validation inside your agent. This skill provides neither.

What this skill adds is a written routine for the rest: which offenses an agent may fix directly, which ones it must list and ask about, and what the final report contains. That routine is a judgement about risk, so read `skills/shopify-theme-check-fix/SKILL.md` before you rely on it.

## Prerequisites

- A theme checked out locally (a folder with `sections/`, `snippets/` and `layout/`), ideally in git.
- Shopify CLI: `npm install -g @shopify/cli@latest`. Shopify lists Node.js 22.12 or later and Git 2.28.0 or later as the requirements ([install notes](https://shopify.dev/docs/api/shopify-cli)).
- Claude Code, or Cursor 2.4 or newer.

## Install

The skill is the folder `skills/shopify-theme-check-fix`. Pick one route.

### Claude Code, manual copy

macOS or Linux:

```bash
git clone https://github.com/AaronOptimus/shopify-theme-check-fix-skill.git
mkdir -p ~/.claude/skills
cp -r shopify-theme-check-fix-skill/skills/shopify-theme-check-fix ~/.claude/skills/
```

Windows PowerShell:

```powershell
git clone https://github.com/AaronOptimus/shopify-theme-check-fix-skill.git
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse shopify-theme-check-fix-skill\skills\shopify-theme-check-fix "$HOME\.claude\skills\"
```

Claude Code picks up new skills in a running session. If `~/.claude/skills` did not exist when the session started, run `/reload-skills` or restart Claude Code. To install for one project only, copy the folder into `.claude/skills/` inside that project instead.

### Claude Code, plugin marketplace

This repository is also a one-plugin marketplace.

```bash
claude plugin marketplace add AaronOptimus/shopify-theme-check-fix-skill
claude plugin install shopify-theme-check-fix@aaronoptimus-skills
```

Inside a Claude Code session, `/plugin marketplace add AaronOptimus/shopify-theme-check-fix-skill` registers the marketplace, and `/plugin install shopify-theme-check-fix@aaronoptimus-skills` opens the plugin in the `/plugin` panel, where you install it.

### Cursor

Cursor 2.4 or newer loads skills from `~/.cursor/skills/` (all projects) and `.cursor/skills/` (one project). It also reads `~/.claude/skills/`, so the Claude Code copy above should also load in Cursor. This follows Cursor's documentation; the skill has not been run in Cursor yet. To install for Cursor only:

macOS or Linux:

```bash
git clone https://github.com/AaronOptimus/shopify-theme-check-fix-skill.git
mkdir -p ~/.cursor/skills
cp -r shopify-theme-check-fix-skill/skills/shopify-theme-check-fix ~/.cursor/skills/
```

Windows PowerShell:

```powershell
git clone https://github.com/AaronOptimus/shopify-theme-check-fix-skill.git
New-Item -ItemType Directory -Force "$HOME\.cursor\skills" | Out-Null
Copy-Item -Recurse shopify-theme-check-fix-skill\skills\shopify-theme-check-fix "$HOME\.cursor\skills\"
```

Keep the folder name as it is. In Cursor the folder name must match the skill name. Type `/` in Agent chat and pick `shopify-theme-check-fix`, or describe the job.

### skills CLI

The [skills CLI](https://github.com/vercel-labs/skills) installs a skill into the agents you choose. It needs Node.js 22.20 or later.

```bash
npx skills add AaronOptimus/shopify-theme-check-fix-skill --list
npx skills add AaronOptimus/shopify-theme-check-fix-skill
```

The first command only lists what the repository offers. The second installs the skill and may ask which agents to install into. Add `-g` to install for all projects instead of the current one, and `-a claude-code` or `-a cursor` to name an agent. The CLI sends anonymous usage data unless you set `DISABLE_TELEMETRY=1` or `DO_NOT_TRACK=1`.

## Use it

Open the agent in the theme folder and ask in plain words. For example:

- "Clean up Theme Check on this theme."
- "Run Theme Check, fix the safe offenses and list the rest for me."
- "Get this theme ready for handoff: lint it and tell me what is left."

To call the skill by name, type `/shopify-theme-check-fix`. If you installed it as a plugin, Claude Code puts the plugin name in front, so the command is `/shopify-theme-check-fix:shopify-theme-check-fix`.

## What it runs and changes

The skill runs `shopify theme check`, writes its output to `theme-check.json` in the folder where the command runs and deletes that file at the end unless you ask to keep it. It edits files inside the theme folder: `.liquid` files, `locales/en.default.json`, and `.theme-check.yml` if you ask it to silence a check. If the theme is in git it may run read-only git commands such as `git status` and `git diff`. The instructions do not ask the agent to make network calls, read credentials or touch a live store. Shopify CLI is a separate tool with its own data practices.

Because the skill edits files in place, it asks you to confirm that the theme is committed or copied before it starts. Review the diff afterwards. No test results for specific themes are published here, so try it on a copy first.

Only run it on themes you trust. Shopify's [Theme Check configuration page](https://shopify.dev/docs/storefronts/themes/tools/theme-check/configuration) says a theme's `.theme-check.yml` can load custom checks from a path (the `require` setting), and custom checks are code.

## Troubleshooting

- **The skill does not show up.** In Claude Code, run `/skills` to see what it found. If you created `~/.claude/skills` after the session started, run `/reload-skills` or restart Claude Code. In Cursor, use version 2.4 or newer, keep the folder name `shopify-theme-check-fix` and restart Cursor, which looks for skills when it starts.
- **The agent says Shopify CLI is missing.** Install it with `npm install -g @shopify/cli@latest` (it needs Node.js 22.12 or later and Git 2.28 or later), then ask again.
- **The agent stops and asks you to commit or copy the theme.** That is the check in "Before you start" in `SKILL.md`. Commit the theme or make a copy, then ask again.
- **The agent stops because the Theme Check output is empty or not valid JSON.** Run `shopify theme check --path PATH_TO_THEME` yourself to see the error message.

## Changes

See [CHANGELOG.md](CHANGELOG.md).

## Support

Bug reports and suggestions are welcome as GitHub issues in this repository.

## License

MIT, see [LICENSE](LICENSE). The license covers this skill only.

## The paid pack

This skill is also one of six in the Shopify Freelancer Skill Pack, a paid product ($39) by the same author: https://tuttler05.gumroad.com/l/xdljum

The other five cover a client-ready store audit, handoff docs, a section builder, metafields wiring and App Store review prep. They are not in this repository, and nothing here needs the pack.

Independent project, not affiliated with Shopify, Anthropic or Cursor.

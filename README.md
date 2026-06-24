# REES46 Skills

AI-agent skills for working with REES46 SDKs.

Russian version: [README.ru.md](README.ru.md).

## What Is Included

This repository contains installable skill directories. Each skill directory starts with `SKILL.md` and may include extra reference files.

Available skills:

- `js-sdk/` - integration, review, and debugging guidance for REES46 JS SDK v3.

Skill structure:

```text
js-sdk/
  SKILL.md
  references/
    api.md
    examples.md
    common-mistakes.md
```

Install the whole skill directory, not only `SKILL.md`. The agent needs the `references/` files for API details, examples, and review checklists.

## Recommended: Install With CLI

The easiest way to install the skills is the `skills` CLI:

```bash
npx skills add rees46/skills --skill '*' -g
```

This installs all skills globally and lets the CLI detect or ask which agents to target.

To install for a specific agent:

```bash
# Codex
npx skills add rees46/skills --skill '*' -g -a codex

# Claude Code
npx skills add rees46/skills --skill '*' -g -a claude-code

# Cursor
npx skills add rees46/skills --skill '*' -g -a cursor

# Windsurf
npx skills add rees46/skills --skill '*' -g -a windsurf
```

For a non-interactive install, add `-y`:

```bash
npx skills add rees46/skills --skill '*' -g -a codex -y
```

To list skills before installing:

```bash
npx skills add rees46/skills --list
```

`npx add-skill` may still work as a compatibility alias, but the old `add-skill` npm package is deprecated. Use `npx skills add` for new installs.

## Update With CLI

If you installed the skills with `npx skills add`, update them with:

```bash
npx skills update -g
```

To update non-interactively:

```bash
npx skills update -g -y
```

To update only a specific skill:

```bash
npx skills update rees46-js-sdk-v3 -g
```

Restart your agent after updating so it reloads the skill files.

## Manual Installation

Use manual installation if you cannot use `npx`, need to inspect files first, or want full control over where the files are placed.

### 1. Download The Skills

Start by cloning this repository:

```bash
git clone https://github.com/rees46/skills.git
cd skills
```

If you use SSH for GitHub:

```bash
git clone git@github.com:rees46/skills.git
cd skills
```

### 2. Install For Your Agent

Choose the section for the agent you use.

#### Codex

Codex loads user skills from `$CODEX_HOME/skills`. If `CODEX_HOME` is not set, use `~/.codex`.

Install all REES46 skills:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "${CODEX_HOME:-$HOME/.codex}/skills/"
done
```

Verify the installation:

```bash
find "${CODEX_HOME:-$HOME/.codex}/skills/js-sdk" -maxdepth 2 -type f
```

Restart your Codex session after installation.

#### Claude Code

If your Claude Code version supports local skills, install the skill directory into your Claude skills directory. A common local setup is:

```bash
mkdir -p "$HOME/.claude/skills"
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "$HOME/.claude/skills/"
done
```

Restart Claude Code after installation.

If your Claude Code version does not load `SKILL.md` directories directly, add the REES46 skill to your project instructions instead. From your project directory:

```bash
cat >> CLAUDE.md <<'EOF'

## REES46 JS SDK

When working with REES46 JS SDK v3, read and follow:

- /absolute/path/to/skills/js-sdk/SKILL.md
- /absolute/path/to/skills/js-sdk/references/api.md
- /absolute/path/to/skills/js-sdk/references/examples.md
- /absolute/path/to/skills/js-sdk/references/common-mistakes.md
EOF
```

Replace `/absolute/path/to/skills` with the path to your local clone of this repository.

#### Cursor, Windsurf, And Other IDE Agents

If your agent does not support `SKILL.md` as a native skill format, use the files as project rules.

1. Clone this repository somewhere stable on your machine.
2. Open your project in the IDE agent.
3. Add a project rule that points the agent to the REES46 skill files.

Example project rule:

```text
When adding, reviewing, or debugging REES46 JS SDK v3 code, follow these local instructions:

/absolute/path/to/skills/js-sdk/SKILL.md
/absolute/path/to/skills/js-sdk/references/api.md
/absolute/path/to/skills/js-sdk/references/examples.md
/absolute/path/to/skills/js-sdk/references/common-mistakes.md

Use only commands and parameter shapes documented in the skill references.
Do not invent REES46 SDK API calls.
```

Replace `/absolute/path/to/skills` with the path to your local clone.

## Symlink Installation

If you plan to update the skills often, install them as symlinks instead of copying files.

Codex:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
for skill in */SKILL.md; do
  skill_dir="${skill%/SKILL.md}"
  ln -sfn "$PWD/$skill_dir" "${CODEX_HOME:-$HOME/.codex}/skills/$skill_dir"
done
```

Claude Code:

```bash
mkdir -p "$HOME/.claude/skills"
for skill in */SKILL.md; do
  skill_dir="${skill%/SKILL.md}"
  ln -sfn "$PWD/$skill_dir" "$HOME/.claude/skills/$skill_dir"
done
```

Restart the agent after creating the symlink.

## Manual Updating

If you installed the skills by copying files:

```bash
cd /path/to/skills
git pull
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "${CODEX_HOME:-$HOME/.codex}/skills/"
done
```

For Claude Code, replace the destination with your Claude skills directory:

```bash
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "$HOME/.claude/skills/"
done
```

If you installed the skills with symlinks:

```bash
cd /path/to/skills
git pull
```

Restart the agent after updating.

## Releases

Releases are managed by [release-please](https://github.com/googleapis/release-please-action).

On every push to `master`, release-please reads Conventional Commits and opens or updates a release PR. The release PR updates `CHANGELOG.md` and `version.txt`. Merging that release PR creates the GitHub Release and tag.

Version bump rules:

- `fix:` or `perf:` creates a patch release, for example `v1.0.1`;
- `feat:` creates a minor release, for example `v1.1.0`;
- `feat!:` / `fix!:` or `BREAKING CHANGE:` creates a major release, for example `v2.0.0`;
- commits such as `docs:`, `chore:`, `ci:`, `refactor:`, and `test:` do not create a release by themselves unless configured otherwise.

After release-please creates the GitHub Release, the workflow uploads `.tar.gz` and `.zip` assets containing only public skill files and README files. The private `docs` submodule is not checked out and is not included in release archives.

## Checking That It Works

Ask your agent to work on a task that clearly requires the REES46 SDK, for example:

```text
Check the REES46 JS SDK integration in this project and find initialization mistakes.
```

Expected behavior:

- the agent says it is using the REES46 JS SDK skill;
- the agent reads `js-sdk/SKILL.md`;
- the agent uses `references/api.md`, `references/examples.md`, and `references/common-mistakes.md` when needed;
- the agent does not invent undocumented SDK commands.

## Security

Skills are instructions for your agent. Read `SKILL.md` and all referenced files before installing third-party skills. Do not install skills that ask for secrets, change permissions, or run commands without a clear reason.

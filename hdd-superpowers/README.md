# hdd-superpowers (PoC)

Hypothesis-driven-development skill pack, mirroring `superpowers`' architecture for an
epistemic lifecycle instead of a design/implementation one:

    framing-hypotheses -> planning-experiments -> executing-experiments -> deciding-hypotheses

Design spec: see the accompanying `2026-08-13-hdd-superpowers-design.md` if included, or ask
the person who gave you this pack for it.

## Install

### Option A — from GitHub, with `npx skills` (recommended)

The [`skills`](https://github.com/vercel-labs/skills) CLI installs straight from the GitHub
link, into Claude Code, Codex, Cursor, OpenCode and others:

    npx skills add KhoaBang/hdd-superpowers-poc

It clones the repo, discovers all four skills from its top-level `skills/` folder, and — when
run inside a project — installs them to `.claude/skills/<name>/SKILL.md`, recording their
source in a `skills-lock.json`. Useful flags:

  - `-l, --list` — show what the repo offers without installing anything
  - `-s, --skill <name>` — install only some (repeatable; `'*'` for all)
  - `-a, --agent claude-code` — pick the target agent instead of auto-detecting
  - `-g, --global` — install user-level instead of project-level
  - `--copy` — copy the files instead of symlinking them
  - `-y, --yes` — skip the interactive picker

A non-interactive, single-skill install:

    npx skills add KhoaBang/hdd-superpowers-poc --skill framing-hypotheses --agent claude-code -y

In a running Claude Code session, `/reload-skills` picks up what was just installed. Later,
`npx skills update` refreshes and `npx skills remove` uninstalls. The CLI declares
`node >=22.20.0`.

### Option B — from GitHub, as a Claude Code plugin

This repo doubles as a plugin marketplace, so Claude Code can install the pack straight from
the GitHub link. In Claude Code, run:

    /plugin marketplace add KhoaBang/hdd-superpowers-poc
    /plugin install hdd-superpowers@hdd-superpowers-poc

`marketplace add` also accepts the full URL (`https://github.com/KhoaBang/hdd-superpowers-poc`)
if you prefer, or a local path to a clone of this repo. Update later with:

    /plugin marketplace update hdd-superpowers-poc

The four skills are discovered from the same top-level `skills/` folder — nothing needs to be
copied into `~/.claude/skills/`.

### Option C — from GitHub, by cloning into your skills directory

If your runtime supports neither, clone the repo and copy the skill folders in:

    git clone https://github.com/KhoaBang/hdd-superpowers-poc.git /tmp/hdd-poc
    cp -r /tmp/hdd-poc/skills/* ~/.claude/skills/

### Option D — from the packaged bundle

Extract the *contents* of this bundle's `skills/` folder — the four `<name>/` folders
themselves, not the `skills/` folder wrapping them — into your personal skills directory
(`~/.claude/skills/`), so you end up with:

    ~/.claude/skills/framing-hypotheses/SKILL.md
    ~/.claude/skills/planning-experiments/SKILL.md
    ~/.claude/skills/executing-experiments/SKILL.md
    ~/.claude/skills/deciding-hypotheses/SKILL.md

Do not copy the `skills/` folder itself in — that produces a nested
`~/.claude/skills/skills/<name>/SKILL.md`, which will not be picked up. For example:

    unzip hdd-superpowers-bundle.zip -d /tmp/hdd && cp -r /tmp/hdd/skills/* ~/.claude/skills/

Alternatively, each skill is also provided separately as its own standalone `.skill` file
(`<name>.skill`, distributed alongside this bundle rather than inside it) if your runtime
supports installing skills one at a time.

## Status

PoC. Deliberately missing a `reviewing-evidence` skill (evidence self-review is folded into
`executing-experiments`) and a `using-hdd` router skill (routing is hand-coded into each
skill's own trigger description and terminal-state section). Both are candidates to split out
if this PoC proves the core loop out across a handful of real hypotheses. See the design spec's
"Known limitations" section for the full list.

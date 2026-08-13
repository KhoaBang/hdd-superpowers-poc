# hdd-superpowers (PoC)

Hypothesis-driven-development skill pack, mirroring `superpowers`' architecture for an
epistemic lifecycle instead of a design/implementation one:

    framing-hypotheses -> planning-experiments -> executing-experiments -> deciding-hypotheses

Design spec: see the accompanying `2026-08-13-hdd-superpowers-design.md` if included, or ask
the person who gave you this pack for it.

## Install

### Option A — from GitHub, as a Claude Code plugin (recommended)

This repo doubles as a plugin marketplace, so Claude Code can install the pack straight from
the GitHub link. In Claude Code, run:

    /plugin marketplace add KhoaBang/hdd-superpowers-poc
    /plugin install hdd-superpowers@hdd-superpowers-poc

`marketplace add` also accepts the full URL (`https://github.com/KhoaBang/hdd-superpowers-poc`)
if you prefer, or a local path to a clone of this repo. Update later with:

    /plugin marketplace update hdd-superpowers-poc

The four skills are then discovered from the plugin's `skills/` folder — nothing needs to be
copied into `~/.claude/skills/`.

### Option B — from GitHub, by cloning into your skills directory

If your runtime does not support plugins, clone the repo and copy the skill folders in:

    git clone https://github.com/KhoaBang/hdd-superpowers-poc.git /tmp/hdd-poc
    cp -r /tmp/hdd-poc/hdd-superpowers/skills/* ~/.claude/skills/

### Option C — from the packaged bundle

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

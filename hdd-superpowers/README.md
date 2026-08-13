# hdd-superpowers (PoC)

Hypothesis-driven-development skill pack, mirroring `superpowers`' architecture for an
epistemic lifecycle instead of a design/implementation one:

    framing-hypotheses -> planning-experiments -> executing-experiments -> deciding-hypotheses

Design spec: see the accompanying `2026-08-13-hdd-superpowers-design.md` if included, or ask
the person who gave you this pack for it.

## Install

Extract the `skills/` folder from this bundle into your personal skills directory
(`~/.claude/skills/`), so you end up with:

    ~/.claude/skills/framing-hypotheses/SKILL.md
    ~/.claude/skills/planning-experiments/SKILL.md
    ~/.claude/skills/executing-experiments/SKILL.md
    ~/.claude/skills/deciding-hypotheses/SKILL.md

Alternatively, each skill also ships as its own standalone `.skill` file (`dist/<name>.skill`)
if your runtime supports installing skills one at a time.

## Status

PoC. Deliberately missing a `reviewing-evidence` skill (evidence self-review is folded into
`executing-experiments`) and a `using-hdd` router skill (routing is hand-coded into each
skill's own trigger description and terminal-state section). Both are candidates to split out
if this PoC proves the core loop out across a handful of real hypotheses. See the design spec's
"Known limitations" section for the full list.

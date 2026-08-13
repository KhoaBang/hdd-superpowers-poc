# hdd-superpowers-poc

Design workspace for **HDD-superpowers** — a proof-of-concept skill pack that mirrors the
[`superpowers`](https://github.com/obra/superpowers) skill pack's architecture, but replaces
the design/implementation lifecycle with a hypothesis/experiment/evidence lifecycle:

    framing-hypotheses -> planning-experiments -> executing-experiments -> deciding-hypotheses

Grounded in 7 invariants distilled from 5 sources (MIT CISR's "learn fast, not fail fast";
Hypothesis-Driven Skill Optimization; the Hypothesis Evolution Protocol; SkillHEX;
SkillHone) — see the design spec for full attribution.

## Install from GitHub

The 4 skills live in this repo's top-level `skills/` folder, which is both the plugin root and
the layout `npx skills` discovers — so either installer works off the GitHub link directly.

As a Claude Code plugin (this repo is its own marketplace, via `.claude-plugin/`):

    /plugin marketplace add KhoaBang/hdd-superpowers-poc
    /plugin install hdd-superpowers@hdd-superpowers-poc

Or with the [`skills`](https://github.com/vercel-labs/skills) CLI, for Claude Code and other
agents:

    npx skills add KhoaBang/hdd-superpowers-poc

See [`hdd-superpowers/README.md`](hdd-superpowers/README.md) for the clone-based and
bundle-based alternatives.

## Contents

- **`docs/superpowers/specs/2026-08-13-hdd-superpowers-design.md`** — the design spec: the
  7 invariants and their sources, the 4-artifact-type model (Hypothesis/Experiment/Evidence/
  Decision), the per-skill hard-gates, and what's explicitly deferred (`reviewing-evidence`,
  `using-hdd`, a GitHub Issues storage adapter).
- **`docs/superpowers/plans/2026-08-13-hdd-superpowers-poc.md`** — the implementation plan
  that turned the spec into the 4 `SKILL.md` files, executed via
  `superpowers:subagent-driven-development` (per-task review, one final-review fix wave).
- **`skills/<name>/SKILL.md`** — the actual deliverable: the 4 skills, at the repo root so that
  both the plugin loader and `npx skills` find them without any subpath.
- **`.claude-plugin/`** — `marketplace.json` (makes this repo an installable marketplace) and
  `plugin.json` (makes the repo root itself the plugin).
- **`hdd-superpowers/`** — packaging only: the `README.md` that ships inside the bundle, and
  `dist/` containing the packaged `.skill` archives and the combined bundle zip.

## Status

PoC. The pack ships exactly 4 skills; evidence self-review lives inside
`executing-experiments` rather than a separate `reviewing-evidence` skill, and there's no
`using-hdd` router — both are candidates to split out if the core loop proves itself across
5-10 real hypotheses. See the spec's "Known limitations" section for the full list, and the
plan's final-review notes for what was fixed vs. deliberately deferred.

# hdd-superpowers-poc

Design workspace for **HDD-superpowers** — a proof-of-concept skill pack that mirrors the
[`superpowers`](https://github.com/obra/superpowers) skill pack's architecture, but replaces
the design/implementation lifecycle with a hypothesis/experiment/evidence lifecycle:

    framing-hypotheses -> planning-experiments -> executing-experiments -> deciding-hypotheses

Grounded in 7 invariants distilled from 5 sources (MIT CISR's "learn fast, not fail fast";
Hypothesis-Driven Skill Optimization; the Hypothesis Evolution Protocol; SkillHEX;
SkillHone) — see the design spec for full attribution.

## Contents

- **`docs/superpowers/specs/2026-08-13-hdd-superpowers-design.md`** — the design spec: the
  7 invariants and their sources, the 4-artifact-type model (Hypothesis/Experiment/Evidence/
  Decision), the per-skill hard-gates, and what's explicitly deferred (`reviewing-evidence`,
  `using-hdd`, a GitHub Issues storage adapter).
- **`docs/superpowers/plans/2026-08-13-hdd-superpowers-poc.md`** — the implementation plan
  that turned the spec into the 4 `SKILL.md` files, executed via
  `superpowers:subagent-driven-development` (per-task review, one final-review fix wave).
- **`hdd-superpowers/`** — the actual deliverable: the 4 skill folders
  (`skills/<name>/SKILL.md`), a `README.md` with install instructions, and `dist/` containing
  the packaged `.skill` archives and a combined bundle zip.

## Status

PoC. The pack ships exactly 4 skills; evidence self-review lives inside
`executing-experiments` rather than a separate `reviewing-evidence` skill, and there's no
`using-hdd` router — both are candidates to split out if the core loop proves itself across
5-10 real hypotheses. See the spec's "Known limitations" section for the full list, and the
plan's final-review notes for what was fixed vs. deliberately deferred.

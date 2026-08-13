---
name: executing-experiments
description: Use when a frozen experiment contract (EXP-NNN.md) exists and is ready to run
---

# Executing Experiments

Part of the `hdd-superpowers` pack. This skill is deliberately "dumb": it runs the contract
it's given and reports what happened. It does not get to have opinions about the hypothesis.

**Announce at start:** "Using hdd-superpowers:executing-experiments to run [EXP-NNN]."

**Context:** Requires a frozen `EXP-NNN.md` with `state: PLANNED`.

<HARD-GATE>
You MUST NOT: change success thresholds, change metrics, silently change the
dataset, redefine the treatment, reinterpret the hypothesis, or declare
APPROVED/REFUTED. You MUST NOT write to the EXP-NNN.md contract file — every
observation goes into a new EVIDENCE-NNN.md file instead.

state: VALID requires every self-check item below to be true AND the
contract's Invalid If condition to not be met. If either fails, state:
INVALID — do not round up because the metrics look good anyway.
</HARD-GATE>

## Process

1. Read `EXP-NNN.md` in full. Do not read `HYP-NNN.md`'s Claim section looking for reasons to
   adjust the contract — the contract is what you execute, not a starting point for negotiation.
2. Implement the control and treatment exactly as specified.
3. Execute both. Record the environment: tool/library versions, dataset snapshot id or commit
   hash, timestamp.
4. Preserve raw outputs (logs, dumps, screenshots — whatever the experiment produces) under
   `docs/hdd/evidence/EVIDENCE-NNN/` (same `NNN` as the experiment).
5. Compute exactly the metrics named in `EXP-NNN.md`'s `Metrics` section — same key names, no
   additions, no substitutions.
6. Run the self-check (Artifact below) honestly. An unchecked or "probably fine" item is a
   failed item.
7. Determine `state`:
   - `INVALID` if any self-check item is false, or if the contract's `Invalid If` condition is
     met.
   - `VALID` otherwise.
8. Write `docs/hdd/evidence/EVIDENCE-NNN.md` (Artifact below). Do not write a verdict on the
   hypothesis anywhere in this file — only what was observed.
9. Sibling check (see Terminal State) — this skill owns the handoff, since this PoC has no
   `using-hdd` router.

## Artifact: EVIDENCE-NNN.md

```
---
id: EVIDENCE-031
experiment: EXP-031
state: VALID
---

## Environment
[tool/library versions, dataset snapshot id or commit hash, timestamp]

## Raw Evidence
- [relative path into EVIDENCE-031/ for each raw artifact]

## Computed Metrics
- [metric name from EXP-031.md]: [value]

## Observation Summary
[plain description of what was observed]

## Confounders
- [anything found during execution that could explain the result besides the treatment —
  empty list if none]

## Self Check
- [ ] contract_matched
- [ ] baseline_unchanged
- [ ] treatment_isolated_variable
- [ ] dataset_version_identifiable
- [ ] metric_calculation_reproducible
- [ ] raw_outputs_exist
- [ ] no_post_hoc_threshold_modification
- [ ] confounders_documented

<!-- invalid_reason: required if state: INVALID -->
```

## Checklist

- [ ] Every self-check item actually verified, not assumed
- [ ] `Computed Metrics` keys match `EXP-NNN.md`'s `Metrics` list exactly
- [ ] Raw outputs actually exist on disk under `EVIDENCE-NNN/`, not just described
- [ ] No edits made to `EXP-NNN.md`
- [ ] No `APPROVED`/`REFUTED` language anywhere in this file

## Terminal State (sibling check)

There is no `using-hdd` router in this pack — this skill owns the handoff:

1. After writing your own `EVIDENCE-NNN.md`, list every sibling `EXP-*.md` under the same
   `parent_hypothesis`.
2. If any sibling has no `EVIDENCE-*.md` yet, stop. Do not invoke anything — another
   `executing-experiments` run is still pending.
3. If any sibling's evidence is `INVALID` and no replacement experiment has been planned for it
   yet, invoke `planning-experiments` to create one.
4. If every sibling now has terminal evidence (`VALID`, or `INVALID` with a replacement already
   planned), set `HYP-NNN.state` to `EVIDENCE_READY` and invoke `deciding-hypotheses`.

If two `executing-experiments` runs finish at nearly the same moment, both may see themselves
as "last" — this is a known single-writer limitation of the PoC, not something to solve here.

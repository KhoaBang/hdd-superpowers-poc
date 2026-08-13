---
name: planning-experiments
description: Use when an approved hypothesis contract exists and needs experiment directions that could discriminate it from plausible alternatives
---

# Planning Experiments

Part of the `hdd-superpowers` pack — mirrors `superpowers:writing-plans`'s decomposition
discipline, but the deliverable is experiment contracts, not implementation tasks.

**Announce at start:** "Using hdd-superpowers:planning-experiments to design experiment directions for [HYP-NNN]."

**Context:** Requires an approved `HYP-NNN.md` with `state: FRAMED` (or a replacement-experiment
pass triggered by `executing-experiments` after an `INVALID` result).

<HARD-GATE>
Do NOT execute an experiment whose decision rule (Supports If / Refutes If /
Invalid If) was defined after observing its results.

Anti-pattern — First Plausible Experiment: do not stop after identifying the
first viable experiment direction. Before committing, consider whether
another independent direction could distinguish the hypothesis from a
plausible alternative explanation at materially lower cost or higher
information gain.
</HARD-GATE>

## Process

1. Read `HYP-NNN.md`. If `state` is not `FRAMED`, and this isn't a replacement pass requested by
   `executing-experiments`, stop — this hypothesis isn't ready to plan against.
2. Propose 2-N independent experiment directions. For each: what would it show, and is it truly
   independent of the others (does it fail or succeed for a different reason)? A single
   direction is acceptable only if you can state explicitly why no other direction would add
   discriminating power — write that reason down, don't skip the step silently.
3. For each direction, decide `selected` or `rejected` (one-line reason each). Rejected
   directions are not discarded — they go into the hypothesis file (step 6).
4. For each `selected` direction, write a frozen `docs/hdd/experiments/EXP-NNN.md` contract (see
   Artifact below). `NNN` = `max(existing NNN in docs/hdd/experiments/) + 1` per file, zero-padded
   to 3 digits.
5. If this is a replacement pass (an earlier `EXP-0XX` came back `INVALID`), add an
   HTML-comment `<!-- replaces: EXP-0XX -->` line directly under that new file's frontmatter.
6. Append a `## Experiment Directions Considered` section to `HYP-NNN.md` listing every
   direction from step 3, and update its `state` to `PLANNED`. Do not touch any other section of
   `HYP-NNN.md`.
7. Invoke `executing-experiments` once per newly-written `EXP-NNN.md`.

## Artifact: EXP-NNN.md (frozen contract)

```
---
id: EXP-031
parent_hypothesis: HYP-017
state: PLANNED
---
<!-- replaces: EXP-0XX   (only present on replacement experiments) -->

## Question
[what this experiment discriminates]

## Control
[the baseline condition]

## Treatment
[the single changed variable — if this bundles more than one change, split it into
separate experiments]

## Controlled Variables
- [held constant by design]

## Dataset
[concrete, identifiable reference: path + commit hash or version tag — not a description]

## Metrics
- [exact metric name 1]
- [exact metric name 2]

## Expected Observation
[what you expect to see if the hypothesis is true]

## Supports If
[condition — frozen before execution]

## Refutes If
[condition — frozen before execution]

## Invalid If
[condition under which the evidence should be thrown out regardless of the metric values —
frozen before execution]
```

## Artifact addition: HYP-NNN.md directions section

Appended to the bottom of the hypothesis file, after its frozen contract fields:

```
## Experiment Directions Considered
- **D1** [idea] — selected -> EXP-031
- **D2** [idea] — rejected: [reason]
- **D3** [idea] — selected -> EXP-032
```

## Checklist

- [ ] At least 2 directions considered, or an explicit written reason why fewer suffice
- [ ] Every selected direction's `Treatment` changes exactly one variable
- [ ] `Dataset` is a concrete reference, not a description
- [ ] `Supports If` / `Refutes If` / `Invalid If` are all filled in before any execution starts
- [ ] Metric names in each `EXP-NNN.md` are the exact strings `executing-experiments` must
      report back — no synonyms, no rewording later
- [ ] Rejected directions are recorded in `HYP-NNN.md`, not just selected ones
- [ ] `HYP-NNN.md`'s Claim/Rationale/Critical Assumptions/Scope/Supporting/Disconfirming
      sections are untouched

## Terminal State

Invoke `executing-experiments` once per new `EXP-NNN.md` (independent experiments may be
dispatched in parallel — see `superpowers:dispatching-parallel-agents`).

# HDD-Superpowers PoC Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Author, package, and deliver the 4-skill `hdd-superpowers` PoC pack (framing-hypotheses, planning-experiments, executing-experiments, deciding-hypotheses) as installable `.skill` files.

**Architecture:** Four standalone `SKILL.md` files under `hdd-superpowers/skills/<name>/`, each written per `superpowers:writing-skills` conventions and matching the spec's per-skill contracts exactly (trigger description, HARD-GATE, artifact schema, checklist, terminal state). Each folder is zipped into its own `.skill` file; all four are also bundled into one zip for one-shot install.

**Tech Stack:** Plain Markdown/YAML content authoring; `zip`/`unzip` for packaging (both present at `/usr/bin/zip`, `/usr/bin/unzip`).

**Spec:** `/home/claude/docs/superpowers/specs/2026-08-13-hdd-superpowers-design.md`

## Global Constraints

- Storage is filesystem-only: hypotheses/experiments/evidence/decisions live under `docs/hdd/{hypotheses,experiments,evidence,decisions}/` in whatever project the shipped skills run in (not this workspace — that path is written *into* the skill content, describing where a *future* user's HDD artifacts will live).
- Numbering: `next_id = max(existing NNN in the target directory) + 1`, zero-padded to 3 digits (never a count).
- Four artifact types only: Hypothesis (`HYP-NNN.md`), Experiment (`EXP-NNN.md`, frozen after `planning-experiments` writes it), Evidence (`EVIDENCE-NNN.md`, same `NNN` as its parent Experiment), Decision (`DECISION-NNN.md`, same `NNN` as its Hypothesis).
- This PoC ships exactly 4 skills. No `reviewing-evidence`, no `using-hdd` — both are deferred per spec §8. Do not add them.
- Every `SKILL.md` description starts with "Use when..." and contains no workflow summary (writing-skills SDO rule) — the description is a trigger, not a table of contents.
- Delivery format is a loose skill pack (no plugin manifest, no marketplace listing) — confirmed by user during brainstorming.
- No git repository exists in this workspace. Skip `git add`/`git commit` steps entirely — writing the file to disk is the persistence step for this plan.
- Full RED/GREEN pressure-scenario testing (the complete `superpowers:writing-skills` TDD process, with subagent baseline runs) is explicitly deferred until after the user's own field test across 5-10 real hypotheses (spec §8's validation criterion). Each task below instead runs a **static structural verification** (frontmatter present, HARD-GATE present, checklist present, terminal state present, description SDO-compliant, no placeholders) — not a subagent pressure test.

---

### Task 1: Scaffold pack directory and README

**Files:**
- Create: `/home/claude/hdd-superpowers/skills/framing-hypotheses/` (directory)
- Create: `/home/claude/hdd-superpowers/skills/planning-experiments/` (directory)
- Create: `/home/claude/hdd-superpowers/skills/executing-experiments/` (directory)
- Create: `/home/claude/hdd-superpowers/skills/deciding-hypotheses/` (directory)
- Create: `/home/claude/hdd-superpowers/README.md`

**Interfaces:**
- Produces: the four directory paths every later task writes its `SKILL.md` into.

- [ ] **Step 1: Create the directory tree**

```bash
mkdir -p /home/claude/hdd-superpowers/skills/framing-hypotheses
mkdir -p /home/claude/hdd-superpowers/skills/planning-experiments
mkdir -p /home/claude/hdd-superpowers/skills/executing-experiments
mkdir -p /home/claude/hdd-superpowers/skills/deciding-hypotheses
mkdir -p /home/claude/hdd-superpowers/dist
```

- [ ] **Step 2: Verify the tree**

Run: `find /home/claude/hdd-superpowers -maxdepth 2 -type d`
Expected: the 4 skill directories plus `dist/` listed, nothing else.

- [ ] **Step 3: Write the README**

Create `/home/claude/hdd-superpowers/README.md`:

```markdown
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
```

- [ ] **Step 4: Verify the README**

Run: `test -f /home/claude/hdd-superpowers/README.md && echo OK`
Expected: `OK`

---

### Task 2: Write `framing-hypotheses/SKILL.md`

**Files:**
- Create: `/home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md`

**Interfaces:**
- Consumes: nothing (entry point of the pipeline).
- Produces: skill name `framing-hypotheses`, invoked by name from Task 3's terminal state. Documents the `HYP-NNN.md` artifact schema that Tasks 3-5 all read.

- [ ] **Step 1: Write the file**

```markdown
---
name: framing-hypotheses
description: Use when the user presents an untested, falsifiable claim they want to validate or refute — a causal, performance, or product hypothesis — before any experiment design, implementation, or evidence collection begins
---

# Framing Hypotheses

Part of the `hdd-superpowers` pack — hypothesis/experiment/evidence lifecycle, mirroring
`superpowers:brainstorming`'s idea-to-approved-design shape for epistemic claims instead of
software designs.

## Overview

Turn an informal hypothesis into a falsifiable, user-approved hypothesis contract before any
experiment, implementation, or evidence collection happens. A hypothesis contract is a
`HYP-NNN.md` file — not a chat message, not a mental note.

**Announce at start:** "Using hdd-superpowers:framing-hypotheses to turn this into a falsifiable hypothesis contract."

<HARD-GATE>
Do NOT design experiments, write code, create implementation tasks, or collect
evidence until the hypothesis has: a falsifiable claim, rationale, critical
assumptions, explicit scope, a supporting observation, and a disconfirming
observation — and the user has approved this hypothesis contract.

A hypothesis is NOT ready because it sounds measurable. It is ready only when
the user and agent can state:
1. Why we believe it
2. What assumption it depends on
3. What observation would support it
4. What observation would kill it

If you cannot state 4, stop: the hypothesis is not yet testable.
</HARD-GATE>

## Process

1. Restate the informal claim back to the user in one sentence. Get confirmation you understood
   it before going further.
2. Ask, one at a time, until each of these has a real answer — not a guess you fill in on the
   user's behalf:
   - **Claim** — the falsifiable statement itself
   - **Rationale** — why we believe it (prior evidence, intuition, a pattern already noticed)
   - **Critical Assumptions** — what has to be true for the claim to even make sense
   - **Scope** — what this claim does and does not cover
   - **Supporting Observation** — the observation pattern that would confirm it
   - **Disconfirming Observation** — the observation pattern that would kill it
3. Write `docs/hdd/hypotheses/HYP-NNN.md` (see Artifact below). `NNN` = `max(existing NNN in
   docs/hdd/hypotheses/) + 1`, zero-padded to 3 digits; `001` if the directory doesn't exist yet.
4. Present the filled contract to the user verbatim and ask for explicit approval. Do not
   proceed on silence or a vague "sounds good" aimed at the conversation — the approval is of
   the written file.
5. On approval, set `state: FRAMED` and invoke `planning-experiments`.

## Artifact: HYP-NNN.md

```
---
id: HYP-017
state: FRAMED
created: 2026-08-13
---

## Claim
[the falsifiable statement]

## Rationale
[why we believe it]

## Critical Assumptions
- [assumption 1]
- [assumption 2]

## Scope
[what this claim covers and doesn't]

## Supporting Observation
[what observation would confirm the claim]

## Disconfirming Observation
[what observation would kill the claim]
```

These six fields are frozen once approved. Nothing after this skill edits `Claim` through
`Disconfirming Observation` — `planning-experiments` may only update `state` and append a new
section below them.

## Checklist

- [ ] Claim is a single falsifiable statement, not a goal or a question
- [ ] Rationale references something concrete (data, a prior run, an observed pattern) — not
      "it seems likely"
- [ ] At least one critical assumption is named
- [ ] Scope excludes at least one thing explicitly (proves scope was actually considered)
- [ ] Supporting and disconfirming observations are both observable facts, not judgment calls
- [ ] User has approved the written file, not just the idea as discussed in chat

## Anti-Pattern: Measurable-Sounding, Not Falsifiable

"Latency will improve" sounds measurable but has no disconfirming observation attached —
improve by how much, measured how, compared to what baseline? Don't accept a claim until the
disconfirming observation is as concrete as the supporting one.

## Terminal State

Invoke `planning-experiments`.
```

- [ ] **Step 2: Static verification**

Run: `test -f /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md && grep -c "^description: Use when" /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md && grep -c "HARD-GATE" /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md && grep -c "Terminal State" /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md`

Expected: file exists, exactly 1 match for `description: Use when`, at least 2 matches for
`HARD-GATE` (open+close tag), exactly 1 match for `Terminal State`.

- [ ] **Step 3: Placeholder scan**

Run: `grep -niE "TBD|TODO|fill in|placeholder" /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md`

Expected: no output (grep exits 1 / prints nothing). If it matches, the file is unfinished —
fix before moving on.

---

### Task 3: Write `planning-experiments/SKILL.md`

**Files:**
- Create: `/home/claude/hdd-superpowers/skills/planning-experiments/SKILL.md`

**Interfaces:**
- Consumes: `HYP-NNN.md` schema from Task 2 (must reference the exact same section names: `Claim`, `Rationale`, `Critical Assumptions`, `Scope`, `Supporting Observation`, `Disconfirming Observation`).
- Produces: skill name `planning-experiments`, invoked by name from Task 2's and Task 4's terminal states. Documents the `EXP-NNN.md` artifact schema that Task 4 reads and Task 5 references.

- [ ] **Step 1: Write the file**

```markdown
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
```

- [ ] **Step 2: Static verification**

Run: `test -f /home/claude/hdd-superpowers/skills/planning-experiments/SKILL.md && grep -c "^description: Use when" /home/claude/hdd-superpowers/skills/planning-experiments/SKILL.md && grep -c "HARD-GATE" /home/claude/hdd-superpowers/skills/planning-experiments/SKILL.md && grep -c "Terminal State" /home/claude/hdd-superpowers/skills/planning-experiments/SKILL.md`

Expected: file exists, 1 match for the description line, at least 2 for `HARD-GATE`, 1 for
`Terminal State`.

- [ ] **Step 3: Placeholder scan**

Run: `grep -niE "TBD|TODO|fill in|placeholder" /home/claude/hdd-superpowers/skills/planning-experiments/SKILL.md`

Expected: no output.

- [ ] **Step 4: Cross-check field names against Task 2**

Run: `grep -o "^## .*" /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md | grep -E "Claim|Rationale|Critical Assumptions|Scope|Supporting Observation|Disconfirming Observation"`

Expected: 6 lines, one per section name. Confirm `planning-experiments/SKILL.md`'s "Artifact
addition" and Process section 6 reference these same 6 names (visual check — the grep above is
just to have the exact strings in front of you while checking).

---

### Task 4: Write `executing-experiments/SKILL.md`

**Files:**
- Create: `/home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md`

**Interfaces:**
- Consumes: `EXP-NNN.md` schema from Task 3 (`Question`, `Control`, `Treatment`, `Controlled Variables`, `Dataset`, `Metrics`, `Expected Observation`, `Supports If`, `Refutes If`, `Invalid If`).
- Produces: skill name `executing-experiments`, invoked by name from Task 3's terminal state and self-invoked (sibling check) from within its own terminal state. Documents the `EVIDENCE-NNN.md` schema that Task 5 reads. Also invokes `planning-experiments` (replacement pass) and `deciding-hypotheses` by name — must match Task 3's and Task 5's `name:` frontmatter exactly.

- [ ] **Step 1: Write the file**

```markdown
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
```

- [ ] **Step 2: Static verification**

Run: `test -f /home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md && grep -c "^description: Use when" /home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md && grep -c "HARD-GATE" /home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md && grep -c "Terminal State" /home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md`

Expected: file exists, 1 match for the description line, at least 2 for `HARD-GATE`, 1 for
`Terminal State`.

- [ ] **Step 3: Placeholder scan**

Run: `grep -niE "TBD|TODO|fill in|placeholder" /home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md`

Expected: no output.

- [ ] **Step 4: Cross-check invoked skill names**

Run: `grep -oE "invoke \`[a-z-]+\`" /home/claude/hdd-superpowers/skills/executing-experiments/SKILL.md`

Expected: matches are exactly `` invoke `planning-experiments` `` and `` invoke
`deciding-hypotheses` `` — both must match the `name:` frontmatter value written in Task 3 and
Task 5 exactly (no typos, no case mismatch).

---

### Task 5: Write `deciding-hypotheses/SKILL.md`

**Files:**
- Create: `/home/claude/hdd-superpowers/skills/deciding-hypotheses/SKILL.md`

**Interfaces:**
- Consumes: `HYP-NNN.md` schema from Task 2, `EVIDENCE-NNN.md` schema from Task 4.
- Produces: skill name `deciding-hypotheses`, invoked by name from Task 4's terminal state. Documents the `DECISION-NNN.md` artifact schema (final artifact type, nothing downstream reads it programmatically).

- [ ] **Step 1: Write the file**

```markdown
---
name: deciding-hypotheses
description: Use when a hypothesis has terminal evidence for every planned experiment and needs a final outcome
---

# Deciding Hypotheses

Part of the `hdd-superpowers` pack — the reducer. It does no new research; it applies the
decision policy the hypothesis already committed to before any evidence existed.

**Announce at start:** "Using hdd-superpowers:deciding-hypotheses to reduce [HYP-NNN] to a final outcome."

**Context:** Requires `HYP-NNN.state: EVIDENCE_READY`.

<HARD-GATE>
Do NOT weaken, reinterpret, or replace the hypothesis after seeing evidence.
If the evidence matches Disconfirming Observation, the result is REFUTED —
regardless of how close it came to Supporting Observation.

A revised claim is a NEW hypothesis. Create it via framing-hypotheses. Never
edit a REFUTED HYP-NNN.md to rescue it.
</HARD-GATE>

## Process

1. Read `HYP-NNN.md` in full, including `Supporting Observation`, `Disconfirming Observation`,
   and `Critical Assumptions`.
2. Collect every `EVIDENCE-*.md` for this hypothesis's experiments. Separate `VALID` from
   `INVALID` — `INVALID` evidence is listed but never weighed in the outcome.
3. For each `VALID` piece of evidence, check it against `Supporting Observation` and
   `Disconfirming Observation`. For each `Critical Assumption`, state explicitly whether the
   evidence shows it held or was violated — an assumption quietly violated invalidates the
   reasoning even if the headline metric looks good.
4. Reduce to one outcome: `APPROVED` if the valid evidence matches `Supporting Observation` and
   no critical assumption was violated; `REFUTED` otherwise — including when the evidence is
   simply inconclusive relative to what was pre-registered. There is no third option.
5. Write `docs/hdd/decisions/DECISION-NNN.md` (Artifact below), same `NNN` as `HYP-NNN`.
6. Set `HYP-NNN.state` to the same outcome.

## Artifact: DECISION-NNN.md

```
---
id: DECISION-017
hypothesis: HYP-017
outcome: REFUTED
---

## Evidence Considered
- EVIDENCE-031
- EVIDENCE-032

## Excluded Evidence
- [EVIDENCE-NNN ids marked INVALID, if any]

## Reasoning
[map each piece of considered evidence against Supporting Observation / Disconfirming
Observation; state explicitly whether each Critical Assumption held or was violated]

## Follow Up
[if REFUTED produced a new insight, name the new HYP-NNN this becomes — created via a
fresh framing-hypotheses run, never by editing this hypothesis]
```

## Checklist

- [ ] Every `VALID` evidence file is accounted for in `Evidence Considered`
- [ ] Every `INVALID` evidence file is accounted for in `Excluded Evidence`, not silently dropped
- [ ] `Reasoning` addresses every `Critical Assumption` from `HYP-NNN.md`, not just the headline
      metric
- [ ] Outcome is exactly `APPROVED` or `REFUTED` — no hedged outcome
- [ ] `HYP-NNN.md`'s frozen contract fields are untouched; only `state` changed

## Anti-Pattern: The Technically-Not-Wrong Rescue

"The result wasn't what we expected, but if we interpret the treatment as..." — this is
reinterpreting the hypothesis after seeing evidence. Stop. If evidence matches the disconfirming
observation, the outcome is `REFUTED`. Any new interpretation is a new hypothesis, framed fresh.

## Terminal State

None. This ends the pipeline for this hypothesis.
```

- [ ] **Step 2: Static verification**

Run: `test -f /home/claude/hdd-superpowers/skills/deciding-hypotheses/SKILL.md && grep -c "^description: Use when" /home/claude/hdd-superpowers/skills/deciding-hypotheses/SKILL.md && grep -c "HARD-GATE" /home/claude/hdd-superpowers/skills/deciding-hypotheses/SKILL.md`

Expected: file exists, 1 match for the description line, at least 2 for `HARD-GATE`.

- [ ] **Step 3: Placeholder scan**

Run: `grep -niE "TBD|TODO|fill in|placeholder" /home/claude/hdd-superpowers/skills/deciding-hypotheses/SKILL.md`

Expected: no output.

---

### Task 6: Cross-skill self-review

**Files:**
- Modify (if issues found): any of the 4 `SKILL.md` files from Tasks 2-5.

**Interfaces:**
- Consumes: all 4 completed `SKILL.md` files.
- Produces: nothing new — this is a verification-only task; fix inline if it finds problems.

- [ ] **Step 1: Spec coverage check**

Open `/home/claude/docs/superpowers/specs/2026-08-13-hdd-superpowers-design.md` side by side
with the 4 `SKILL.md` files. For each of spec §5.1-5.4 (one per skill), confirm the written
`SKILL.md` contains: the exact trigger description, the exact HARD-GATE text, the exact artifact
schema fields, and the exact terminal-state routing. List any gap; if found, fix the `SKILL.md`
inline (the spec is the source of truth — the skill must match it, not the other way around).

- [ ] **Step 2: Terminal-state routing table**

Run this to extract every skill name referenced as an invocation target across all 4 files:

```bash
grep -rhoE "invoke \`[a-z-]+\`" /home/claude/hdd-superpowers/skills/*/SKILL.md | sort -u
```

Expected output (exactly these 4 lines, nothing else):
```
invoke `deciding-hypotheses`
invoke `executing-experiments`
invoke `planning-experiments`
```

Then run:

```bash
grep -h "^name:" /home/claude/hdd-superpowers/skills/*/SKILL.md | sort -u
```

Expected output (exactly these 4 lines):
```
name: deciding-hypotheses
name: executing-experiments
name: framing-hypotheses
name: planning-experiments
```

Every invocation target from the first command must appear as a `name:` value in the second.
(`framing-hypotheses` legitimately has zero inbound `invoke` references from other skills in
this pack — it's the user-triggered entry point — confirm that's the only name missing from the
first list, not a typo elsewhere.)

- [ ] **Step 3: Artifact field-name consistency**

Run:

```bash
grep -h "^## " /home/claude/hdd-superpowers/skills/framing-hypotheses/SKILL.md
```

Then confirm every one of those section names is spelled identically (case, spacing) everywhere
else it's referenced — in `planning-experiments/SKILL.md`'s Process step 6 and "Artifact
addition" section, and in `deciding-hypotheses/SKILL.md`'s Process steps 1 and 3. Fix any
mismatch inline.

- [ ] **Step 4: Word-count sanity check**

Run: `wc -w /home/claude/hdd-superpowers/skills/*/SKILL.md`

Expected: each file in the 500-1200 word range. These are technique/discipline skills invoked
mid-task (not `getting-started`-style skills loaded into every conversation), so the tighter
`<200 word` SDO target doesn't apply — but a file over ~1500 words likely has repeated content
that should be trimmed or cross-referenced instead.

---

### Task 7: Package into `.skill` files and a bundle

**Files:**
- Create: `/home/claude/hdd-superpowers/dist/framing-hypotheses.skill`
- Create: `/home/claude/hdd-superpowers/dist/planning-experiments.skill`
- Create: `/home/claude/hdd-superpowers/dist/executing-experiments.skill`
- Create: `/home/claude/hdd-superpowers/dist/deciding-hypotheses.skill`
- Create: `/home/claude/hdd-superpowers/dist/hdd-superpowers-bundle.zip`

**Interfaces:**
- Consumes: the 4 verified `SKILL.md` files from Tasks 2-6.
- Produces: 5 zip archives ready for `SendUserFile` in Task 8.

- [ ] **Step 1: Package each skill as a standalone `.skill` file**

Each `.skill` archive contains just that skill's `SKILL.md` at the archive root (no wrapping
folder), so it can be dropped in as `~/.claude/skills/<name>/SKILL.md` on extraction into a
folder named after the skill:

```bash
cd /home/claude/hdd-superpowers/skills/framing-hypotheses && zip -j /home/claude/hdd-superpowers/dist/framing-hypotheses.skill SKILL.md
cd /home/claude/hdd-superpowers/skills/planning-experiments && zip -j /home/claude/hdd-superpowers/dist/planning-experiments.skill SKILL.md
cd /home/claude/hdd-superpowers/skills/executing-experiments && zip -j /home/claude/hdd-superpowers/dist/executing-experiments.skill SKILL.md
cd /home/claude/hdd-superpowers/skills/deciding-hypotheses && zip -j /home/claude/hdd-superpowers/dist/deciding-hypotheses.skill SKILL.md
```

- [ ] **Step 2: Verify each `.skill` archive**

Run: `for f in /home/claude/hdd-superpowers/dist/*.skill; do echo "== $f =="; unzip -l "$f"; done`

Expected: each archive lists exactly one entry, `SKILL.md`, non-zero size.

- [ ] **Step 3: Package the full bundle (folder structure preserved, for manual multi-skill extraction)**

```bash
cd /home/claude/hdd-superpowers && zip -r dist/hdd-superpowers-bundle.zip README.md skills/
```

- [ ] **Step 4: Verify the bundle**

Run: `unzip -l /home/claude/hdd-superpowers/dist/hdd-superpowers-bundle.zip`

Expected: `README.md` plus `skills/<name>/SKILL.md` for all 4 skill names, 5 entries total (plus
directory entries, which is fine).

---

### Task 8: Deliver to the user

**Files:** none created — delivery only.

- [ ] **Step 1: Send the bundle and the 4 standalone `.skill` files**

Use `SendUserFile` with all 5 paths from `/home/claude/hdd-superpowers/dist/`, status `normal`,
a one-line caption noting the bundle is for extracting all 4 at once and the individual
`.skill` files are for installing one at a time.

- [ ] **Step 2: Point back to the spec for context**

If the spec file from the brainstorming phase wasn't already sent to the user in this session,
send `/home/claude/docs/superpowers/specs/2026-08-13-hdd-superpowers-design.md` alongside it.
(It was already sent earlier in this conversation — skip if so.)

---

## Self-Review (run once, after all tasks drafted)

**Spec coverage:** §2 (invariants) → surfaced as HARD-GATE text and checklist items across all 4
skills, not a separate artifact — correct, the spec never asked for a standalone invariants
doc inside the pack. §3 (lifecycle) → Task 2-5 terminal states form the exact loop. §4 (artifact
model) → all 4 schemas reproduced verbatim in Tasks 2-5, including the `max(NNN)+1` numbering
rule and the confounders/self_check split fixed during spec self-review. §5 (skills) → one task
per skill, gates and checklists copied 1:1. §6 (coordination table) → reproduced as the sibling-
check procedure in Task 4's Terminal State. §7 (delivery) → Task 7-8. §8 (limitations) → stated
in the README (Task 1) and echoed in Global Constraints above, so nothing here contradicts the
deferred scope. §9 (next step) → this plan is that next step.

**Placeholder scan:** none found — every task step has literal commands or literal file content,
not descriptions of content.

**Type/name consistency:** verified while drafting — `name:` frontmatter values
(`framing-hypotheses`, `planning-experiments`, `executing-experiments`, `deciding-hypotheses`)
match every `invoke` reference across all 4 files; artifact section headers (`Claim` through
`Disconfirming Observation` on the hypothesis, `Question` through `Invalid If` on the experiment,
`Environment` through `Self Check` on the evidence, `Evidence Considered` through `Follow Up` on
the decision) are spelled identically everywhere they're referenced. Task 6 re-verifies this
mechanically rather than trusting this note alone.

## Execution Handoff

**Plan complete and saved to `docs/superpowers/plans/2026-08-13-hdd-superpowers-poc.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**

# HDD-Superpowers Design Spec

**Status:** Draft — pending user review
**Date:** 2026-08-13
**Scope:** PoC (4 skills), filesystem-only storage, delivered as a loose skill pack

## 1. Goal

Mirror the architecture and prose style of the `superpowers` skill pack, but replace the
design/implementation lifecycle with a hypothesis/experiment/evidence lifecycle. The target
property to validate over 5-10 real hypotheses:

> Any `APPROVED`/`REFUTED` outcome must be traceable back through
> `Decision → Evidence → Experiment contract → Hypothesis` without reading chat history.

If that property holds up in practice, HDD-superpowers graduates from PoC to a maintained pack.
If self-review inside `executing-experiments` turns out untrustworthy, `reviewing-evidence`
gets split out as a fifth skill (deferred, not designed here — see §8).

## 2. Theoretical grounding

Superpowers supplies the skill-packaging mechanics (HARD-GATE, checklist, anti-pattern,
explicit next-skill, artifact convention). The epistemic rules come from five sources, reduced
to seven invariants:

| # | Invariant | Source |
|---|---|---|
| 1 | Falsifiability — every hypothesis must have an observation that can kill it | MIT CISR, "Learn from Hypotheses, Not Failures" (2019) |
| 2 | Pre-registration — decision rules are frozen before results are observed | Hypothesis-Driven Skill Optimization (HDSO), arXiv:2606.22330 |
| 3 | Control — treatment must isolate the claimed causal change | HDSO, arXiv:2606.22330 |
| 4 | Test branching — one hypothesis may spawn multiple independent experiments; don't commit to the first plausible one | SkillHEX, arXiv:2608.05628 |
| 5 | Evidence provenance — every conclusion traces back to inspectable evidence, as first-class objects, not chat logs | Hypothesis Evolution Protocol (HEP), arXiv:2607.09195 |
| 6 | Persistent decision history — rejected directions and invalid experiments are kept, not just successes | SkillHone, arXiv:2606.08671 |
| 7 | No post-hoc rescue — a refuted hypothesis stays refuted; a revised claim is a new hypothesis | Synthesis (all sources) |

Every hard-gate and field below traces to one of these seven. Bayesian belief scores, PUCT
search, automatic skill evolution, and confidence estimation from the source papers are
explicitly **out of scope** for the PoC — they're optimization machinery for a problem this
pack hasn't proven it has yet.

## 3. Lifecycle overview

```
framing-hypotheses
        ↓
planning-experiments
        ↓
┌───────────────┬───────────────┬───────────────┐
│               │               │
execute EXP-A   execute EXP-B   execute EXP-C     (independent → dispatchable in parallel)
│               │               │
EVIDENCE-A      EVIDENCE-B      EVIDENCE-C
└───────────────┴───────┬───────┴───────────────┘
                         ↓
              (last-to-finish checks: all siblings terminal?)
                         ↓
              deciding-hypotheses
                         ↓
              DECISION → APPROVED | REFUTED
```

If an experiment's evidence comes back `INVALID`, `planning-experiments` is invoked again to
create a **replacement** experiment (new ID, `replaces:` pointer to the invalid one). The
invalid experiment and its evidence are never deleted or overwritten — invariant 6.

`INVALID` is a property of *evidence*, never a terminal state of a *hypothesis*. A hypothesis
converges only to `APPROVED` or `REFUTED`.

## 4. Artifact model

Four first-class artifact types (invariant 5 — HEP). Each is a Markdown file with a strict YAML
frontmatter block (`id`, `state`/`outcome`, and cross-references) followed by structured
Markdown body sections. Frontmatter is machine-parseable; body sections are for the prose a
human or agent needs to read.

```
docs/hdd/
├── hypotheses/
│   └── HYP-017.md
├── experiments/
│   ├── EXP-031.md
│   └── EXP-032.md
├── evidence/
│   ├── EVIDENCE-031.md
│   ├── EVIDENCE-031/          ← raw artifacts (logs, dumps) referenced by EVIDENCE-031.md
│   └── EVIDENCE-032.md
└── decisions/
    └── DECISION-017.md
```

**Numbering:** `next_id = max(existing NNN in that directory) + 1`, zero-padded to 3 digits.
(Not a count — a count breaks if a file is ever removed.) `EVIDENCE-NNN` always reuses its
parent `EXP-NNN`'s number (1:1). `DECISION-NNN` always reuses its hypothesis's `HYP-NNN` number
(1:1 — one decision closes one hypothesis; a revised claim is a new `HYP-NNN`, invariant 7).

**Single-writer caveat:** this numbering and the state-transition rules in §6 assume one agent
(or sequentially-coordinated agents) acting on a given hypothesis at a time. Two agents framing
a hypothesis in the same second, or two `executing-experiments` runs finishing simultaneously,
can race. Acceptable for a supervised PoC; flagged as a v2 concern, not solved here.

### 4.1 HYP-NNN.md (written by `framing-hypotheses`; frozen after approval)

```yaml
id: HYP-017
state: PROPOSED | FRAMED | PLANNED | TESTING | EVIDENCE_READY | APPROVED | REFUTED
created: <date>
```

Body, all required, no placeholders:
- **claim** — the falsifiable statement
- **rationale** — why we believe it (prior evidence, intuition, invariant 1)
- **critical_assumptions** — list; what must hold true for the claim to even make sense
- **scope** — explicit boundary of what this claim does and doesn't cover
- **supporting_observation** — the observation pattern that would confirm the claim
- **disconfirming_observation** — the observation pattern that would kill it (invariant 1);
  this doubles as the decision policy `deciding-hypotheses` consumes

Only `planning-experiments` may touch this file after framing, and only to (a) update `state`,
and (b) append a `## Experiment Directions Considered` section (§5.2). The six contract fields
above are never edited again.

### 4.2 EXP-NNN.md (written once by `planning-experiments`; immutable contract)

```yaml
id: EXP-031
parent_hypothesis: HYP-017
state: PLANNED
replaces: EXP-0XX   # only present on replacement experiments
```

Body:
- **question** — what this experiment discriminates
- **control** / **treatment** — treatment isolates exactly the variable the claim is about
  (invariant 3); if a proposed treatment bundles more than one changed variable, it's not a
  valid experiment for this hypothesis — split it
- **controlled_variables** — held constant by design
- **dataset** — must be a concrete, identifiable reference (path + commit hash or version tag,
  not a description) — feeds `self_check.dataset_version_identifiable`
- **metrics** — exact metric names; `EVIDENCE-NNN.computed_metrics` must use these same keys,
  no others
- **expected_observation**
- **supports_if / refutes_if / invalid_if** — the decision rule, frozen before execution
  (invariant 2)

`executing-experiments` never writes to this file. Physical file separation is the enforcement
mechanism for "don't change the threshold after seeing results" — there's no file for it to
edit even if tempted.

### 4.3 EVIDENCE-NNN.md (written once by `executing-experiments`)

```yaml
id: EVIDENCE-031
experiment: EXP-031
state: VALID | INVALID
```

Body:
- **environment** — versions, dataset snapshot id, commit hash, timestamp
- **raw_evidence** — relative paths into the sibling `EVIDENCE-031/` directory
- **computed_metrics** — keys must exactly match `EXP-031.metrics`
- **observation_summary** — plain description of what was observed
- **confounders** — list of confounders found during execution; empty list if none
- **self_check** — every item boolean, all required:
  - `contract_matched`
  - `baseline_unchanged`
  - `treatment_isolated_variable`
  - `dataset_version_identifiable`
  - `metric_calculation_reproducible`
  - `raw_outputs_exist`
  - `no_post_hoc_threshold_modification`
  - `confounders_documented` — true only if `confounders` above was actually filled in
    (even as an empty list) rather than skipped
- **invalid_reason** — required if `state: INVALID`

**Rule:** `state: VALID` requires every `self_check` item to be `true` AND the experiment's own
`invalid_if` condition (from EXP-NNN.md) to not be met. Either failure forces `state: INVALID`.
This is the self-review the executor performs; see §8 for why it's deliberately not delegated
to a separate reviewer yet.

### 4.4 DECISION-NNN.md (written once by `deciding-hypotheses`; closes the hypothesis)

```yaml
id: DECISION-017
hypothesis: HYP-017
outcome: APPROVED | REFUTED
```

Body:
- **evidence_considered** — list of `EVIDENCE-NNN` ids used
- **excluded_evidence** — `EVIDENCE-NNN` ids marked `INVALID`, excluded from the decision
- **reasoning** — maps `evidence_considered` against `HYP-017.supporting_observation` /
  `disconfirming_observation`; must explicitly state whether each `critical_assumption` held or
  was violated (surfaces silent confounds instead of burying them)
- **follow_up** — if `REFUTED` produced a new insight, name the new `HYP-NNN` this becomes
  (created via a fresh `framing-hypotheses` run — never by editing `HYP-017`)

After writing this file, `deciding-hypotheses` sets `HYP-017.state` to the same `outcome`. The
decision file is the audit record; the hypothesis file's `state` is just a routing pointer.

## 5. Skills

Each skill: announce at start ("Using hdd-superpowers:<skill> to <purpose>"), one clear
responsibility, one HARD-GATE, one terminal state pointing to the next skill (or none).

### 5.1 `framing-hypotheses`

**Trigger (SKILL.md description):** "Use when the user presents an untested, falsifiable
claim they want to validate or refute — a causal, performance, or product hypothesis — before
any experiment design, implementation, or evidence collection begins."

**HARD-GATE:**
```
Do NOT design experiments, write code, create implementation tasks, or collect
evidence until the hypothesis has: a falsifiable claim, rationale, critical
assumptions, explicit scope, a supporting observation, and a disconfirming
observation — and the user has approved this hypothesis contract.

A hypothesis is NOT ready because it sounds measurable. It is ready only when
the user and agent can state: (1) why we believe it, (2) what assumption it
depends on, (3) what observation would support it, (4) what observation would
kill it. If you cannot state (4), stop: the hypothesis is not yet testable.
```

**Output:** `HYP-NNN.md`, `state: FRAMED`, user-approved.
**Terminal:** invoke `planning-experiments`.

### 5.2 `planning-experiments`

**Trigger:** "Use when an approved hypothesis contract exists and needs experiment directions
that could discriminate it from plausible alternatives."

**HARD-GATE:**
```
Do NOT execute an experiment whose decision rule (supports_if/refutes_if/
invalid_if) was defined after observing its results.

Anti-pattern — First Plausible Experiment: do not stop after identifying the
first viable experiment direction. Before committing, consider whether another
independent direction could distinguish the hypothesis from a plausible
alternative explanation at materially lower cost or higher information gain.
```

**Process:** propose 2-N experiment directions; for each, decide `selected` or `rejected` (with
reason). Selected directions each get a frozen `EXP-NNN.md`. Append the full directions list
(selected and rejected) to `HYP-NNN.md` under `## Experiment Directions Considered` — rejected
directions are kept, not discarded (invariant 6).

**Output:** one or more `EXP-NNN.md`; `HYP-NNN.state → PLANNED`.
**Terminal:** invoke `executing-experiments` once per `EXP-NNN` (independent → eligible for
`superpowers:dispatching-parallel-agents`).

### 5.3 `executing-experiments`

**Trigger:** "Use when a frozen experiment contract (EXP-NNN.md) exists and is ready to run."

**HARD-GATE:**
```
You MUST NOT: change success thresholds, change metrics, silently change the
dataset, redefine the treatment, reinterpret the hypothesis, or declare
APPROVED/REFUTED. You MUST NOT write to the EXP-NNN.md contract file.

state: VALID requires every self_check item to be true AND the contract's
invalid_if condition to not be met. If either fails, state: INVALID — do not
round up.
```

**Process:** read `EXP-NNN.md` → implement → execute → record environment → preserve raw
evidence under `evidence/EVIDENCE-NNN/` → compute exactly the declared metrics → run the
`self_check` → write `EVIDENCE-NNN.md`.

**Terminal (sibling check — this skill owns the handoff, since there is no `using-hdd`
router in the PoC):** after writing its own `EVIDENCE-NNN.md`, check every sibling `EXP-*.md`
under the same `parent_hypothesis`. If any sibling has no `EVIDENCE-*.md` yet, or has one with
`state: INVALID` and no replacement experiment planned, stop — do not invoke anything. If every
sibling has terminal evidence (all `VALID`, or `INVALID` with a replacement already planned),
set `HYP-NNN.state → EVIDENCE_READY` and invoke `deciding-hypotheses`. If any sibling is
`INVALID` with no replacement yet, invoke `planning-experiments` instead to create one.

(This hand-rolled coordination is the single-writer caveat from §4 in practice — acceptable for
a supervised PoC.)

### 5.4 `deciding-hypotheses`

**Trigger:** "Use when a hypothesis has terminal evidence for every planned experiment and
needs a final outcome."

**HARD-GATE:**
```
Do NOT weaken, reinterpret, or replace the hypothesis after seeing evidence.
If the evidence contradicts disconfirming_observation, the result is REFUTED.
A revised claim is a NEW hypothesis — create it via framing-hypotheses, never
by editing the refuted HYP-NNN.
```

**Process:** consume `HYP-NNN.md` + every `EVIDENCE-*.md` with `state: VALID` for this
hypothesis (excluding `INVALID` ones, listed but not weighed) → reduce against
`supporting_observation`/`disconfirming_observation` → write `DECISION-NNN.md` → set
`HYP-NNN.state` to the same outcome.

**Terminal:** none. This ends the pipeline for this hypothesis.

## 6. Coordination summary

| Field | Written by | When |
|---|---|---|
| `HYP.state = FRAMED` | framing-hypotheses | on user approval |
| `HYP.state = PLANNED` | planning-experiments | after writing EXP files |
| `HYP.state = EVIDENCE_READY` | executing-experiments | sibling check passes (§5.3) |
| `HYP.state = APPROVED/REFUTED` | deciding-hypotheses | after writing DECISION file |
| `EXP-NNN.md` (entire file) | planning-experiments | once, immutable after |
| `EVIDENCE-NNN.md` (entire file) | executing-experiments | once, immutable after |
| `DECISION-NNN.md` (entire file) | deciding-hypotheses | once, immutable after |

No skill other than the one listed ever writes that field/file. Where a skill needs to react to
another skill's output (executing-experiments deciding whether to invoke deciding-hypotheses vs.
planning-experiments), the rule is spelled out in that skill's own terminal-state section (§5.3)
rather than delegated to a shared router — this is the deliberate `using-hdd`-less design (§8).

## 7. Delivery format

Loose skill pack — four independent skill folders, each a standalone `.skill` (zip):

```
hdd-superpowers/
├── framing-hypotheses/SKILL.md
├── planning-experiments/SKILL.md
├── executing-experiments/SKILL.md
└── deciding-hypotheses/SKILL.md
```

Delivered as one bundle zip containing all four folders, to be extracted into the user's
personal skills directory (`~/.claude/skills/`). No plugin manifest, no marketplace listing —
matches the "skill pack rời" decision. Discovery relies on the existing `using-superpowers`
mechanism already installed (its "if a skill applies, you must use it" rule is generic, not
Superpowers-specific) — this is an assumption to sanity-check at install time, not something
this pack can control.

## 8. Known limitations / explicitly deferred

- **`reviewing-evidence` (deferred):** the PoC deliberately keeps evidence self-review inside
  `executing-experiments` (§4.3, §5.3) rather than splitting out a separate reviewer. This is
  the thing the PoC is meant to test — the self_check checklist is the strongest version of
  self-review this design has without a second skill. If real hypotheses show the executor
  rubber-stamping its own evidence, split `reviewing-evidence` out as a fifth skill that
  consumes `EVIDENCE-NNN.md` and can flip `state: VALID → INVALID` on review — nothing in the
  artifact model needs to change for that split, only a skill gets inserted between
  `executing-experiments` and the sibling-check.
- **`using-hdd` (deferred):** no meta-skill for lifecycle routing. Entry-point routing relies on
  each skill's own description; mid-pipeline routing is hand-coded into `executing-experiments`'
  terminal step (§5.3). Revisit if the sibling-check logic proves too easy to get wrong in
  practice.
- **Concurrency:** numbering and state-field writes assume a single active writer per
  hypothesis (§4). Not safe for uncoordinated parallel agents mutating the same HYP.
- **Storage adapter:** filesystem only. The GitHub Issues convention from the original proposal
  (parent issue = hypothesis, sub-issues = experiments, PR = treatment, comments = evidence
  review) is a plausible v2 adapter but isn't implemented or documented as usable in v1 — only
  filesystem paths appear in the skills.
- **Optimization machinery intentionally excluded:** Bayesian belief scores, PUCT tree search,
  automatic skill evolution, dynamic evidence matrices (all present in the source papers) are
  not part of this pack. Revisit only if the plain four-skill loop proves the core workflow
  valuable across 5-10 real hypotheses.

## 9. Next step

This spec is the input to `superpowers:writing-plans`, which will produce an implementation
plan for authoring the four `SKILL.md` files (using `superpowers:writing-skills` conventions —
SDO-compliant descriptions, checklists, flowcharts only where a decision is non-obvious),
packaging them, and delivering the bundle.

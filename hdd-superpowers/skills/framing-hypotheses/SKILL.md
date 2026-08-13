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

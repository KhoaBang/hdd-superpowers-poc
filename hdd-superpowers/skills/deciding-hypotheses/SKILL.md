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

# Progression

**Status:** Draft · **Prefix:** PROG

## Purpose

How a slot entry's prescription (sets, weight, reps, reps in reserve, rest) is set for iteration 1 and changes in later iterations, optionally only after the user hit the previous target.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| PROG-1 | Fields: `sets`, `max_sets`, `weight`, `max_weight`, `repetitions`, `max_repetitions`, `rir`, `max_rir`, `rest` (seconds), `max_rest`. A `max_*` field turns its base field into a range. | weight 60, max_weight 70 → shown 60–70 |
| PROG-2 | Rule per field: `{slot_entry, field, iteration ≥ 1, value (0–3000, 2 decimals), operation: add\|subtract\|replace (default replace), step: absolute\|percent (default absolute), repeat (default false), requirements?}`; one rule per `(slot_entry, field, iteration)`. A rule at iteration 1 is always `replace`. | add rule at iteration 1 → stored as replace |
| PROG-3 | Value at iteration n: start empty and walk i = 1..n. At i, the field's rule for i becomes the active rule and is applied; with no rule for i, an active rule with `repeat` is applied again; otherwise a waiting rule (PROG-6) is tried; otherwise the value stays. | weight replace 60 @1, add 2.5 repeat @2 → @4 = 67.5 |
| PROG-4 | Applying: `replace` → value; `add`/`subtract` with `absolute` → current ± value; with `percent` → current ± current × value / 100; an empty current counts as 0. Result capped at 9999.99 (`rir`, `max_rir`: 9.5). | 100 + 10 % → 110 |
| PROG-5 | Requirements: `{rules: subset of weight\|repetitions\|rir\|rest\|max_weight\|max_repetitions (≥ 1), all_sets (default false)}`; unknown keys or rules → `invalid_requirements`. A rule with requirements applies at iteration i > 1 only if the set logs of iteration i − 1 for this slot entry qualify: by default at least one log has, for every listed rule, its logged value ≥ the shown target of iteration i − 1 (`max_weight`/`max_repetitions` read the top of the range, or the base value when no range); with `all_sets`, at least `sets` logs (min 1) were made and every one qualifies. | target 8 reps, logs 8, 6 → qualifies; with all_sets → not |
| PROG-6 | A gated rule that does not qualify is skipped; if it does not `repeat` it waits and is tried on each next iteration until it applies or a newer rule for that field replaces it. A `max_*` rule is gated by its base field's current rule. | add 5 gated @2 fails, passes @3 → applied once at @3 |
| PROG-7 | Shown values: `weight` rounded to the entry's `weight_rounding`, `repetitions` to `repetitions_rounding`, `rir` to 0.5, `rest` to 1 s; rounding r gives r × round(x / r), halves to even; no rounding when r is empty. Whole numbers are shown without decimals. A `max_*` value is shown only if it is greater than its base value; `sets` with no value is 1. Units shown only when the base value exists. | 61.3 kg, rounding 2.5 → 62.5; max_weight 50, weight 60 → no range shown |
| PROG-8 | `rir` values (logged or prescribed) are one of 0, 0.5, 1, … 4.5; anything else → `invalid_rir`. | rir 5 → refused |
| PROG-9 | The prescription for an iteration is recomputed when rules, rounding or the slot entry's logs change; a change never alters logs already written ([LOG-2](sessions.md) keeps targets as logged). | edit rule → past log `weight_target` unchanged |

## Out of scope

- Named progression schemes shipped as code (e.g. 5/3/1 calculators); they are expressible as rules.

## Decisions

- Rounding is half-to-even, matching the source's rounding function; implementations must not use half-up here.
- A field's rules are walked iteration by iteration so gated, non-repeating rules can earn one step each time the user qualifies, like the source.

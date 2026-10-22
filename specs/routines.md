# Routines

**Status:** Draft · **Prefix:** RTN

## Purpose

Training plans: a routine of ordered days that repeat over a date range, each day holding slots of exercises with set prescriptions.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| RTN-1 | Routine: `{id, name (≤ 25), description (≤ 1000), start (date), end (date), is_template (default false), fit_in_week (default false), created_at}`. `end` ≥ `start` and `end − start` ≤ 120 days, else `invalid_range`. Owned by one user. | start Jan 1, end Jun 1 → refused |
| RTN-2 | Day: `{routine, order (≥ 1), name (≤ 20), description (≤ 1000), type: custom\|emom\|amrap\|hiit\|tabata\|edt\|rft\|afap (default custom), is_rest (default false), needs_logs_to_advance (default false)}`. `type` is a label only. A routine with no days has an empty calendar. A rest day has no slots: setting `is_rest` deletes the day's slots, and adding a slot to a rest day → `rest_day`. | routine without days → calendar `[]`; add slot to rest day → refused |
| RTN-3 | Slot: `{day, order (≥ 1), comment (≤ 200)}` holding ≥ 0 slot entries. A slot with more than one entry is a superset. | slot with 2 entries → superset |
| RTN-4 | Slot entry: `{slot, exercise, order, type: normal\|warmup\|dropset\|myo\|partial\|forced\|tut\|iso_hold\|jump (default normal), comment (≤ 100), weight_unit (default kg), repetitions_unit (default repetitions), weight_rounding?, repetitions_rounding?}` plus progression rules ([PROG](progression.md)). New entries copy the rounding defaults from the profile ([PRF-1](profile.md)) and get `order` = last + 1. | new entry, profile weight_rounding 2.5 → entry 2.5 |
| RTN-5 | Weight units: `kg\|lb\|body_weight\|plates\|km_per_hour\|miles_per_hour`. Repetition units: `repetitions\|until_failure\|max_reps\|seconds\|minutes\|meters\|kilometers\|miles`. Only `kg` and `lb` convert ([PRF-3](profile.md)); only `repetitions` counts as reps in statistics ([STAT](records.md)). | log 30 `seconds` → excluded from volume |
| RTN-6 | Calendar: walk the dates from `start` to `end`. The first date gets day 1. Each next date moves to the next day by `order`, wrapping to day 1 after the last, unless the current day has `needs_logs_to_advance`, there is no session for that day on the previous date ([LOG-3](sessions.md)), and the date is not after the user's local today; then the current day repeats. Today's workout: the calendar entry whose date is the user's local today; none if outside the range or a free date. | 3 days, no flags → Mon d1, Tue d2, Wed d3, Thu d1; today after `end` → none |
| RTN-7 | Iteration: the k-th run of a day in the calendar has iteration k; a day repeated by RTN-6 keeps its iteration. Iterations select progression values ([PROG-3](progression.md)). | d1 on Mon and Thu → iterations 1, 2 |
| RTN-8 | `fit_in_week`: when the day count is not a multiple of 7 and the cycle wraps to day 1, the dates up to the next Monday get no day (free), and day 1 resumes that Monday. | 3 days, start Mon, fit → Thu–Sun free, d1 next Mon |
| RTN-9 | Labels: `{routine, start_offset ≥ 0, end_offset ≥ start_offset, text (≤ 35)}`; a label applies to dates `start + start_offset` through `start + end_offset` and is shown on those calendar dates. | label 0–6 "deload" → first 7 dates labelled |
| RTN-10 | Set expansion for doing a day: a single-entry slot gives its entry `sets` times; a superset interleaves round-robin, one set per entry per round, skipping entries whose sets are done. | A×3, B×2 → A, B, A, B, A |
| RTN-11 | Templates: a template (`is_template = true`) is read-only except for clearing the flag. Copying any routine creates a new non-template routine owned by the copier with `start` = local today, `end` = today + (original end − original start), and copies of all days, slots, entries, rules and labels; logs and sessions are not copied. | copy 8-week template → new routine 8 weeks from today, no logs |
| RTN-12 | Deleting a routine deletes its days, slots, entries, rules and labels; its sessions and set logs remain with `routine` and `slot_entry` cleared. | delete routine → logs still listed in history |

## Out of scope

- Public or shared templates, trainer-assigned routines, calendar (iCal) and PDF export.
- Custom progression code plug-ins per slot entry.

## Decisions

- Deleting a routine keeps the training history; the source kept logs too but by clearing links, which this spec makes explicit.
- Day `type` is a display label only; the source stored it without behaviour.
- Public templates removed: a personal app does not share.

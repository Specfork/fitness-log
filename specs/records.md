# Records

**Status:** Draft · **Prefix:** STAT

## Purpose

Numbers derived from set logs (estimated one-rep max, personal records, volume, intensity) and the rules every progress chart follows.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| STAT-1 | Qualifying set: `repetitions_unit = repetitions`, `weight_unit` in `kg\|lb`, weight > 0, repetitions ≥ 1. Weights are converted to kg ([PRF-3](profile.md)) before any formula here. | 100 lb × 5 → 45.36 kg × 5 |
| STAT-2 | Estimated one-rep max (Brzycki) = weight_kg × 36 / (37 − repetitions), rounded to 2 decimals, defined only for 1 ≤ repetitions ≤ 10; RiR is ignored. | 100 kg × 5 → 112.50 |
| STAT-3 | Personal records per exercise, over all qualifying sets: `best_estimated_1rm` (STAT-2), `heaviest_weight` (max weight_kg with ≥ 1 rep), `most_reps_at_weight` (max repetitions per exact weight_kg). Each record keeps `{value, log_id, performed_at}`; the earliest log wins ties. Saving a log that beats a current record (strictly greater) marks that log as a new record of that kind; editing or deleting logs recomputes records from the remaining logs. | 100×5 then 105×3 → 1RM 112.50 stays, heaviest 105; delete record log → previous best restored |
| STAT-4 | Daily 1RM series per exercise: one point per local day with qualifying sets of ≤ `max_reps` (1–10, default 5): the day's highest STAT-2 value. | sets 3 reps and 8 reps same day, max 5 → only 3-rep set counts |
| STAT-5 | 1RM total: for 2–5 distinct exercises and a `window_days` of 7–120 (default 30), one point on each local day any of them was trained: sum over the exercises of their best daily 1RM within the window ending that day; no point when any exercise has none in the window. | bench+squat, squat last trained 40 d ago, window 30 → no point |
| STAT-6 | Volume = Σ weight_kg × repetitions over qualifying sets (non-qualifying count 0); set count = number of logs; intensity = average of weight_kg / 1RM_estimate (= (37 − repetitions) / 36) over qualifying sets with repetitions ≤ 10. | 2 sets 100 kg × 5 → volume 1000, intensity 0.89 |
| STAT-7 | STAT-6 values are grouped for a routine by local day, ISO week, iteration and whole routine, each split by exercise and by primary muscle ([EXC-1](exercises.md)); a muscle total counts each set once per primary muscle. | squat with 2 primary muscles → both get its volume |
| STAT-8 | Chart buckets: a series is condensed to calendar buckets `hour\|day\|week\|month` (or `auto`) in the user's time zone; `auto` takes the finest unit that yields ≤ `max_points` (default 200) buckets, else `month`. Weeks start on the user's `week_start` (`@specfork/auth:AUTH-1`). | 3 years of daily weights, auto → week |
| STAT-9 | Each bucket returns `{start, count, sum, min, max}`, mean = sum / count, grouped also by stored unit so kg and lb are never averaged together; the client converts then merges. An entry that already summarises a range (e.g. an imported day of steps with `min`/`max`) contributes its own bounds. | bucket with 80 kg and 176.37 lb → two rows |
| STAT-10 | Charts available: body measurement categories ([BODY](body.md)), STAT-4 and STAT-5 series, routine volume/sets/intensity (STAT-7), daily energy and macros logged vs goal ([NUT-9](nutrition.md)). Each chart covers a chosen date range and shows only the user's data. | chart weight last 90 days → points only in range |

## Out of scope

- Trophies, badges, streak rewards, leaderboards or comparisons with other people.

## Decisions

- One 1RM formula (36 / (37 − reps)) for records and series; the source added RiR to reps for records but not for series. Capped at 10 reps because the estimate degrades above that.
- Volume and intensity include lb sets after conversion; the source dropped non-kg sets.
- Records replace the source's trophy for beating an estimate; no gamification.

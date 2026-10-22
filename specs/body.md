# Body

**Status:** Draft · **Prefix:** BODY

## Purpose

Body weight and other body measurements over time, values calculated from them, weight import, and private progress photos.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| BODY-1 | Measurement category: `{id, name (1–100), unit (≤ 30), metric_type, chart_type?: line\|bar\|heatmap\|delta\|distribution, order, calculated: none\|bmi\|waist_to_height\|one_rep_max\|one_rep_max_total (default none), calculated_params}`. At most one category per user per `metric_type` other than `custom`. | second `body_fat` category → refused `duplicate_metric_type` |
| BODY-2 | `metric_type`: `custom\|body_weight\|body_fat\|lean_body_mass\|height\|steps\|distance\|energy`; anything else → `invalid_metric_type`. | `heart_rate` → refused |
| BODY-3 | Entry: `{id, category, measured_at (default now), value (≥ 0, 2 decimals, ≤ 999999.99), unit?, notes (≤ 100), source: manual\|import\|calculated}`. `unit` is the unit the value was entered in, default the category unit. Only body weight converts between `kg` and `lb` on read ([PRF-3](profile.md)); any other unit mismatch → `unit_mismatch`. | weight 80 kg read as lb → 176.37 |
| BODY-4 | Hard limits (value in stored unit, outside → `implausible_value`): body_weight 20–350 kg / 44–770 lb; body_fat 2–60 %; lean_body_mass 10–250 kg; height 50–250 cm; steps 0–100000; distance 0–500 km; energy 0–10000 kcal; custom: BODY-3 range. | body weight 900 kg → refused |
| BODY-5 | Body weight: each user has one official `body_weight` category (unit = profile unit at creation), created on first need; it cannot be deleted while it has entries. "Latest body weight" = its entry with the newest `measured_at`. | first weight entry → category auto-created |
| BODY-6 | Calculated categories are read-only (manual entries → `read_only`), hold `source: calculated` entries only, and cannot be the input of another calculated category. They are recomputed whenever an input entry, the profile height or their params change. | add entry to BMI category → refused |
| BODY-7 | `bmi`: one entry per body weight entry, at the same time, value = weight_kg / (height_cm / 100)², 2 decimals; none while profile height is empty. Shown as a number labelled "estimate", with no category ([PRIV-9](privacy.md)). | 80 kg, 180 cm → 24.69, labelled estimate |
| BODY-8 | `waist_to_height` (param `category_id` of a length category: unit `mm\|cm\|m\|in`): one entry per source entry, value = length_cm / height_cm, 2 decimals; unknown unit entries are skipped. | waist 34 in, 180 cm → 0.48 |
| BODY-9 | `one_rep_max` (params `exercise_id`, `max_reps` 1–10 default 5) and `one_rep_max_total` (params `exercise_ids` 2–5, `max_reps`, `window_days` 7–120 default 30) hold the series of [STAT-4](records.md) and [STAT-5](records.md) as entries in kg. | log new squat set → 1RM category gains a point |
| BODY-10 | Weight import from CSV text: ≤ 1000 rows of `date, value`, a user-chosen date format, decimal comma accepted, value in the profile unit. Rows that fail to parse, break BODY-4, repeat a date in the file or match an existing entry's `measured_at` are skipped; the response lists created count and skipped rows with reasons. | 3 rows, 1 duplicate → 2 created, 1 skipped |
| BODY-11 | Progress photo: `{id, taken_on (date), image (JPEG/PNG/WebP, ≤ 10 MB), width, height, description (≤ 1000)}`. Location and device metadata are stripped before storage; photos are served only to their owner ([PRIV-2](privacy.md)). | upload photo with GPS EXIF → stored file has none |
| BODY-12 | Deleting a category deletes its entries; deleting an input category empties calculated categories that read it. | delete waist category → WHtR entries gone |

## Out of scope

- Import from phone health platforms or wearables (the `import` source is reserved for them).
- Blood pressure, heart rate, resting heart rate, blood oxygen and sleep (stages and totals): not in 0.1.
- Soft "normal range" bands, warnings or colour coding of health values.

## Decisions

- Entries keep the unit they were typed in; conversion on read avoids rounding drift, as in the source.
- No range bands: the source used soft limits for warnings; here they would read as medical judgement ([PRIV-9](privacy.md)).
- Photo metadata stripping and a size limit are added; the source stored uploads as given.

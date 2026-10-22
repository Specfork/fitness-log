# Profile

**Status:** Draft · **Prefix:** PRF

## Purpose

The user's units and body profile, and the exact formulas that turn them into estimates.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| PRF-1 | Profile (one per user, created with the account): `unit_system: metric\|imperial` (default `metric`), `energy_unit: kcal\|kj` (default `kcal`), `birthdate?`, `height_cm?` (100–250, 1 decimal), `sex_for_formulas?: male\|female` (no default), `weight_rounding?`, `repetitions_rounding?` (each 0.01–99.99, default null = no rounding). `birthdate` must give an age of 10–100 full years today; age = full years between `birthdate` and the user's local today. | birthdate 5 years ago → refused |
| PRF-2 | Local day: a timestamp counts for the calendar date it has in the user's `time_zone` (`@specfork/auth:AUTH-1`). Every per-day and "today" rule in this spec uses it. | 23:30 UTC, zone `Asia/Tokyo` → next date |
| PRF-3 | Units: `metric` shows kg and cm, `imperial` shows lb and inches. Exact factors: 1 lb = 0.45359237 kg, 1 in = 2.54 cm, 1 kcal = 4.184 kJ. Shown values round half-up to 2 decimals; stored values keep the unit they were entered in ([BODY-3](body.md)). | 100 kg shown imperial → 220.46 lb |
| PRF-4 | Changing `unit_system` or `energy_unit` changes display and new-entry defaults only; no stored value is rewritten. Profile edits never rewrite past entries; formulas always read the current profile and the latest body weight. | switch to imperial → old entries still stored in kg; change height → past weight entries unchanged |
| PRF-5 | Basal rate estimate (Mifflin–St Jeor), kcal/day = 10 × weight_kg + 6.25 × height_cm − 5 × age_years + s, where s = +5 for `male`, −161 for `female`; weight_kg is the latest body weight entry ([BODY-3](body.md)); result rounded to 2 decimals and labelled "estimate". Any input missing → `missing_inputs` listing them. | 80 kg, 180 cm, 30 y, male → 1780.00, labelled estimate |
| PRF-6 | Activity inputs: `sleep_hours` 4–10 (default 7), `work_hours` 1–15 (default 8), `work_intensity`, `sport_hours_per_week` 0–30 (default 3), `sport_intensity`, `free_hours` 1–15 (default 8), `free_intensity`; intensities `low\|medium\|high` (defaults low, medium, low). `sleep_hours + work_hours + free_hours` > 24 → `hours_exceed_day`. | 10 + 10 + 10 → refused |
| PRF-7 | Activity factor = (sleep_hours × 0.95 + work_hours × w + sport_hours_per_week / 7 × p + free_hours × f) / 24, rounded to 2 decimals; w = 1.5/1.8/2.2, p = 4/6/10, f = 1.3/1.9/2.4 for low/medium/high. | defaults → 1.32 |
| PRF-8 | Daily energy estimate = round(basal rate × activity factor) kcal, computed only on request, labelled "estimate", and never written anywhere unless the user copies it into a plan goal ([NUT-2](nutrition.md)). | 1780 × 1.32 → 2350 kcal; plan goal unchanged |

## Out of scope

- Email reminders (workout ending, weight entry overdue), notification language, gamified trophies.
- Gym membership, trainer-created users, contracts.

## Decisions

- No default sex: the source defaulted to male, which silently skews the basal rate. Without it the estimate is unavailable.
- Height 100–250 cm instead of the source's 140–230, which rejected real adults.
- lb ↔ kg uses the exact international factor both ways; the source used a rounded inverse.
- Sport hours may be 0; the source required at least 1.

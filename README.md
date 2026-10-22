---
specfork: 1
name: "@specfork/fitness-log"
summary: Personal fitness and health log with exercises, routines with progression, set logging, records, body measurements, food logging with macros, charts and private data export.
license: CC-BY-4.0
tags: [personal, health]
requires:
  "@specfork/auth": "^0.3.0"
  "@specfork/privacy": "^0.1.0"
derived_from:
  repo: https://github.com/wger-project/wger
  commit: 48e02bfe45e83b87781058261986cda88ff2e6d5
  url: https://wger.de
  license: AGPL-3.0-or-later
---

# Fitness log

One person keeps their training and health data in one private place: an exercise catalogue,
routines that plan sets, reps and weight week by week, a log of every set actually done, personal
records, body weight and other measurements, nutrition plans and a food diary with calories and
macros, and charts over all of it. Metric or imperial units, the user's own time zone.
Nothing is shared, and everything can be exported, imported or deleted at once.

Behaviour derived from wger's open-source code (AGPL-3.0-or-later), written fresh and stack-free.
wger's exercise and food data (Creative Commons and ODbL licences, set per entry) were not used;
catalogue and food data come through the generic source contracts in [EXC](specs/exercises.md)
and [FOOD](specs/foods.md).
Not affiliated with or endorsed by the wger project. The name is used only to credit the source.

| Area | Prefix | Covers |
|---|---|---|
| [profile.md](specs/profile.md) | PRF | Units, body profile, energy estimate formulas |
| [exercises.md](specs/exercises.md) | EXC | Exercise catalogue, catalogue source, custom exercises, search |
| [routines.md](specs/routines.md) | RTN | Routines, days, slots, set types, calendar sequence, templates |
| [progression.md](specs/progression.md) | PROG | Per-iteration rules for sets, weight, reps, RiR, rest; gating; rounding |
| [sessions.md](specs/sessions.md) | LOG | Workout sessions and set logs |
| [records.md](specs/records.md) | STAT | 1RM estimate, personal records, volume, intensity, charts |
| [body.md](specs/body.md) | BODY | Body weight, measurements, calculated metrics, import, progress photos |
| [foods.md](specs/foods.md) | FOOD | Foods, nutrients, portions, validation, food source contract |
| [nutrition.md](specs/nutrition.md) | NUT | Nutrition plans, meals, food diary, totals and goals |
| [privacy.md](specs/privacy.md) | PRIV | Single owner, owner-only access, inventory, export contents, import, delete-all, no advice |

Accounts and sign-in come from `@specfork/auth`, in `single_owner` mode by default ([PRIV-1](specs/privacy.md));
export, import and delete-all from `@specfork/privacy`. All times are stored in UTC; calendar days
follow the user's time zone ([PRF-2](specs/profile.md)); API names are `snake_case`.

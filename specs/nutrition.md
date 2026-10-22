# Nutrition

**Status:** Draft · **Prefix:** NUT

## Purpose

Nutrition plans with meals and goals, a food diary, and the exact arithmetic for calories and macros.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| NUT-1 | Plan: `{id, description (≤ 80), start (default local today), end?, diary_only (default false), created_at}`; `end` ≥ `start` else `invalid_range`. A plan is active on dates in `[start, end]` (open end = no end). The current plan = active plan with the latest `start`. | two active plans → later start is current |
| NUT-2 | Goals per plan, each optional integer ≥ 0: `goal_energy_kcal`, `goal_protein_g`, `goal_carbohydrates_g`, `goal_fat_g`, `goal_fiber_g`. Goals are set only by the user. | estimate shown → goal stays empty until saved |
| NUT-3 | Meal: `{plan, order, time?, name (≤ 25)}`; meal item: `{meal, food, portion?, amount (0.01–1000, 2 decimals), order}`. `diary_only` plans have no meals (adding one → `diary_only`). | add meal to diary-only plan → refused |
| NUT-4 | Item grams = amount × portion.grams, or amount when no portion. Item nutrient = food nutrient per 100 g × grams / 100, for energy and each FOOD-1 nutrient ([FOOD-1](foods.md)). Stored unrounded; shown rounded to 1 decimal (energy to whole kcal). | 2 slices of 30 g, 250 kcal/100 g → 150 kcal |
| NUT-5 | Totals of a meal, plan or day = sum of items. For `sugar`, `saturated_fat`, `fiber`, `sodium`: items lacking the value are skipped, and the total is empty only if every item lacks it. | 2 items, fiber 3 and empty → fiber 3 |
| NUT-6 | Energy share of a macro = macro_g × factor / total_energy_kcal × 100 with factors protein 4, carbohydrates 4, fat 9; all 0 when total energy is 0. Shares need not sum to 100. | P 100 g, energy 2000 → 20 % |
| NUT-7 | Per body weight: macro_g / weight_kg, using the body weight entry closest in date to the plan's `created_at` (tie → earlier entry); empty when there is no body weight. | P 160 g, 80 kg → 2.0 g/kg |
| NUT-8 | Diary entry: `{id, plan, meal?, eaten_at (default now), food, portion?, amount (0.01–1000), comment?}`; nutrients per NUT-4. Deleting a meal clears `meal` on its diary entries; deleting a plan deletes its meals and diary entries. | delete meal → its diary entries keep `meal = null` |
| NUT-9 | Day view: for a local date, diary entries of the plan with their totals, the plan's planned totals (sum of meals), and per goal: `remaining = goal − logged` (may be negative). Diary history: per local day totals for a date range, energy in the user's `energy_unit`; days without entries are omitted. | goal 2000, logged 1500 → remaining 500; 7-day range, 5 logged days → 5 rows |
| NUT-10 | Log a planned meal: creates one diary entry per item of the meal with the same food, portion and amount, `meal` set and `eaten_at` = given time or now. | meal of 3 items → 3 diary entries |
| NUT-11 | Goal band for a day's energy vs `goal_energy_kcal`: `on_target` within ±3 %, `close` within ±7 %, `off` within ±10 %, else `far` (bounds exclusive). Shown as a band name only, no advice. | goal 2000, logged 2100 → `close` |
| NUT-12 | Copy plan: new plan owned by the user with `start` = local today, same description, goals, meals and items; diary entries are not copied. | copy plan with diary → copy has 0 entries |

## Out of scope

- Meal suggestions, diet generation, micronutrient tracking beyond FOOD-1, PDF export of plans.

## Decisions

- Amount minimum 0.01 instead of the source's 1, so fractional portions (half a slice) work.
- Goals never fill themselves from the energy estimate ([PRF-8](profile.md)).
- Goal bands compare the logged day, not the plan; the source compared plan totals with a profile calorie value.

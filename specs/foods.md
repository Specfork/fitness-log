# Foods

**Status:** Draft · **Prefix:** FOOD

## Purpose

The foods a user can plan and log, their nutrients per 100 g and portion sizes, the checks every food passes, and a generic contract for external food sources.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| FOOD-1 | Food: `{id, uuid, origin: source\|custom, name (3–200), brand? (≤ 200), language, barcode? (≤ 200), energy_kcal (integer ≥ 0), protein, carbohydrates, sugar?, fat, saturated_fat?, fiber?, sodium?, is_vegan?, is_vegetarian?, updated_at}`. All nutrients are grams per 100 g, 0–100, 2 decimals. Energy given only in kJ is converted: kcal = round(kJ / 4.184). Shown in the user's `energy_unit` ([PRF-1](profile.md)). | protein 120 → refused; 1674 kJ → 400 kcal |
| FOOD-2 | Consistency: `saturated_fat` ≤ `fat`; `sugar` ≤ `carbohydrates`; `protein + carbohydrates + fat` ≤ 105. Failing → `inconsistent_nutrients` naming the rule. | sugar 30, carbs 20 → refused |
| FOOD-3 | Energy check: computed = 4 × protein + 4 × carbohydrates + 9 × fat + 2 × (fiber or 0). Skipped when both `energy_kcal` and computed are ≤ 13; otherwise computed must lie within [energy_kcal × 0.7 − 5, energy_kcal × 1.3 + 5], else `energy_mismatch`. | 400 kcal, P 10 C 10 F 1 → refused |
| FOOD-4 | Portion: `{food, name (1–200), grams (integer 1–5000)}`; a food has 0–20 portions; plain grams are always available without a portion. | add "slice" 30 g → choosable in diary |
| FOOD-5 | Custom food: `origin: custom`, owned by one user, private ([PRIV-2](privacy.md)), editable and deletable by them; deleting one that plans or diary entries use → `in_use`. | delete custom food used in diary → refused |
| FOOD-6 | Source foods also carry `{source_name, remote_id, source_url?, license_id (SPDX), attribution?}`, shown on the food detail; they are read-only to users, and the user may copy one into a custom food to correct it. | edit source food → refused; copy → custom food |
| FOOD-7 | Food source contract: `search(text, language, limit ≤ 50)` → candidates; `lookup_barcode(code)` → candidate or none; candidate = FOOD-1 fields + FOOD-6 fields + `portions[]`. A candidate failing FOOD-1–3 is dropped. Several sources may be configured; none is configured by default ([PRIV-4](privacy.md)). | candidate with fat 150 → not offered |
| FOOD-8 | A source candidate is stored locally on first use (upsert by `source_name` + `remote_id`) so plans and diary keep working when the source is unreachable; later lookups refresh fields that changed. | source offline → logged food still shows nutrients |
| FOOD-9 | Requests to a food source carry only the search text, language, barcode and limit, never user identifiers or diary data; a source failing or timing out after 5 s gives local results only, flagged `source_unavailable`. | source timeout → local matches + flag |
| FOOD-10 | Search: `q` (≥ 2 chars) case-insensitive on name and brand in the user's language, local foods first (user's custom, then stored source foods), then source candidates not yet stored; filters `is_vegan`, `is_vegetarian`; exact barcode in `q` returns that food first. | `q=oat` → custom "oat mix" before source foods |

## Out of scope

- Bundled food datasets, bulk dataset sync, product images, quality grades from specific databases.
- Recipes built from other foods.

## Decisions

- Food databases are a generic source contract; no vendor is named or required.
- The 105 g cap and energy tolerance follow common label tolerances, as in the source.
- Source foods are cached locally so the diary never depends on a third party at read time.

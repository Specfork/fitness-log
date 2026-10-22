# Privacy

**Status:** Draft · **Prefix:** PRIV

## Purpose

Health data is sensitive: who may see it, what is stored, what leaves the instance, and what its export, import and delete-all cover.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| PRIV-1 | Default auth mode `single_owner` (`@specfork/auth:AUTH-18`); any further account (`@specfork/auth:AUTH-22`) is a separate, fully isolated owner. | second sign-up on fresh instance → refused |
| PRIV-2 | Owner-only access: every routine, session, log, measurement, photo, plan, diary entry, custom exercise, custom food and profile belongs to one user and is readable and writable only by them; a request for another user's record → `not_found` (never `forbidden`). Instance admins get no read access to it either. | user B fetches A's weight entry → not found |
| PRIV-3 | Inventory (`@specfork/privacy:PRV-9`): account (`@specfork/auth`), profile ([PRF-1](profile.md), [PRF-6](profile.md)), routines, sessions and logs, measurements and photos, plans and diary, custom exercises and foods; each kept until the user deletes it. No location, contacts, device identifiers or advertising IDs are collected. | manual: inventory lists these and the data model has no other personal fields |
| PRIV-4 | No sharing: no public pages, share links, social feeds or third-party analytics. Outbound traffic goes only to the operator-configured exercise source ([EXC-6](exercises.md)) and food sources ([FOOD-9](foods.md)), which receive no personal data; the inventory names them (`@specfork/privacy:PRV-10`). | manual: network log of a full session shows no other hosts |
| PRIV-5 | Export (`@specfork/privacy:PRV-1`), `format` `@specfork/fitness-log`, `version` 1, one JSON Lines file per entity: `profile`, `routines`, `routine_days`, `slots`, `slot_entries`, `progression_rules`, `routine_labels`, `sessions`, `set_logs`, `measurement_categories`, `measurements`, `progress_photos`, `plans`, `meals`, `meal_items`, `diary_entries`, `custom_exercises`, `custom_foods` (with portions). Catalogue exercises and source foods are referenced by `uuid` and name. Also `photos/<id>.<ext>` (original files) and `set_logs.csv`, `measurements.csv`, `diary.csv` for spreadsheets. | export → ZIP contains every log and photo |
| PRIV-6 | Import (`@specfork/privacy:PRV-5`): referenced catalogue exercises and source foods are matched by `uuid`; unknown ones become custom items with the exported name. Calculated categories are recomputed, not imported ([BODY-6](body.md)). | import into empty account → same charts and records |
| PRIV-7 | Delete all my data (`@specfork/privacy:PRV-6`) removes every record of PRIV-3 except the account, photo files included, and resets the profile to defaults; account deletion is `@specfork/privacy:PRV-7`. | delete all → history, weights, photos empty; can still sign in |
| PRIV-8 | Health values (weights, measurements, food, notes, photos) never appear in logs, error reports, URLs or query strings; records are addressed by id only. | manual: server log after logging weight has no value |
| PRIV-9 | No medical features: the app shows values and the stated formulas only; it gives no diagnosis, risk score, health category (e.g. BMI classes), normal ranges, warnings or advice. Every estimate (basal rate, daily energy, BMI, 1RM) is labelled "estimate". | BMI 31 → shown "31.00 (estimate)", no class label |

## Out of scope

- Sharing with trainers, coaches, gyms or friends; multi-user teams (`@specfork/teams` is not used).
- End-to-end encryption.

## Decisions

- Not-found instead of forbidden, so record ids reveal nothing.
- One owner per instance by default (`single_owner`); extra accounts possible but never connected.
- Export packaging, delivery, limits, re-auth, delete-all timing and backup purge come from `@specfork/privacy`; this area keeps what is health-specific.
- Delete-all keeps the account so a user can restart cleanly; the source only offered full account deletion.

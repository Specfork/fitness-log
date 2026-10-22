# Exercises

**Status:** Draft · **Prefix:** EXC

## Purpose

The exercises a user can plan and log: a read-only catalogue filled from an exercise source, plus the user's own private exercises.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| EXC-1 | Exercise: `{id, uuid, origin: catalogue\|custom, category, primary_muscles[], secondary_muscles[], equipment[], variation_group?, translations[], images[], license, updated_at, archived (default false)}`. Translation: `{language (ISO 639-1), name (2–200 chars), description (≤ 2000 chars, plain text), aliases[] (each ≤ 200)}`; one translation per language. Variations: exercises sharing `variation_group` are listed on each other's detail. | two `en` translations → second refused `duplicate_language`; two exercises in one group → each lists the other |
| EXC-2 | Reference lists: `category {id, name}`, `muscle {id, name, is_front}`, `equipment {id, name}`. They arrive with the catalogue and are read-only; custom exercises pick from them. | create custom exercise with unknown muscle id → refused |
| EXC-3 | Display name: translation in the user's `locale` (`@specfork/auth:AUTH-1`), else `en`, else the first translation by language code. | locale `de`, only `en` and `fr` → `en` name |
| EXC-4 | Licence and attribution per catalogue exercise, image and translation: `{license_id (SPDX), author?, author_url?, source_url?, derived_from_url?}`; shown on the exercise detail. Custom exercises carry none. | catalogue exercise detail → licence and author shown |
| EXC-5 | Exercise source contract: `list_changes(since?, cursor?)` → pages of `{exercises[], reference_lists, deletions: [{uuid, replaced_by_uuid?}], next_cursor?}`. A sync upserts by `uuid`, stores the newest `updated_at` seen and passes it as `since` next time. The source is configured by the instance operator; with none configured the catalogue is empty and the app still works. | second sync with no upstream change → 0 writes |
| EXC-6 | Sync is pull-only: it sends only `since` and `cursor`, never user data, and runs on demand or at most daily. A failed page leaves earlier pages applied and `since` unchanged. | network error on page 3 → pages 1–2 kept, retry resumes |
| EXC-7 | A source deletion with `replaced_by_uuid` re-points the user's slot entries ([RTN-4](routines.md)) and set logs ([LOG-1](sessions.md)) to the replacement, then removes the exercise. Without a replacement, an exercise that any routine or log uses is set `archived` instead of removed; an unused one is removed. Logs are never deleted by a sync. Archived exercises stay readable and loggable from existing routines but are hidden from search and from choosing in new slot entries. | upstream deletes used exercise → logs keep it, archived; search for archived name → no hit |
| EXC-8 | Custom exercise: `origin: custom`, owned by one user, visible only to them ([PRIV-2](privacy.md)); same fields as EXC-1 except licence; at least one translation in the user's locale. Edit and delete are owner-only; deleting one that routines or logs use archives it instead. | delete used custom exercise → archived, logs intact |
| EXC-9 | Search: `q` (≥ 2 chars) matches name or alias words by case-insensitive prefix in the chosen language plus `en`; filters `category`, `muscle` (primary or secondary), `equipment` (all given must match), `origin`. Ordered: exact name match, then name prefix, then alias match, then by name; page size ≤ 50. | `q=bench`, equipment barbell → barbell bench press first |

## Out of scope

- Crowd editing of the shared catalogue: submissions, edit history, trust levels, review, translation workflows.
- Exercise videos and image upload to the catalogue; the catalogue's own media hosting.
- Muscle diagrams.

## Decisions

- Catalogue changes never delete a user's history: unused exercises are removed, used ones archived. The source cascaded deletes.
- The catalogue source is a generic contract; no dataset is bundled. Licences are kept per item because catalogue data may be licensed differently from the app.
- Custom exercises are private; the source let users add to the shared catalogue.

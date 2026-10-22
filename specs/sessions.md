# Sessions

**Status:** Draft · **Prefix:** LOG

## Purpose

What the user actually did: one set log per performed set, grouped into workout sessions.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| LOG-1 | Set log: `{id, performed_at (default now), exercise, session, routine?, slot_entry?, iteration?, repetitions?, repetitions_unit, weight?, weight_unit, rir?, rest? (seconds, integer ≥ 0), next_log?, plus *_target copies}`. `repetitions`, `weight` ≥ 0 with 2 decimals, ≤ 9999.99. | weight −5 → refused |
| LOG-2 | A log needs `repetitions` or `weight` (both empty → `empty_log`); a unit is required for each value given. Logging from a routine copies the shown prescription of that iteration into `repetitions_target`, `weight_target`, `rir_target`, `rest_target`; targets never change afterwards. | log with neither value → refused |
| LOG-3 | Session: `{id, routine?, day?, started_at (default now), ended_at?, notes?, impression: bad\|neutral\|good (default neutral)}`. `ended_at` ≥ `started_at`, and the span ≤ the instance's `max_session_hours` (default 5) → else `invalid_session_span`. A session counts for the local day of `started_at` ([PRF-2](profile.md)). | session 23:00–01:00 → counts for first date |
| LOG-4 | A log without a session is attached on save: first a session of the same routine whose `[started_at, ended_at]` contains `performed_at`; else the newest open session of that routine started within `max_session_hours` before `performed_at`; else a new session starting at `performed_at`. Concurrent saves for one user must not create two sessions. | two logs 10 min apart, no session → one session |
| LOG-5 | When all routine-linked logs of a session come from slot entries of one day of that session's routine and the session has no day, the session's `day` is set to it. | 3 logs all from day "Legs" → session day = Legs |
| LOG-6 | Drop sets and similar chains: `next_log` points to the following log of the chain; deleting a log deletes the logs after it in its chain. Deleting a session deletes its logs. Deleting a log never deletes its session. | delete first of 3-chain → all 3 gone; delete session with 5 logs → 0 logs left |
| LOG-7 | A log or session may reference only the user's own routine, slot entry, session or log; a foreign reference → `not_found`, nothing saved. | log with another user's session → refused |
| LOG-8 | History: logs listed newest first, filterable by `exercise`, `routine`, local date range; grouped by session with the session's notes and impression. Logging works without a routine: a free session with any exercises, `routine`, `slot_entry` and `iteration` empty. | filter exercise squat → only squat logs; log curl with no routine → stored, new session |

## Out of scope

- Live workout timer, rest timer sounds, wearable heart-rate capture during sets.

## Decisions

- Chained logs are deleted together; the source cascaded the same way.
- Sessions longer than the limit are refused, not split.

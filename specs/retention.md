# Retention

**Status:** Draft · **Prefix:** RET

## Purpose

How long analytics data lives, and how it is removed.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| RET-1 | `retention_months` per site: 1–120 or null (keep forever, the default). Instance setting `max_retention_months` (default null = no cap) caps it; a larger value → refused. Cutoff = start of today (UTC) − `retention_months`. | instance cap 24, site 36 → refused |
| RET-2 | A purge runs at least daily and deletes the site's events, event properties, session links and sessions whose events are all before the cutoff; session properties go with their session. | 13-month-old event, retention 12 → gone after purge |
| RET-3 | Queries never return data before the cutoff or `reset_at`, even if the purge hasn't run yet ([QRY-5](query.md)). | before purge, range 2 years → only last 12 months |
| RET-4 | Reset: deletes all of the site's sessions, events and properties and sets `reset_at` = now; the site, its settings, shares and segments stay; tracking continues. Requires typing the site's name to confirm. | reset → stats 0, new pageview counted |
| RET-5 | Erasure: `DELETE /api/sites/{id}/visitors?distinct_id=` deletes every session linked to that `distinct_id` ([SES-6](sessions.md)) with their events, properties and links; → `{sessions_deleted}`. | erase `u1` with 3 sessions → 3 |
| RET-6 | Deleting a site deletes its analytics data, shares and segments at once; its tracking id then gets `unknown_site` ([COL-3](collect.md)). Deleting a team deletes its sites (`@specfork/teams:TEAM-9`). | delete site, send pageview → 400 |

## Out of scope

- Data import and raw data export (beyond [MET-11](metrics.md)); archiving to cold storage.

## Decisions

- Designed here: the source had no retention setting (its hosted service capped free history separately) and no erasure by visitor.

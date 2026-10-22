# Sessions

**Status:** Draft · **Prefix:** SES

## Purpose

How events group into sessions (visitors) and visits, and what a session records.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| SES-1 | Session: `{id ([PRIV-3](privacy.md)), site_id, browser?, os?, device?, screen?, language?, country?, region?, city?, distinct_id?, created_at}`, created by its first accepted request. Device and location fields come from that first request and are not changed later. | 2nd request from new screen size → session keeps first screen |
| SES-2 | A visitor is a distinct session id: same site, network address, browser and `distinct_id` within one salt period ([PRIV-4](privacy.md)). Another browser, network or period counts as another visitor; with the default daily salt a visit running across 00:00 UTC continues as a new session and visit. | same person on phone and laptop → 2 visitors; browsing 23:50–00:10 UTC → 2 sessions |
| SES-3 | Visit: `visit_id` (random UUID) groups a session's events; a new visit starts when the session has no event in the last 30 minutes. Back-filled events ([COL-5](collect.md)) join the visit whose events lie within 30 min of their `timestamp`, else start one. | events at 10:00, 10:20, 10:55 → 2 visits |
| SES-4 | Device fields: `browser` (family, e.g. `Chrome`, `Safari`, `Firefox`, `Edge`, `Samsung`, `Opera`, in-app and iOS variants named apart), `os` (e.g. `Windows 10/11`, `macOS`, `iOS`, `Android`, `Linux`, `ChromeOS`), `device`: `mobile` \| `tablet` \| `laptop` \| `desktop` \| `other` (a desktop UA with screen width ≤ 1920 → `laptop`), `screen` `WxH` (≤ 11), `language` (BCP 47, ≤ 35). Unknown → null. | iPad UA → `tablet`; desktop UA, `1440x900` → `laptop` |
| SES-5 | Stored lengths: `browser`, `os`, `device`, `region` ≤ 20; `city` ≤ 50; `distinct_id` ≤ 50; longer values are truncated. | 60-char distinct id → first 50 kept |
| SES-6 | Identify (`type: identify`): `distinct_id?` and `data?`. With `distinct_id`, the hash includes it, so later events of this browser go to the identified session, and a link `{site_id, distinct_id, session_id}` is stored so all sessions of one `distinct_id` can be listed. Repeating the same identify writes no new link. | identify `u1` twice → 1 link |
| SES-7 | Session properties: `data` flattened like event properties ([EVT-9](events.md)) and upserted per `(session, key)`: the latest value wins. | identify `{plan: free}` then `{plan: pro}` → `pro` |
| SES-8 | Session list for a range and filters ([QRY](query.md)): `{id, distinct_id?, browser, os, device, country, region, city, first_at, last_at, visits, views, events}`, newest `last_at` first, paged 20 by default (max 100). | 25 sessions → 2 pages |
| SES-9 | Session detail: its fields, properties, all sessions sharing its `distinct_id`, and its events in time order grouped by visit. | open session → events by visit |

## Out of scope

- Merging anonymous history into an identified user retroactively; user-level (cross-device) reports.

## Decisions

- Visits end after 30 minutes of inactivity; the source ended a visit 30 minutes after it started (and hourly without its cache token), which split long visits.
- Device class `other` replaces the source's long tail (console, smart TV, wearable, embedded).

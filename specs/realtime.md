# Realtime

**Status:** Draft · **Prefix:** LIVE

## Purpose

Who is on the site right now and what they just did.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| LIVE-1 | `GET /api/sites/{id}/active` → `{visitors}`: distinct sessions with any event in the last 5 minutes. The site dashboard header shows it and refreshes every 10 s. | event 4 min ago → counted; 6 min ago → not |
| LIVE-2 | `GET /api/sites/{id}/realtime` covers the last 30 minutes (from the start of the current minute − 30): per-minute `pageviews` and `visitors` series, totals `{views, visitors, events, countries}`, top `paths`, `referrers` and `countries` with counts. | 2 sessions from DE and FR → countries 2 |
| LIVE-3 | Its feed lists the newest 100 items, newest first: `{kind: session_start\|pageview\|event, at, session_id, path, name?, referrer_domain?, country, browser, os, device}`; a session's first event in the window also yields one `session_start`. | new visitor views 2 pages → 1 start + 2 pageviews |
| LIVE-4 | Realtime takes filters ([QRY-6](query.md)) but not a range or compare; an open realtime view updates every 10 s without reload, and a collected event shows up within 10 s. | send pageview → appears ≤ 10 s |
| LIVE-5 | `distinct_id` is never part of the feed. | identified visitor → feed shows no distinct id |

## Out of scope

- Push transports (WebSocket, SSE) as a requirement; polling meets LIVE-4.

## Decisions

- 5-minute active window and 30-minute realtime window, as in the source.

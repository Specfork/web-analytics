# Collect

**Status:** Draft · **Prefix:** COL

## Purpose

The public endpoint that receives pageviews, events and identify calls, and what it drops.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| COL-1 | `POST /api/collect`, no authentication, JSON body `{type: event\|identify, payload}`. CORS: any origin, methods `POST, OPTIONS`, request headers `Content-Type, X-Analytics-Cache`, preflight cached 24 h. | preflight from any origin → 204 |
| COL-2 | `payload`: `site` (UUID, required), `url` (absolute URL or path; required for `event`), `referrer?` (URL or path), `hostname?`, `title?`, `screen?` (`^\d{1,5}x\d{1,5}$`), `language?`, `name?`, `data?` (object), `tag?`, `distinct_id?`, `timestamp?` (unix seconds), `user_agent?`, `ip?`, `browser?`, `os?`, `device?`. Unknown fields ignored; a malformed field → 400 naming it. | `screen: "big"` → 400 `screen` |
| COL-3 | Unknown or deleted `site` → 400 `unknown_site`; nothing stored. | random UUID → 400 |
| COL-4 | `name` or `tag` starting with `=`, `+`, `-`, `@`, tab or carriage return → 400 (spreadsheet formula guard). | `name: "=SUM(A1)"` → 400 |
| COL-5 | `timestamp` sets `created_at` (back-fill from servers); more than 60 s in the future → 400; absent → receive time. | timestamp now + 1 h → 400 |
| COL-6 | Success → 200 `{cache, session_id, visit_id}`. `cache` is a token signed (HMAC-SHA256) with an instance secret holding `{site, session_id, visit_id, last_event_at}`; sent back in `X-Analytics-Cache`, it saves lookups. A token with a bad signature, another site, or issued before the current salt period began ([PRIV-4](privacy.md)) is ignored and the ids are recomputed. A paused site ([SITE-6](sites.md)) → 200 `{disabled: true}`, nothing stored. | tampered token → same session as without token |
| COL-7 | `user_agent`, `ip`, `browser`, `os`, `device` in the payload (server-side senders) replace the values taken from the request; they are handled exactly like request values ([PRIV-2](privacy.md)). | payload `ip` from Germany → country `DE` |
| COL-8 | Batch: `POST /api/collect/batch` with an array of ≤ 500 COL-1 bodies, each processed independently in order; → 200 `{size, processed, errors, details: [{index, error}]}`. More than 500 → 400, nothing processed. | 3 items, 1 invalid → processed 2, errors 1 |
| COL-9 | Bots: a request whose user agent is empty or matches the instance's crawler list (search and social crawlers, headless browsers, HTTP libraries, uptime monitors) is not stored and gets 200 `{ignored: true}`. Operator switch `bot_filter` (default on). | UA `Googlebot/2.1` → ignored |
| COL-10 | Excluded IPs: instance list plus the site's `excluded_ips` (≤ 50 entries), each an IPv4/IPv6 address or CIDR range; IPv4-mapped IPv6 addresses compare as IPv4. A match → 200 `{ignored: true}`, nothing stored. | site excludes `10.0.0.0/8`, IP `10.1.2.3` → ignored |
| COL-11 | Client IP: by default the connection's peer address; the operator may name one trusted proxy header (first address of `X-Forwarded-For`, `for=` of `Forwarded`, or a single-value CDN header). Other forwarding headers are ignored. A port suffix is stripped. | spoofed `X-Real-IP`, no header configured → peer IP used |
| COL-12 | Collect answers within 200 ms p95 at 100 requests/s on reference hardware; stored events appear in queries within 10 s. | manual: load test |

## Out of scope

- Web-vitals, replay and heatmap payload types; per-site rate limits and quotas.

## Decisions

- Only one operator-named header is trusted for the client IP; the source took the first of a dozen proxy headers, so any visitor could spoof it.
- Excluded IPs answer like bots (200, ignored) instead of the source's 403, so senders don't retry or learn the list; a per-site list was added to the instance one.
- Events dated in the future are refused; the source accepted any timestamp.

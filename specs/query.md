# Query

**Status:** Draft · **Prefix:** QRY

## Purpose

The time range, bucket unit, time zone and filters every analytics request takes; saved segments.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| QRY-1 | Every analytics read takes `start_at`, `end_at` (unix ms, both inclusive, `start_at` ≤ `end_at`), `unit` (`minute`\|`hour`\|`day`\|`month`\|`year`) and `timezone` (IANA, default the signed-in user's `@specfork/auth:AUTH-1` time zone, else the team's `@specfork/teams:TEAM-13`; invalid → 400). Buckets and calendar boundaries are computed in that time zone. | `timezone=Mars/Base` → 400 |
| QRY-2 | Presets: today, last 24 hours (default), this week, last 7 days, this month, last 30 days, last 90 days, this year, last 6 months, last 12 months, all time (from the site's first event), custom. Weeks start on the user's `week_start` (`@specfork/auth:AUTH-1`), Monday on public shares. The chosen preset is remembered per user. | open dashboard first time → last 24 hours |
| QRY-3 | Minimum unit by range length: ≤ 60 min → `minute`; ≤ 30 days → `hour`; ≤ 7 calendar months → `day`; longer → `month`. A requested unit finer than the minimum is replaced by the minimum; `year` is allowed for any range > 24 months. | 90-day range, `unit=minute` → `day` |
| QRY-4 | Stepping a range back or forward moves it by its own length; calendar presets move by their calendar unit (this month → previous month). Ranges never extend past now. | this month (Mar) back → Feb 1–28/29 |
| QRY-5 | `start_at` is raised to the site's `reset_at` and retention cutoff ([RET-1](retention.md)) before querying. | reset yesterday, range last 7 days → only since reset |
| QRY-6 | Filter: `{field, op, value}`. Event fields: `path`, `entry`, `exit`, `title`, `query`, `hostname`, `referrer`, `event`, `tag`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`, `channel`. Session fields: `browser`, `os`, `device`, `screen`, `language`, `country`, `region`, `city`, `distinct_id`. Unknown field → 400. | `{field: "color"}` → 400 |
| QRY-7 | Operators: `eq`, `neq` (value is a list: any of / none of), `contains`, `not_contains` (case-insensitive substring), `regex`, `not_regex` (case-insensitive; invalid pattern → 400). Default `eq`. | `path contains BLOG` → `/blog/x` matches |
| QRY-8 | Combining: `match` = `all` (default, AND) or `any` (OR). Event filters restrict which events count; session filters restrict which sessions count; a `referrer` filter never matches self-referrals. | `country eq DE` + `path eq /a`, all → German views of /a |
| QRY-9 | Property filters: `{scope: event\|session, key, type, op, value}`; ops by type: string `eq`, `neq`, `contains`, `not_contains`; number `eq`, `neq`, `gt`, `gte`, `lt`, `lte`; boolean `is_true`, `is_false`; date `before`, `after`; any type `is_set`, `not_set`. Op not valid for the type → 400. | number `qty gt 2` → only events with qty > 2 |
| QRY-10 | Segment: saved filters per site `{id, site_id, name (1–200), filters, property_filters, match, created_at}`, visible to all site members; `segment=<id>` adds its filters to the request's (request `match` wins if given). A segment of another site → 404. | apply "Germany mobile" → stats narrowed |
| QRY-11 | Filter value suggestions: for a field and range, up to 100 distinct values ordered by frequency, optional substring `search`. | `field=country` → country codes by traffic |
| QRY-12 | Filters, range, unit, compare and segment live in the dashboard URL, so a copied URL reopens the same view. | paste URL in new tab → same view |

## Out of scope

- Behavioural cohorts (sessions that did X in another range), nested filter groups.

## Decisions

- Readable operator names (`contains`, `not_regex`) instead of the source's two-letter codes.
- Year buckets only for ranges over two years; the source's unit rules differed between preset and custom ranges.

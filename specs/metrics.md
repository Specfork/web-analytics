# Metrics

**Status:** Draft · **Prefix:** MET

## Purpose

The numbers a site dashboard shows: summary stats, charts, breakdown tables, channels and export.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| MET-1 | `GET /api/sites/{id}/stats` for a range and filters ([QRY](query.md)) → `{pageviews, visitors, visits, bounces, total_time}`: `pageviews` = pageview events; `visitors` = distinct sessions with ≥ 1 pageview; `visits` = distinct visits with ≥ 1 pageview; `total_time` in seconds (MET-3). Custom events count in none of these. With the default daily salt ([PRIV-4](privacy.md)) a person returning on another day is another session, so multi-day `visitors` add up daily uniques. | 3 pageviews + 2 events in 1 visit → pageviews 3, visits 1; same browser Mon and Tue → visitors 2 |
| MET-2 | Bounce: a visit with exactly one pageview and no custom event. `bounce_rate` = min(bounces, visits) / visits × 100, shown rounded; `visits` = 0 → null. | 4 visits, 1 bounce → 25 % |
| MET-3 | Visit duration = last pageview time − first pageview time of the visit, in whole seconds (single-pageview visit → 0); `total_time` = sum over visits; `avg_visit_duration` = total_time / visits. | pageviews at 0 s and 90 s → 90 s |
| MET-4 | `compare`: `prev` (same length, ending where the range starts) or `yoy` (range minus one year). The response adds `comparison` with the same fields; the dashboard shows each value's change (absolute, and percent when the previous value > 0). | 7-day range, prev → the 7 days before |
| MET-5 | `GET .../series` → `{pageviews: [{t, y}], visitors: [{t, y}]}` per `unit` bucket in the request time zone; buckets with no data are returned with `y` = 0; `visitors` counts distinct sessions per bucket. | last 7 days by day → 7 points each |
| MET-6 | `GET .../breakdown?type=` with `type` one of `path`, `entry`, `exit`, `title`, `hostname`, `query`, `referrer`, `channel`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`, `event`, `tag`, `browser`, `os`, `device`, `screen`, `language`, `country`, `region`, `city`; other → 400. Rows `{value, visitors}` (`views` added for page types), visitors descending, empty values left out, `limit` default 10 (max 500), `offset`, optional `search` (substring). | `type=browser` → Chrome first |
| MET-7 | `entry` and `exit` = first and last pageview path of each visit; `query` splits stored query strings into `key=value` rows; `referrer` excludes empty and self-referrals; `region` and `city` rows also carry `country`; `event` counts custom events only. | visit /a → /b → entry /a, exit /b |
| MET-8 | Channels: each visit takes the channel of its earliest event that classifies; none → `direct`. Rules in order: `click_id` set → `paid_ads`; `utm_medium` contains `affiliate` → `affiliate`; `utm_medium` or `utm_source` contains `sms` → `sms`; referrer is an AI assistant → `ai`; search engine (or `utm_medium` contains `organic`) → `*_search`; social network → `*_social`; webmail (or `utm_medium` contains `mail`) → `email`; marketplace → `*_shopping`; video site → `*_video`; any other external referrer or `utm_medium` in `referral`, `app`, `link` → `referral`. `*` = `paid` when `utm_medium` contains `cpc`, `ppc`, `paid` or `retargeting`, else `organic`. | google.com referrer, `utm_medium=cpc` → `paid_search` |
| MET-9 | Channel domain lists (suffix or label match, operator-editable): AI: chatgpt.com, claude.ai, perplexity.ai, gemini.google.com, copilot.microsoft.com; search: google.*, bing.com, duckduckgo.com, yandex.*, baidu.com, ecosia.org, search.brave.com; social: facebook.com, instagram.com, x.com, t.co, twitter.com, linkedin.*, reddit.*, news.ycombinator.com, bsky.app, threads.net, tiktok.*, pinterest.*; webmail: mail.google.com, outlook.*, mail.yahoo.*, proton.me; marketplace: amazon.*, ebay.*, etsy.com, aliexpress.com; video: youtube.*, twitch.*. | referrer `chatgpt.com` → `ai` |
| MET-10 | Dashboard (manual layout): summary row (visitors, visits, pageviews, bounce rate, avg visit duration, each with change), chart (MET-5), then tables: pages/entry/exit, referrers/channels, browsers/OS/devices, countries (with world map)/regions/cities, languages/screens, events. | manual: overview page |
| MET-11 | Export: one archive of CSV files (pages, referrers, channels, events, browsers, OS, devices, countries) for the range and filters, each row `value, visitors`. Cells starting with `=`, `+`, `-`, `@`, tab or carriage return are prefixed with `'`. | export → 8 CSVs; `=x` cell → `'=x` |

## Out of scope

- Funnels, journeys, cohort retention, goals, revenue and attribution reports.

## Decisions

- Channel rules written once, in one order; the source had two differing versions (`p%` vs `cp` for paid) per database engine.
- A visit's channel comes from its earliest classifying event, as in the source.
